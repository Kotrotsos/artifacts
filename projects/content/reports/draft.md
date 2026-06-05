# Prompt Engineering, Summer 2026: What Is Out, What Stayed, and What Leads Now

*The model started doing the reasoning. That one change reshuffled the whole craft. A field guide to what works, what stopped working, and what is genuinely new, with the numbers behind each call.*

![An isometric prompt console: a small effort dial with notched settings, a context-window tray that is filling up, a structured-output card, and a short clean instruction slip, with a discarded pile of long elaborate prompt scrolls off to one side](hero.png)

Two years ago the job was finding the words. You coaxed a model that could not really reason into reasoning by telling it to think step by step, you primed it with "you are a world-class expert," and you stacked few-shot examples to drag it toward the answer. Those moves worked because the model needed the help.

The models now reason on their own, and that single shift has quietly retired half of the old playbook and promoted a different half. The technique that gave a 2024 model a double-digit accuracy boost gives a 2026 reasoning model almost nothing, and sometimes makes it worse. Meanwhile the things that used to feel like housekeeping, the structure of your context and the effort setting on the model, became the main event.

This is the state of prompting as of summer 2026: what is out, what stayed strong, what is new, and what is leading edge, with the evidence behind each one.

**Three things to take away:**

- The biggest change is that you no longer prompt the model to reason. You set how hard it should reason with an effort or thinking control, then spend your words on the outcome and the context. Telling a reasoning model to "think step by step" now buys you 2 to 3 percent at best and a 20 to 80 percent time tax, per a Wharton study.
- The craft did not die, it moved. Clear instructions, examples, structure, and structured output all still work. What changed is that the high-value work shifted from phrasing a single prompt to engineering the whole context window and the loop around it.
- The strongest prompt optimizer in 2026 is often another model. Automatic optimizers like GEPA lift accuracy on hard benchmarks by twenty-plus points over a human baseline, with no fine-tuning, just by rewriting the instructions.

## The big shift: from words to effort and context

The defining move of 2026 prompting is that depth of reasoning became a dial, not a sentence. On Claude Opus 4.8, thinking is off until you turn it on, and when you do, you choose an effort level that the model now respects strictly, scoping its work to what you asked at low effort and going deep at high or max. On GPT-5.x, the same idea ships as a `reasoning.effort` knob with settings from none through xhigh. You do not write "reason carefully" anymore. You set the effort and let the model decide how to use it.

That changes what your words are for. They are no longer there to induce reasoning, they are there to specify the destination. OpenAI's own guidance for GPT-5 is to define the target outcome, the success criteria, the constraints, and the context, then let the model find the route, because the reasoning engine is now better at finding an efficient path than your step-by-step instructions are at dictating one. Anthropic frames the same shift from the context side: building with models is becoming less about finding the right words and more about answering "what configuration of context is most likely to generate the desired behavior."

Hold those two ideas together, effort as a dial and context as the real surface, and most of what follows falls out of them.

![Two prompt cards side by side. The 2023 card is long and cluttered: "You are a world-class expert. Take a deep breath and think step by step. I'll tip you $200..." with a numbered procedure. The 2026 card is short: an effort setting, then a one-line outcome with explicit success criteria and a "do not start until you have a plan" line](screenshot-prompts.png)

## What is out

These techniques earned their place in the 2023 playbook and have aged badly. The evidence that they stopped working is the most useful thing in this article, because people are still paying the cost of using them.

**"Think step by step" on reasoning models.** This is the headline casualty. The Wharton Generative AI Labs report on the decreasing value of chain-of-thought put real numbers on it. On reasoning models, forcing explicit chain-of-thought moved accuracy by 2.9 percent on o3-mini, 3.1 percent on o4-mini, and *negative* 3.3 percent on Gemini Flash 2.5, while adding 20 to 80 percent to the response time. The model already reasons internally, so telling it to reason out loud mostly adds latency and occasionally derails it. OpenAI's documentation now warns against it directly.

**Manual thinking budgets.** On Claude Opus 4.8, the old approach of setting a fixed `budget_tokens` for thinking is gone. Adaptive thinking is the only thinking-on mode, and Anthropic's evaluations show it reliably beats hand-tuned fixed budgets. If reasoning looks shallow, you raise the effort level, you do not micromanage a token count.

**Personality padding.** "You are a world-class expert." "Take a deep breath." "I'll tip you $200." OpenAI's GPT-5 guidance is blunt that this is now noise the model reads as filler. The capability is already there, so priming it as an expert does nothing, and the theatrical instructions just clutter the prompt. Politeness and emotional framing do still nudge outputs, but the effect is small, inconsistent across models, and shrinking, because newer models increasingly process "please" as a string rather than an emotional signal. Do not build on it.

**Few-shot examples to teach reasoning.** Few-shot is not dead, but its job changed. Stacking worked examples to show a reasoning model *how* to think no longer improves the reasoning, because the model decomposes internally. The remaining value of examples is format alignment, showing the shape of the output you want, which is a real and different use covered below.

**The forty-page system prompt.** The era of the enormous rigid system prompt full of chain-of-thought scaffolding, tree-of-thought branches, and dozens of rules is over, and not just because it is unfashionable. It actively hurts, because of context rot: a model's ability to recall information accurately decreases as the token count climbs, so a bloated prompt degrades the very behavior it was trying to pin down.

![A four-column map of prompting techniques in 2026: Out (think step by step on reasoning models, manual thinking budgets, persona and emotion padding, few-shot for reasoning, 40-page system prompts), Stayed strong (clear specific instructions, examples for format, structure and XML tags, structured output, CoT on non-reasoning models), New (effort and thinking dials, adaptive thinking, outcome-first prompts, status-update messages), Leading edge (context engineering, automatic prompt optimization, meta-prompting, eval-driven prompts, prompts as files)](diagram-map.png)

## What stayed strong

Plenty survived, and it is the unglamorous core. If you only know this section, you are most of the way there.

**Clarity and ruthless specificity.** The single most reliable thing you can do is still say exactly what you want, for whom, in what form, with what constraints. The frameworks that package this, Role-Context-Task-Format and its cousins, persist because clear thinking expressed clearly is model-agnostic and timeless. This never stopped mattering and it never will.

**Examples, for shape not for reasoning.** Three to five diverse examples that show the format and tone you want are still high-return, especially for Claude, where wrapping them in XML tags reads cleanly. The reframing is that examples teach the *shape* of a good answer, not the path to it.

**Structure and Claude's XML tags.** Sectioning a prompt into identity, rules, context, and output expectations still helps every model parse it. For Claude specifically, XML tags to delimit those sections remain a recommended, durable technique.

**Structured output.** Asking for valid JSON against a schema, or a fixed set of keys, is more important than ever now that prompts feed agents and pipelines rather than humans. "Respond only in valid JSON with these exact keys" is not a trick, it is the interface.

**Chain-of-thought, on the right models.** Here is the nuance the headlines miss. CoT did not die everywhere. On non-reasoning models it still delivers, the same Wharton report found roughly 11 to 13 percent gains on Sonnet 3.5 and Gemini Flash 2.0, and on genuinely hard tasks it still helps. It is specifically on reasoning models that it stopped paying. Match the technique to the model.

## What is new

The genuinely new surface in 2026 is control over how the model thinks, exposed as settings rather than sentences.

**The effort and thinking dials.** Claude's effort levels (low, high, xhigh, max, plus the ultracode mode for agentic work) and GPT-5's `reasoning.effort` (none, low, medium, high, xhigh) are the new primary lever. You match the dial to the task: low for bounded mechanical work, high or max for the hard problem where being wrong is expensive. This is the single most important new habit, and most people leave the dial on one setting for everything.

**Adaptive thinking.** Rather than a fixed thinking budget, the model decides how much to think per request. On Claude this is now the only thinking mode, and it outperforms the hand-tuned approach it replaced.

**Outcome-first prompting.** The structural change underneath all of this. You write the target and the success criteria, not the procedure. "Produce a migration plan that keeps the existing tests green and touches no public API" beats a numbered list of steps, because the model routes better than your steps do.

**Status-update messages.** A small, practical addition for agents and long-running reasoning: when the model will think for a while before a visible answer, have it emit a short one or two sentence acknowledgement of the request and the first step, so the user is not staring at nothing. OpenAI's GPT-5 guidance specifically recommends this.

## The leading edge

This is where the serious practitioners are spending their attention, and where the phrase "prompt engineering" is quietly being replaced.

**Context engineering.** Anthropic put the clearest stake in the ground: the question moved from "what words" to "what configuration of context." That means treating the context window as a finite resource with diminishing returns, fighting context rot, and using just-in-time loading, where an agent holds lightweight references and pulls data at runtime instead of stuffing everything in up front. The counterintuitive headline that comes with it: more tokens can make an agent worse, not better.

**Tool descriptions as a prompt surface.** For anyone building agents, the tool descriptions are now one of the highest-leverage things to prompt-engineer, because they sit in the agent's context and steer its behavior on every call. Anthropic's advice is to write them the way you would brief a new hire: spell out the niche terminology, the query formats, and the relationships you would otherwise leave implicit.

**Automatic prompt optimization.** The strongest result in this whole space is that a model can now write better prompts than you can. GEPA, an evolutionary optimizer in the DSPy ecosystem and an ICLR 2026 paper, reports lifting a chain-of-thought program on the MATH benchmark from a 67 percent baseline to 93 percent through instruction refinement alone, no few-shot examples and no fine-tuning. In a production case, Shopify moved a single GPT-5 prompt to a small open model optimized with GEPA and reported a solution roughly 75 times cheaper and twice as reliable. Hand-tuning prompts by eye is starting to look like hand-tuning assembly.

**Eval-driven prompting.** You cannot run an optimizer, or trust a prompt change, without a way to score it. The leading-edge workflow treats prompts like code: a small set of cases with expected outputs, a script that scores them, and every change gated on the number rather than on how it felt. The optimizer and the eval are the same loop.

**Prompts as files.** The prompt stopped living in a text box. It lives in `CLAUDE.md`, `AGENTS.md`, and skill files checked into the repo, versioned, reviewed, and loaded on demand. The instruction set became part of the codebase, which is the clearest signal that the discipline grew up.

## Best practices, concretely

The working synthesis, with the prompt shapes that go with each.

**Set the dial, then describe the outcome.** Choose the effort level for the task, then spend the prompt on the destination and the constraints.

> *(effort: high)* Refactor the auth module to use the new token service. Done means: every existing test passes, no public function signature changes, and the diff is under 400 lines. Do not start coding until you have a plan.

**Stop telling reasoning models to think.** Drop "think step by step," "take a deep breath," and "you are an expert" from anything running on a reasoning model. They add latency and noise. Keep CoT only for non-reasoning models and genuinely novel problems.

**Use examples for shape, not for reasoning.** When the format matters, show two or three examples of the exact output you want, wrapped in tags. Do not use examples to demonstrate how to reason.

**Demand the structure.** If the output feeds anything other than a human eye, specify it as JSON against named keys, and say "only" so nothing else comes back.

**Engineer the context, not just the prompt.** Put the stable, important material first, keep the window lean, and load detail on demand. Treat every token as competing with every other token for the model's attention.

**Write tool descriptions like onboarding docs.** For agents, invest in the tool specs. Name the formats, the terms, and the relationships a new hire would need.

**Optimize with evals, not vibes.** Build a few scored cases, then let a change earn its place by moving the number. When the stakes are high, let an optimizer rewrite the prompt against those cases.

## Benchmarks and evidence

The numbers that anchor the calls above, with sources.

![A grouped bar chart of chain-of-thought accuracy change by model type, from the Wharton GAIL report. Non-reasoning models gain: Gemini Flash 2.0 plus 13.5 percent, Sonnet 3.5 plus 11.7 percent, GPT-4o-mini plus 4.4 percent. Reasoning models barely move: o3-mini plus 2.9 percent, o4-mini plus 3.1 percent, Gemini Flash 2.5 minus 3.3 percent. A note records the 20 to 80 percent time cost on reasoning models](chart-cot.png)

The Wharton Generative AI Labs chain-of-thought report is the cleanest evidence for the central claim. On non-reasoning models, explicit CoT helped: Gemini Flash 2.0 by 13.5 percent, Sonnet 3.5 by 11.7 percent, GPT-4o-mini by a non-significant 4.4 percent. On reasoning models it barely registered: o3-mini 2.9 percent, o4-mini 3.1 percent, Gemini Flash 2.5 down 3.3 percent, all while adding 20 to 80 percent to response time. The report's own conclusion is that for reasoning models the gains rarely justify the time.

![A simple before-and-after bar showing GEPA prompt optimization lifting MATH benchmark accuracy from a 67 percent unoptimized chain-of-thought baseline to 93 percent, labeled as instruction refinement only, no fine-tuning, from the DSPy GEPA work](chart-gepa.png)

On the optimization side, GEPA's reported jump from 67 to 93 percent on MATH, through instruction rewriting alone, is the strongest single data point that automatic optimization beats hand-tuning. Treat the headline figure as the authors' benchmark rather than a universal guarantee, but the direction is consistent across the DSPy results and the Shopify production case.

One honest caveat on the softer numbers. The widely repeated survey statistics that "82 percent of leaders say prompting alone is insufficient" or "95 percent plan to invest in context engineering" come from industry reports rather than peer-reviewed work, so treat them as directional sentiment, not measurement. The shift they describe is real, the Anthropic and OpenAI primary guidance confirms it, but the precise percentages are marketing-grade.

## Surprising findings that held up

The counterintuitive results worth knowing, after filtering the ones that did not survive a second look.

**Telling a reasoning model to think can make it worse.** Not just neutral, negative, as the Gemini Flash 2.5 number shows. The instinct that more explicit reasoning is always safer is now wrong on the models built to reason.

**The best prompt writer is a machine.** A model with an evolutionary loop and an eval set rewrites instructions better than an expert human, by a wide margin on hard tasks. The skill is moving from writing the prompt to defining the objective the optimizer writes against.

**More context can hurt.** Context rot means that padding an agent with extra information degrades its recall and its behavior. The reflex to give the model everything is the opposite of what the leading practice now recommends.

**Politeness is becoming a no-op.** The research on tone is genuinely mixed, and the trend is that as models improve they treat politeness as text rather than emotional payload, so the effect shrinks. Be polite because it keeps you civil, not because it changes the output.

## Where this leaves you

The through-line of summer 2026 is that the model absorbed the part of prompting that used to be clever. You no longer trick it into reasoning, prime it into competence, or scaffold its thinking with elaborate instructions, because it brings those itself. What is left for you is the part that was always the real work: say precisely what you want, set how hard the model should work, engineer the context it sees, and measure whether the result is right.

That is a less magical craft than 2023 prompting and a more durable one. The clever phrases were always going to be temporary, because each model release made another one obsolete. Clarity, structure, context discipline, and evaluation do not expire. Spend your time there, set the dial, and let the model do the reasoning it is now very good at.

---

*Marco Kotrotsos, specializing in practical AI implementation for organizations ready to close the gap between AI hype and AI value. With 30 years of IT experience now focused purely on AI deployment, he works hands-on with companies to turn AI potential into measurable business outcomes.*

*This article is published in [Autocomplete](https://medium.com/autocomplete-real-world-ai), a Medium publication about real-world AI for practitioners and decision-makers. We're always looking for writers. If you're building with AI and have something worth sharing, reach out.*

*My free Substack newsletter, also called Autocomplete, can be found here: https://acdigest.substack.com.*

*My books on Amazon: [Claude Code for Everyone Else](https://www.amazon.com/dp/B0H35YY851) and [From Vibe to Production](https://www.amazon.com/dp/B0H34GK9VW).*
