# The Smallest Eval That Beats an Opinion

*You change a prompt, it feels better, you ship it. Next week it is worse on a case you forgot existed. The fix is three test cases and one script, and you can build it this afternoon.*

![An isometric scene: a small rack of coral input cards each running through a teal checker that stamps it pass or fail against an expected answer card, feeding a single round score gauge at the end](hero.png)

Here is the loop almost every team runs without noticing. You tweak a prompt or swap a model. You eyeball a few outputs. They look better. You ship. A week later something that used to work is broken, and you cannot tell which change did it, because "looks better" left no record. The whole quality process was a feeling, and feelings do not diff.

The fix is small and unglamorous. You write down a handful of cases with their expected answers, and a short script that runs them and prints a number. Now every change produces a score instead of a vibe. A prompt edit that drops you from five-of-five to three-of-five gets caught before it ships, not after a user finds it.

This is the concrete version of holding the bar. You cannot verify a result against a standard you never wrote down, and an eval is that standard made executable. You do not need a framework, a platform, or a vendor. You need three cases and thirty lines.

**Three things to take away:**

- An eval turns "it feels better" into a number. That single change, from judgement to measurement, is what lets you make prompt and model changes without quietly breaking things you already shipped.
- You do not need exact-match scoring or a fancy harness. Most useful evals check a property (the right label, valid JSON, a required field) over a few cases you care about, and the smallest version is about thirty lines of plain code.
- The cases that matter are the ones that already burned you. Every bug becomes a permanent test case, which is how a three-case eval grows into real coverage without anyone planning it.

## "It feels better" is not a signal

The reason eyeballing fails is not that you have bad judgement. It is that the sample is tiny, the comparison is from memory, and the cost of a regression is paid later by someone else. You look at three outputs after a change, they read well, and you have no idea what happened to the twenty cases you did not look at, including the awkward one from last month that you fixed and forgot.

A model or prompt change is rarely uniformly better. It improves some inputs and quietly degrades others. Without a fixed set of cases you re-run every time, you only ever see the inputs you happen to glance at, which are almost always the easy ones. The hard case that regressed is exactly the one you will not think to check by hand.

An eval removes the memory and the sampling from the loop. The cases are fixed, the scoring is mechanical, and the number is the same kind of number every time, so two runs are actually comparable. That comparability is the entire point.

![A before-and-after loop. Before: change to prompt or model, then a thumbs-up gut feeling, then ship, with a hidden broken case. After: change, then run the eval, then a pass-count score, then keep or revert based on the number](diagram-loop.png)

## An eval is three things

Strip away the tooling and an eval is just three parts.

An **input**: the thing you feed the system. A user message, a document, a row of data.

An **expectation**: what a correct result looks like. Sometimes that is an exact answer, but more often it is a property, the right label, a valid shape, the presence of a required field, the absence of a forbidden one.

A **scorer**: a function that takes the system's output and the expectation and returns pass or fail. The scorer is where the judgement you used to apply by eye gets written down once and applied forever.

That is the whole model. Everything else, dashboards, datasets, judges, CI, is an elaboration on input plus expectation plus scorer. Start with the three parts and add only what a real problem forces you to add.

## The smallest version, in about thirty lines

Say you have a function that classifies a support message by intent. The thing under test returns something like `{"intent": "refund"}`. Your cases are a file, one JSON object per line:

```json
{"id": "refund-clear",     "input": "I want my money back, this is broken", "expect": "refund"}
{"id": "praise-not-refund","input": "love it, works great",                 "expect": "praise"}
{"id": "refund-polite",    "input": "could you process a return for #5512?", "expect": "refund"}
{"id": "cancel-intent",    "input": "please cancel my plan",                 "expect": "cancel"}
{"id": "refund-typo",      "input": "wnat a refundd pls",                    "expect": "refund"}
```

And the runner is short enough to read in one sitting:

```python
import json, sys
from app import classify   # the thing under test: returns {"intent": "..."}

def load(path):
    with open(path) as f:
        return [json.loads(line) for line in f if line.strip()]

def run(path):
    cases = load(path)
    passed = 0
    for c in cases:
        got = classify(c["input"]).get("intent")
        ok = got == c["expect"]
        passed += ok
        print(f"[{'PASS' if ok else 'FAIL'}] {c['id']:18} "
              f"expected={c['expect']:8} got={got}")
    print(f"\n{passed}/{len(cases)} passed  ({passed/len(cases):.0%})")
    sys.exit(0 if passed == len(cases) else 1)

if __name__ == "__main__":
    run(sys.argv[1] if len(sys.argv) > 1 else "cases.jsonl")
```

Run it and you get a verdict you can actually act on:

```text
$ python eval.py cases.jsonl
[PASS] refund-clear        expected=refund   got=refund
[PASS] praise-not-refund   expected=praise   got=praise
[FAIL] refund-polite       expected=refund   got=question
[PASS] cancel-intent       expected=cancel   got=cancel
[PASS] refund-typo         expected=refund   got=refund

4/5 passed  (80%)
```

There is the regression, in plain sight. A polite return request now reads as a generic question. You caught it because the case was in the file, not because you happened to test that exact phrasing today. The exit code is non-zero, so this can fail a commit or a CI step the moment it drops below your bar.

![A terminal running the eval: five named cases, four marked PASS in green and one FAIL in coral where a polite refund was misclassified as a question, ending with 4/5 passed and 80 percent](cc-eval-run.png)

## How to score (it is rarely exact match)

Exact match works for classification, where there is one right label. Most other tasks need a softer scorer, and picking the right kind is most of the craft.

![A grid of four scoring approaches: exact match for labels and closed answers; property and assertion checks for shape, contains, valid JSON, required fields; LLM-as-judge for open-ended answers against a rubric, pinned and used sparingly; pairwise comparison for ranking two outputs when there is no single right answer](diagram-scoring.png)

**Exact match** is for closed answers: the label, the category, the boolean. Fast, deterministic, no ambiguity.

**Property and assertion checks** cover most real work. You are not asking "is this the one true output," you are asking "does it have the properties a correct output must have." Is it valid JSON. Does it contain the order number. Does it stay under the length limit. Does it avoid the banned phrase. A handful of asserts catches the failures that actually matter and ignores harmless variation in wording.

```python
def score_extraction(output: dict) -> bool:
    return (
        isinstance(output.get("total"), (int, float))
        and output.get("currency") in {"EUR", "USD", "GBP"}
        and len(output.get("line_items", [])) > 0
    )
```

**LLM-as-judge** is for open-ended output where there is no clean property to check: a summary, a rewrite, an explanation. You ask a model to grade the answer against a rubric. It is genuinely useful and also the easiest scorer to get wrong, so three rules. Pin the judge model, its version, and temperature zero, or your scores drift under you. Never let the model under test grade its own output. And use it only where a property check cannot do the job, because it costs money and adds noise.

```python
def judge(question: str, answer: str, rubric: str) -> bool:
    prompt = (f"Question: {question}\nAnswer: {answer}\n"
              f"Criteria: {rubric}\n"
              "Reply with PASS or FAIL and one short reason.")
    verdict = call_judge(prompt, model="pinned-judge-v1", temperature=0)
    return verdict.strip().upper().startswith("PASS")
```

**Pairwise comparison** sidesteps absolute scoring entirely. When there is no single right answer, ask which of two outputs is better. Models are better at "A or B" than at "rate this 1 to 10," so a pile of pairwise judgements aggregated into a ranking beats absolute scores for things like tone or quality.

## Wire it to every change

An eval that you remember to run is worth far less than one that runs on its own. Make it a single command, `make eval` or `npm run eval`, so running it costs nothing. Then attach it to the moments that matter: before you commit a prompt change, when you bump a model version, on every pull request that touches the prompt or the chain.

The discipline is the same red-to-green loop tests gave you. You change the prompt, the eval drops, you see exactly which case broke, you fix it or you revert. The number is the gate. A change that lowers the score does not ship until you either recover the case or consciously decide that case no longer matters and remove it, on purpose, with a note.

This is what closes the loop on the bar. The bar is the set of cases and the threshold. The eval is the check that holds it. "It feels better" becomes "five of five, up from four," and that sentence is something you can stand behind.

## The traps that make an eval lie to you

A bad eval is worse than none, because it gives you false confidence. The common ways one goes wrong:

- **Only happy-path cases.** An eval full of easy inputs always passes and protects nothing. Seed it from real failures, the inputs that actually broke, not the ones you know work.
- **The answer leaks into the prompt.** If your prompt mentions the exact phrasings in your cases, you are testing memorization, not behavior. Keep the eval set separate from anything the prompt sees.
- **The judge grades its own work.** Using the same model and prompt to produce and to grade inflates every score. The grader must be independent.
- **Chasing one hundred percent.** The goal is catching regressions, not a perfect score. An eval that sits at ninety percent and reliably drops when you break something is more useful than one gamed to a hundred.
- **A score with no cases behind it.** "Quality is 8.4" means nothing without the inputs that produced it. Keep the per-case results, not just the aggregate, so a drop tells you what broke.

## Growing from three cases to thirty

The three-case version is not a toy you replace later. It is the seed. The rule that grows it is simple and runs itself: every time something breaks in production, the broken input becomes a new case with its correct expected output, before you fix the bug. The fix then has to make that case pass without breaking the others. Do this for a few months and you have a real regression suite that nobody sat down to design, built entirely from the failures you actually hit.

When the file gets big, add light structure. Tag cases by category so a drop tells you where. Set a threshold so the gate is "no regressions and at least N of M," not "all green." Only reach for a dedicated eval framework when you genuinely need things the script cannot give you cheaply, parallel runs over thousands of cases, shared dashboards, statistical significance. Most teams need that later than they think, and many never do.

The whole point is to stop shipping on a feeling. You change something, you run the cases, you read the number, you keep it or you revert. It is the cheapest reliable signal you can build, it takes an afternoon, and it is the difference between knowing a change is better and hoping it is. Write the three cases today. The thirtieth will write itself.

---

*Marco Kotrotsos, specializing in practical AI implementation for organizations ready to close the gap between AI hype and AI value. With 30 years of IT experience now focused purely on AI deployment, he works hands-on with companies to turn AI potential into measurable business outcomes.*

*This article is published in [Autocomplete](https://medium.com/autocomplete-real-world-ai), a Medium publication about real-world AI for practitioners and decision-makers. We're always looking for writers. If you're building with AI and have something worth sharing, reach out.*

*My free Substack newsletter, also called Autocomplete, can be found here: https://acdigest.substack.com.*

*My books on Amazon: [Claude Code for Everyone Else](https://www.amazon.com/dp/B0H35YY851) and [From Vibe to Production](https://www.amazon.com/dp/B0H34GK9VW).*
