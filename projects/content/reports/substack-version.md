# Your 2024 Prompt Stack Is Slowing Down Your 2026 Models

*OpenAI quietly told you to throw it away. The deeper shift is that there are now two prompt-engineering jobs. Most teams are using the wrong playbook for both.*

![Two distinct prompt shapes on an isometric stage: a compact structured card labeled assistant on the left and a single bold goal-shaped slab labeled agent on the right](hero.png)

I have rewritten more client prompt stacks in the last two months than in the previous two years combined. The reason is something most of you can do this afternoon.

OpenAI's GPT-5.5 prompting guide says it plainly: "Begin migration with a fresh baseline instead of carrying over every instruction from an older prompt stack." Anthropic's Claude Opus 4.7 best practices say the same thing slightly differently. The prompts you wrote in 2024 are net negative on the models you are paying for in 2026.

That is not the headline. The headline is the deeper one underneath. There are now two prompt engineering jobs, and they require different prompts. Most teams I see are using assistant-style prompts on agentic tasks (and getting rigidly-followed wrong paths) or agent-style prompts on assistant tasks (and getting vague output that nobody likes).

This newsletter is the operational walkthrough of both moves. Strip the old rituals out of your stack. Then decide which of the two new jobs you are doing, and write the prompt that matches.

Hit reply if you want me to look at a specific prompt your team uses.

**Three things to know up front:**

OpenAI's GPT-5.5 prompting guide tells you, in plain language, to start with a fresh baseline instead of carrying over every instruction from your old prompt stack. The guidance under that sentence is more consequential than the sentence itself: the things that used to give you a 5-7% accuracy bump (chain-of-thought primers, "take a deep breath," role-play openers) now hurt the newest models because they were trained against exactly those tics. The stack you wrote in 2024 is actively slowing down the model you are paying for in 2026.

There are now two prompt engineering jobs, and they are different jobs. Prompting an assistant (single turn, predictable steps, you supervise the output) rewards precision. Prompting an agent (multi-turn, open-ended, autonomous for minutes to hours) rewards clear outcomes plus trust. The mistake most teams make is using assistant-style prompts on agents (over-specifying every step) or agent-style prompts on assistants (vague goal, no constraints, no format).

Prompt engineering did not die. It got narrower. It is now one layer in a five-layer stack alongside model choice, system prompt design, tools and MCP wiring, context engineering, and eval harnesses. Treat it as one move in the playbook, not the whole playbook, and the rest of 2026 gets easier.

## What the older tricks are doing to your newer models

For three years, prompt engineering culture accumulated a set of standard moves that gave measurable accuracy bumps on the models of the day. A few of the famous ones:

- "Let's think step by step" before a hard reasoning problem
- "Take a deep breath and work through this carefully"
- "You are an expert in X" or "Act as a senior engineer in Y" role-play openers
- Worked examples (few-shot prompting) for almost every task
- "Output JSON, do not include any other text" rituals
- Splitting complex tasks into "first, then, finally" sequential micro-steps

These patterns worked. The original chain-of-thought paper from 2022 showed that "let's think step by step" alone improved grade-school math accuracy by roughly 7 percentage points. The performance gain was real and reproducible. So the patterns spread, into engineering documents, into Cursor and Claude Code defaults, into the README of every internal AI tool at every company shipping anything in 2023 and 2024.

What changed is that the model providers noticed too. RLHF and the newer constitutional methods explicitly trained the next generation of models to internalize what those patterns were trying to elicit. Modern reasoning models think step-by-step by default. Telling Opus 4.7 or GPT-5.5 to "think step by step" no longer gives you an accuracy bump. In some evaluations it gives a small accuracy loss, because you are spending tokens to instruct the model to do something it would already do better on its own, and you may be narrowing the reasoning path it would otherwise explore.

Role-play openers ("You are a senior security engineer") are in a similar bucket. They sometimes help on weaker or older models. On Opus 4.7 they tend to add a layer of performative persona that the model has to deconflict with the actual task, which is wasted effort. The cleaner pattern is just describing the task and the success criteria.

Few-shot examples remain useful, but more selectively. For structured extraction where the format is non-obvious, examples still help. For reasoning tasks, one or two thoughtful examples are usually better than five. Five examples often anchor the model to imitate the surface shape rather than apply the underlying reasoning.

The summary, said plainly: the prompts you wrote two years ago are not neutral on the models you are running now. They are net negative for a non-trivial fraction of tasks. OpenAI's own documentation says so. Anthropic's GPT-5.5 prompting guide and Claude Opus 4.7 prompting best practices both say so.

## The split nobody is being precise about

Here is the part of the conversation that has been muddy.

The phrase "prompt engineering" in 2026 covers two genuinely different jobs. The skill set overlaps, but the right moves differ enough that conflating them is the source of most of the bad prompts I see.

![Side-by-side comparison of an assistant prompt anatomy with six precise sections and an agent prompt anatomy with four outcome-shaped sections](diagram-1-anatomy.png)

### Job one: prompting an assistant

You are asking the model for a single answer or a short bounded conversation. You see the output and decide what to do with it. The model is collaborating, you are supervising.

Examples: ChatGPT for a draft email, Claude for a literature review, Cursor for an inline code suggestion, an internal RAG bot for a question against your wiki.

The right prompt shape here is precise:

- **Role.** Who is the model for this turn. Brief. (Not a performance.)
- **Context.** What background it needs that is not obvious.
- **The exact task.** What you want produced, written as a clear instruction.
- **Format.** What the output should look like (JSON shape, headings, length).
- **Examples.** One or two if the format or judgment is non-obvious.
- **Constraints.** What it should not do.

This is the prompt-engineering most write-ups still teach, and it remains correct for the assistant case. The 2026 update is mostly: drop the magic-phrase additives. Keep the structure. Tighten the role to a description rather than a costume.

### Job two: prompting an agent

You are delegating a multi-step task. The model will work for minutes, hours, or longer. It will call tools, read files, write files, make decisions you do not see in real time. You will supervise the outcome, not every step.

Examples: Claude Code building a feature end to end, a research agent producing a report, an agent fixing a bug in your repo, an autonomous deploy validation flow.

The right prompt shape:

- **Goal.** What success looks like. Specific and observable.
- **Success criteria.** How the agent (and you) will know it is done.
- **Available tools.** What the agent can use, with rough sense of when.
- **Guardrails.** What it must not do, and when to stop and ask.

The notable absence: a step-by-step process. Anthropic's own guidance: "Agents can be used for open-ended problems where it's difficult or impossible to predict the required number of steps." OpenAI's: "GPT-5.5 is strongest when the prompt defines the target outcome, success criteria, constraints, and available context, then lets the model choose the path."

For agents, you specify the destination. You do not specify the road.

The teams I have advised who try to write agent prompts the assistant way end up with two failure modes. Either the agent rigidly follows the steps you specified and misses obvious better paths, or it ignores half your steps and you cannot tell which half because the rest is hidden inside a long autonomous loop. Both failure modes are caused by the same mistake: assistant-style prompting on an agent-shaped task.

## The mirror failure on the other side

Equally common, and slightly less talked about: agent-style prompting on an assistant-shaped task.

This is what happens when a team reads "describe the destination, not the road" and starts writing prompts like "summarize this document for me, you decide how" against a single-turn ChatGPT conversation. The model produces something. Sometimes useful, often vague, never quite what you wanted.

Assistants benefit from precision. If you are doing a single-turn task with a known output format, telling the model "you decide" is leaving accuracy on the table. The model is happy to make decisions about format and structure that you would have made tighter if you had specified them. For assistants, the underspecified prompt is just a hidden form of "I will tolerate variance in the output."

## What to drop, what to keep, what to add

Two years of prompt-engineering blog posts have left most stacks with a lot of accumulated rituals.

![Three columns showing what to stop, what to keep, and what to start: stop list includes take a deep breath and lets think step by step, keep list includes clear outcome and explicit constraints, start list includes context budget and eval harness](diagram-2-stop-keep-start.png)

**Stop.**

- "Take a deep breath." The phrase was funny for a quarter. It has been trained against. Drop it.
- "Let's think step by step." Modern reasoning models do this by default. The phrase is now worth nothing, and sometimes worth slightly less than nothing because it spends tokens.
- "You are an expert in X." Role-play openers add a persona the model has to deconflict with the actual task. State the task. Skip the costume.
- Over-specified processes. If you find yourself writing "first do A, then do B, then do C" against an agent, you are using assistant prompting on an agent task.
- "Important: do not ignore these instructions." Modern models follow instructions. Adding emphatic reminders mostly tells the model that the underlying instructions are weak.

**Keep.**

- A clear, specific outcome. What does success look like, observably.
- Concrete examples when the format is non-obvious. One or two is plenty.
- Explicit constraints. What must not happen. Boundaries.
- Output format. JSON schema, structure, length, voice. Specific.
- Context the model genuinely needs that it cannot infer.

**Start.**

- A context budget for each task. How much of your context window is being burned on instructions vs the actual work. If your prompt is 2,000 tokens and the work is 5,000 tokens, that ratio is probably wrong.
- Explicit tool selection for agentic tasks. Don't just install every MCP server. Curate which tools the agent has access to for this task.
- An eval harness for prompts that matter. Three to ten test cases that you re-run every time you change the prompt. Not vibe-checking the output. Comparing structured outputs against expectations.
- Agent-level delegation. If the task is multi-step, learn to write a goal-shaped prompt instead of a process-shaped prompt. The instinct from 2023 to micromanage the model has to be unlearned.

## The bigger picture: prompt engineering as one of five layers

The "prompt engineering is dead" headlines miss what is actually happening. Prompt engineering is one layer in a five-layer stack that real production AI systems run on in 2026.

![A vertical stack of five layered slabs showing the 2026 AI engineering stack: evals on top, then context, then system prompt, then tools and MCP, then model choice at the bottom](diagram-3-stack.png)

**Model choice.** The foundation. Picking GPT-5.5 vs Opus 4.7 vs Sonnet 4.6 vs Haiku for a given task is a real engineering decision now. The same prompt that works on Opus might underperform on Sonnet.

**Tools and MCP.** What capabilities the model has access to. For agentic work this is often more important than the prompt itself.

**System prompt.** The constitution. The standing instructions that apply to every interaction. This is where most production-grade prompt engineering actually lives in 2026.

**Context engineering.** What you put into the context window beyond the prompt. A 2026 survey found 95% of data teams planning to invest in context-engineering capability this year.

**Evals.** The feedback layer. A small set of test cases that tell you whether a prompt change actually improved anything.

Prompt engineering sits in this stack. It is the writing-the-actual-words part. It is necessary, not sufficient on its own.

## What I would do this month

Three concrete moves for the next thirty days.

**Audit your prompts against the assistant-or-agent split.** For each non-trivial prompt in your codebase, your CLAUDE.md, or your skills directory, classify it: assistant or agent? Then check the shape. Most teams have at least a few miscast prompts. Fixing them is fast.

**Strip the rituals.** Take a deep breath. Let's think step by step. You are an expert in. Find them in your prompts. Remove them. Re-run your evals (if you have them) or your standard tasks (if you don't) and observe. In my experience, most prompts get slightly better or stay neutral.

**Build the smallest possible eval harness.** Three test cases. JSON-shaped expected outputs. A script that runs them on demand. This is the highest-return investment most teams have not made.

Hit reply if you want me to look at a specific prompt. I read everything.

## What is coming next

The next two newsletter issues will continue this thread:

- **The system prompt audit.** A walkthrough of what production-grade system prompts look like in 2026, with the patterns I see most often and the ones I avoid.
- **Context engineering in practice.** What it actually means to "fill the window" deliberately, with the specific patterns that work in production agents.

**One open question for you, if you are willing to share in the comments:** what is the oldest prompt ritual still in your active stack? The line you wrote in 2023 and have never updated?

Until next week.

Marco

---

*Autocomplete is a free weekly newsletter on practical AI implementation. If you are not subscribed yet, the button above this paragraph is the easiest way to fix that. If you are already subscribed, forwarding this to one engineering lead who would benefit is the single best thing you can do for me.*
