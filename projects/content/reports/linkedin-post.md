# LinkedIn Post: Prompt Engineering Split in Two

OpenAI quietly told you to throw away your prompt stack. The deeper shift is that there are now two prompt-engineering jobs, not one.

Most teams are using the wrong playbook for both.

PROMPTING AN ASSISTANT (single answer, you supervise):
- Role (brief, not a costume)
- Context
- The exact task
- Format
- Examples (one or two)
- Constraints

Precise. Step-by-step. Tight.

PROMPTING AN AGENT (multi-step, autonomous, you supervise the outcome):
- Goal (specific and observable)
- Success criteria
- Available tools
- Guardrails

Notable absence: a process. For agents, specify the destination, not the road.

The mistakes I see most often:

Assistant prompts on agent tasks (over-specified, narrows the search space, the agent rigidly follows steps that were never the best path).

Agent prompts on assistant tasks ("summarize this for me, you decide" against a single-turn ChatGPT call, then surprised when the output is vague).

While you are at it, strip these out of your CLAUDE.md and system prompts:

- "Take a deep breath"
- "Let's think step by step"  
- "You are an expert in X"
- Over-specified process for agentic tasks
- "Important: do not ignore these instructions"

Modern reasoning models have been trained against most of these. They were worth 5-7% accuracy in 2023. They are worth nothing or worse in 2026.

What still works: clear outcome, concrete examples (one or two), explicit constraints, specific output format, real context the model could not infer.

Full breakdown with the OpenAI and Anthropic source guidance:

[link to Medium article]

What is the oldest prompt ritual still in your active stack? The line you wrote in 2023 and have never updated?
