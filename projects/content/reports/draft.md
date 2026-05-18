# Prompt Engineering Split in Two in 2026

*OpenAI quietly told you to throw away your prompt stack. The deeper shift is that there are now two prompt-engineering jobs, not one. Most teams are still using the 2024 playbook for both.*

![Two distinct prompt shapes on an isometric stage: a compact structured card labeled assistant on the left and a single bold goal-shaped slab labeled agent on the right](hero.png)

**Key takeaways:**

- OpenAI's GPT-5.5 prompting guide tells you, in plain language, to start with a fresh baseline instead of carrying over every instruction from your old prompt stack. The guidance under that sentence is more consequential than the sentence itself: the things that used to give you a 5-7% accuracy bump (chain-of-thought primers, "take a deep breath," role-play openers) now hurt the newest models because they were trained against exactly those tics. The stack you wrote in 2024 is actively slowing down the model you are paying for in 2026.
- There are now two prompt engineering jobs, and they are different jobs. Prompting an assistant (single turn, predictable steps, you supervise the output) rewards precision. Prompting an agent (multi-turn, open-ended, autonomous for minutes to hours) rewards clear outcomes plus trust. The mistake most teams make is using assistant-style prompts on agents (over-specifying every step) or agent-style prompts on assistants (vague goal, no constraints, no format).
- Prompt engineering did not die. It got narrower. It is now one layer in a five-layer stack alongside model choice, system prompt design, tools and MCP wiring, context engineering, and eval harnesses. Treat it as one move in the playbook, not the whole playbook, and the rest of 2026 gets easier.

---

The Medium piece making the rounds this week is titled "OpenAI quietly told you to throw away your prompt stack." The headline is dramatic. The underlying observation is correct.

If you read OpenAI's own GPT-5.5 prompting guide carefully, the framing is plain: "Begin migration with a fresh baseline instead of carrying over every instruction from an older prompt stack. Legacy prompts often over-specify the process because earlier models needed more help staying on track. With GPT-5.5, that can add noise, narrow the model's search space, or lead to overly mechanical answers." That sentence is doing a lot of work.

The first time I read it, I assumed it was the usual model-release positioning: this one is better, please update your code. After a few weeks of using GPT-5.5 and Claude Opus 4.7 against the prompt scaffolding I had been carrying since GPT-4 days, I realized the recommendation is more specific than that. The newer models are worse at following the old prompt patterns. They were trained against them.

The interesting question is what to do instead, and the answer is two questions. What are you prompting? An assistant or an agent? Those are different jobs in 2026, and they need different prompts.

This is the article version of the answer I keep giving in client calls. Strip-mine your 2024 prompt stack. Then decide which of the two new jobs you are doing, and write the prompt that matches.

## What the older tricks are doing to your newer models

For three years, prompt engineering culture accumulated a set of standard moves that gave measurable accuracy bumps on the models of the day. A few of the famous ones:

- "Let's think step by step" before a hard reasoning problem
- "Take a deep breath and work through this carefully"
- "You are an expert in X" or "Act as a senior engineer in Y" role-play openers
- Worked examples (few-shot prompting) for almost every task
- "Output JSON, do not include any other text" rituals
- Splitting complex tasks into "first, then, finally" sequential micro-steps

These patterns worked. The original chain-of-thought paper from 2022 showed that "let's think step by step" alone improved grade-school math accuracy by roughly 7 percentage points. The performance gain was real and reproducible. So the patterns spread, into engineering documents, into Cursor and Claude Code defaults, into the README of every internal AI tool at every company shipping anything in 2023 and 2024.

What changed is that the model providers noticed too. RLHF (reinforcement learning from human feedback) and the newer constitutional methods explicitly trained the next generation of models to internalize what those patterns were trying to elicit. Modern reasoning models think step-by-step by default. Telling Opus 4.7 or GPT-5.5 to "think step by step" no longer gives you an accuracy bump. In some evaluations it gives a small accuracy loss, because you are spending tokens to instruct the model to do something it would already do better on its own, and you may be narrowing the reasoning path it would otherwise explore.

Role-play openers ("You are a senior security engineer") are in a similar bucket. They sometimes help on weaker or older models. On Opus 4.7 they tend to add a layer of performative persona that the model has to deconflict with the actual task, which is wasted effort. The cleaner pattern is just describing the task and the success criteria. The model figures out the right register.

Few-shot examples remain useful, but more selectively. For structured extraction where the format is non-obvious, examples still help. For reasoning tasks, one or two thoughtful examples are usually better than five. Five examples often anchor the model to imitate the surface shape of the examples rather than apply the underlying reasoning.

The summary, said plainly: the prompts you wrote two years ago are not neutral on the models you are running now. They are net negative for a non-trivial fraction of tasks. OpenAI's own documentation says so. Anthropic's GPT-5.5 prompting guide and Claude Opus 4.7 prompting best practices both say so. The point is not subtle. The implementation is what most teams have not done.

## The split nobody is being precise about

Here is the part of the conversation that has been muddy.

The phrase "prompt engineering" in 2026 covers two genuinely different jobs. The skill set overlaps, but the right moves differ enough that conflating them is the source of most of the bad prompts I see.

![Side-by-side comparison of an assistant prompt anatomy (role, context, exact task, format, examples, constraints) and an agent prompt anatomy (goal, success criteria, available tools, guardrails)](diagram-1-anatomy.png)

### Job one: prompting an assistant

You are asking the model for a single answer or a short bounded conversation. You see the output and decide what to do with it. The model is collaborating, you are supervising.

Examples: ChatGPT for a draft email, Claude for a literature review, Cursor for an inline code suggestion, an internal RAG bot for a question against your wiki.

The right prompt shape here is precise. The classic frame is still useful:

- **Role.** Who is the model for this turn. Brief. (Not a performance.)
- **Context.** What background it needs that is not obvious.
- **The exact task.** What you want produced, written as a clear instruction.
- **Format.** What the output should look like (JSON shape, headings, length).
- **Examples.** One or two if the format or judgment is non-obvious.
- **Constraints.** What it should not do.

This is the prompt-engineering most write-ups still teach, and it remains correct for the assistant case. The 2026 update is mostly: drop the magic-phrase additives ("think step by step", "take a deep breath"). Keep the structure. Tighten the role to a description rather than a costume.

### Job two: prompting an agent

You are delegating a multi-step task. The model will work for minutes, hours, or longer. It will call tools, read files, write files, make decisions you do not see in real time. You will supervise the outcome, not every step. The model is executing, you are setting it up to succeed.

Examples: Claude Code building a feature end to end, a research agent producing a report, an agent fixing a bug in your repo, an autonomous deploy validation flow.

The right prompt shape here is different. Anthropic's effective-agents guidance and OpenAI's GPT-5.5 prompting guide land on the same architecture for this case:

- **Goal.** What success looks like. Specific and observable.
- **Success criteria.** How the agent (and you) will know it is done.
- **Available tools.** What the agent can use, with rough sense of when.
- **Guardrails.** What it must not do, and when to stop and ask.

The notable absence: a step-by-step process. For agentic tasks, telling the model exactly how to proceed adds noise and narrows the search space. Anthropic's own guidance: "Agents can be used for open-ended problems where it's difficult or impossible to predict the required number of steps." OpenAI's: "GPT-5.5 is strongest when the prompt defines the target outcome, success criteria, constraints, and available context, then lets the model choose the path."

For agents, you specify the destination. You do not specify the road.

The teams I have advised who try to write agent prompts the assistant way end up with two failure modes. Either the agent rigidly follows the steps you specified and misses obvious better paths, or it ignores half your steps and you cannot tell which half because the rest is hidden inside a long autonomous loop. Both failure modes are caused by the same mistake: assistant-style prompting on an agent-shaped task.

## The mirror failure on the other side

Equally common, and slightly less talked about: agent-style prompting on an assistant-shaped task.

This is what happens when a team reads the same "describe the destination, not the road" advice and starts writing prompts like "summarize this document for me, you decide how" against a single-turn ChatGPT conversation. The model produces something. Sometimes useful, often vague, never quite what you wanted.

Assistants benefit from precision. If you are doing a single-turn task with a known output format, telling the model "you decide" is leaving accuracy on the table. The model is happy to make decisions about format and structure that you would have made tighter if you had specified them. For assistants, the underspecified prompt is just a hidden form of "I will tolerate variance in the output."

## What to drop, what to keep, what to add

Two years of prompt-engineering blog posts have left most stacks with a lot of accumulated rituals. A pragmatic cleanup, based on what the model providers themselves now recommend.

![Three columns showing what to stop, what to keep, and what to start: stop list includes take a deep breath and lets think step by step, keep list includes clear outcome and explicit constraints, start list includes context budget and eval harness](diagram-2-stop-keep-start.png)

**Stop.**

- "Take a deep breath." The phrase was funny for a quarter. It has been trained against. Drop it.
- "Let's think step by step." Modern reasoning models do this by default. The phrase is now ~7% accuracy on grade-school math worth nothing, and sometimes worth slightly less than nothing because it spends tokens.
- "You are an expert in X." Role-play openers add a persona the model has to deconflict with the actual task. State the task. Skip the costume.
- Over-specified processes. If you find yourself writing "first do A, then do B, then do C" against an agent, you are using assistant prompting on an agent task. Specify the goal instead.
- "Important: do not ignore these instructions." Modern models follow instructions. Adding emphatic reminders mostly tells the model that the underlying instructions are weak. Make the instructions clear once. Trust them.

**Keep.**

- A clear, specific outcome. What does success look like, observably.
- Concrete examples when the format is non-obvious. One or two is plenty.
- Explicit constraints. What must not happen. Boundaries.
- Output format. JSON schema, structure, length, voice. Specific.
- Context the model genuinely needs that it cannot infer.

**Start.**

- A context budget for each task. How much of your context window is being burned on instructions vs the actual work. If your prompt is 2,000 tokens and the work is 5,000 tokens, that ratio is probably wrong.
- Explicit tool selection for agentic tasks. Don't just install every MCP server. Curate which tools the agent has access to for this task. Fewer, more relevant tools work better than more tools available.
- An eval harness for prompts that matter. Three to ten test cases that you re-run every time you change the prompt. Not "vibe-checking" the output. Comparing structured outputs against expectations.
- Agent-level delegation. If the task is multi-step, learn to write a goal-shaped prompt instead of a process-shaped prompt. The instinct from 2023 to micromanage the model has to be unlearned.

## The bigger picture: prompt engineering as one of five layers

The "prompt engineering is dead" headlines miss what is actually happening. Prompt engineering is one layer in a five-layer stack that real production AI systems run on in 2026. The layer is still important. It is just no longer the whole job.

![A vertical stack of five layered slabs showing the 2026 AI engineering stack: evals on top, then context, then system prompt, then tools and MCP, then model choice at the bottom](diagram-3-stack.png)

**Model choice.** The foundation. Picking GPT-5.5 vs Opus 4.7 vs Sonnet 4.6 vs Haiku for a given task is a real engineering decision now. Different models have different prompt sensitivities. The same prompt that works on Opus might underperform on Sonnet, and the right answer is often not "tune the prompt harder" but "pick the right model for the job."

**Tools and MCP.** What capabilities the model has access to. For agentic work this is often more important than the prompt itself. An agent with the right MCP servers wired up can do work that no amount of prompt engineering would unlock without them. (I wrote a separate piece on the Skills/MCP/Tools split if you want the deeper version of this layer.)

**System prompt.** The constitution. The standing instructions that apply to every interaction in a given app or agent. This is where most production-grade prompt engineering actually lives in 2026. The user prompt is often short. The system prompt is doing the heavy lifting.

**Context engineering.** What you put into the context window beyond the prompt itself. Documents, history, relevant code, retrieved snippets, prior outputs. A 2026 survey found 95% of data teams planning to invest in context-engineering capability this year. The phrase is new. The work is real: deciding what fills the window and what does not is now a primary engineering activity.

**Evals.** The feedback layer. The thing nobody had in 2023 and everyone needs in 2026. A small set of test cases that tell you whether a prompt change actually improved anything. Without evals, every prompt update is vibes. With evals, you compound.

Prompt engineering sits in this stack. It is the writing-the-actual-words part. It is necessary. It is not, on its own, sufficient. The teams that win in 2026 will have all five layers working, not heroic prompts compensating for missing layers below and above them.

## What this means if you are managing an AI build today

Three concrete moves for the next thirty days.

**Audit your existing prompts against the assistant-or-agent split.** For each non-trivial prompt in your codebase or your CLAUDE.md or your skills directory, classify it: is the model doing assistant work (single answer, you supervise the output) or agent work (multi-step, autonomous)? Then check the prompt shape. Are the assistant prompts precise? Are the agent prompts goal-shaped? Most teams have at least a few miscast prompts. Fixing them is fast.

**Strip the rituals.** Take a deep breath. Let's think step by step. You are an expert in. Find them in your prompts. Remove them. Re-run your evals (if you have them) or your standard tasks (if you don't) and observe. In my experience, most prompts get slightly better or stay neutral. None of mine got measurably worse from the removal. The free win is sitting on the table.

**Build the smallest possible eval harness.** Three test cases. JSON-shaped expected outputs. A script that runs them on demand. This is the highest-return investment most teams have not made. Without it, every prompt change is opinion. With it, every prompt change has a number.

After that, the bigger architectural moves (system prompt design, context engineering, MCP server selection) are still worth doing. But the audit, the strip, and the eval harness are the quickest path from "we are doing prompt engineering" to "we know our prompts are working."

## Closing

The framing that has been most useful to me in 2026 is the simple one: stop writing one kind of prompt, start writing two kinds. The OpenAI guidance and the Anthropic guidance both quietly say the same thing once you read them carefully. The split is real, the implications are operational, and the teams that internalize this in the next quarter will compound away from the ones still running 2024 prompt patterns against 2026 models.

There is still a craft. It just split into two crafts. You probably need both. Write the prompt that matches the job.

---

*Marco Kotrotsos, specializing in practical AI implementation for organizations ready to close the gap between AI hype and AI value. With 30 years of IT experience now focused purely on AI deployment, he works hands-on with companies to turn AI potential into measurable business outcomes.*

*This article is published in [Autocomplete](https://medium.com/autocomplete-real-world-ai), a Medium publication about real-world AI for practitioners and decision-makers. We're always looking for writers. If you're building with AI and have something worth sharing, reach out.*

*My free Substack newsletter, also called Autocomplete, can be found here: https://acdigest.substack.com.*

*Sources: OpenAI GPT-5.5 prompting guide (developers.openai.com), Claude Opus 4.7 prompting best practices, Anthropic effective-agents guidance, "OpenAI quietly told you to throw away your prompt stack" (AI Advances / Medium, 2026), 2026 State of Context Engineering survey.*
