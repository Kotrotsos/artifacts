# From Autocomplete to Delegation: Restructuring Your Workflow for Coding Agents

*The unit of work moved from the line to the issue. Your hands moved from the keyboard to the brief and the review. Here is how to reorganize your actual day around it, with the issue template, the review gates, and a one-week plan to switch.*

![An isometric scene: a person-shaped abstract form at a small control desk handing off labeled task cards to a row of teal worker units that build in parallel, with finished work returning to a review tray on the desk](hero.png)

A year ago my workday was a long sequence of small completions. Write the function. Fix the line the autocomplete got slightly wrong. Tab, tab, tab. The tool finished my sentences and I finished the work, one keystroke at a time. I was faster than I had been without it, but the shape of the day was unchanged: me, at the keyboard, producing code by hand with a faster autocomplete bolted on.

That shape broke this year. When I built my last product, a full SaaS, the work did not arrive as a stream of lines to complete. It arrived as a queue of issues to hand off. Add the invitation system. Migrate the auth library. Refactor the results export. I wrote each one as a brief, handed it to an agent, and the agent came back with a pull request. Seventy-four of them, over about a week. My hands were barely on the keyboard. They were on the brief and the review.

That is the shift, and it has a name worth using: from completion to delegation. Autocomplete finishes your sentence. An agent finishes your task. The skill you are building is no longer typing faster, it is handing off well and checking honestly. This piece is how to restructure your day around that, concretely.

**Three things to take away:**

- The unit of work changed from the line to the issue. Your day stops being a stream of keystrokes and becomes a loop of three moves: write a delegable brief, hand it off, review what comes back. Most of your value moves to the first and third move.
- A delegable issue is an acceptance test written in prose. If a colleague could not check the result against your brief without asking you a question, an agent cannot either. The quality of the handoff is the quality of the result, because nobody is in the loop to correct vagueness mid-run.
- Trust comes from gates, not from faith. A check that runs without you, a fresh reviewer that only sees the diff, a scope check, and you reading the change at altitude. The gates are what let you delegate work you did not watch get made.

## The shift: completion versus delegation

The honest before-and-after looks like this.

**Completion, the old loop.** You hold the task in your head. You write a line, the tool suggests the next few, you accept or correct, you move to the next line. You are the one going, the whole time. The tool makes each step faster, but you take every step. Your throughput is capped by how fast you can move through the work yourself.

**Delegation, the new loop.** You hold the task in your head only long enough to write it down clearly. Then you hand it to an agent that does the steps, and you move on to writing the next brief or reviewing the last result. You are not the one going. You are the one deciding what should be built and checking that it was. Your throughput is capped by how many briefs you can write and reviews you can do, which is a much higher ceiling, because several agents can be working while you are thinking.

![A before-and-after diagram. Completion: a single human figure stepping through every line of the work, one keystroke at a time, throughput capped by their own speed. Delegation: the human writes briefs and reviews results while three agents build in parallel, throughput capped by briefs-and-reviews, not typing](diagram-shift.png)

The trap in the new loop is the middle. Your instinct, trained by years of completion, is to hover over the agent, watch it work, and nudge it line by line. That is nostalgia for the old job, and it throws away the entire gain. The work is at the two ends now: writing the brief well enough that you can walk away, and checking the result well enough that you can trust it. If you find yourself babysitting the middle, the fix is almost always a sharper brief, not closer supervision.

## Which tasks to hand off first

You do not flip your whole workflow on day one. You start with the tasks that delegate cleanly, build the habit, and expand. The tasks that hand off well share three properties: they are bounded, they are verifiable, and they are low on taste.

**Bounded** means the work has clear edges. A migration touches a known set of files. A test covers a known function. A refactor has a defined start and end state. Open-ended exploration is not bounded and does not delegate well yet.

**Verifiable** means there is an objective way to know it worked. Tests pass. The build is green. The output matches a fixture. If the only way to judge the result is your taste, the agent has nothing to check itself against.

**Low on taste** means the right answer is not a judgment call. There is a correct migration; there is no correct answer to "make the dashboard feel more premium."

Run those three filters and the first-wave tasks pick themselves:

- **Tests.** Writing tests for existing code is the canonical first handoff. Bounded, verifiable (the tests run), and low on taste. It is also the handoff that makes every future handoff safer, because it builds the verification you will lean on.
- **Migrations and dependency bumps.** Move from one library to another, upgrade a framework version, rename a model across the codebase. Mechanical, well-defined, and checkable against a passing suite.
- **Refactors with a clear target.** Extract a module, split a fat function, change a pattern across files. The end state is specifiable and the tests guard the behavior.
- **Cleanup and documentation.** Dead-code removal, comment passes, README updates, lint fixes. Low stakes, easy to verify, good practice for the loop.

![A map of which tasks to delegate first. A quadrant with bounded-and-verifiable on one axis and low-taste on the other. In the strong-handoff corner: tests, migrations, refactors with a target, cleanup, dependency bumps. In the keep-it-yourself corner: architecture decisions, ambiguous product calls, security-critical design, anything judged purely by taste](diagram-handoff-map.png)

What to keep, for now: architecture decisions you will live with for months, ambiguous product calls, security-critical design, and anything judged purely by taste. Not because an agent cannot touch them, but because they fail the three filters, so the handoff is harder and the review has to be much heavier. Delegate those last, and review them like a hawk when you do.

## Writing an issue an agent can execute

This is the highest-leverage skill in the new workflow, and it is mostly a writing skill. A delegable issue is an acceptance test in prose. It says what done means in terms someone else could check, not what to type.

The difference is stark. Here is a brief that will waste an agent's run:

> Fix the subscription flow, it is buggy.

And here is the same task, written so an agent can execute and check itself:

> In `src/billing/subscribe.ts`, a user who cancels still gets charged at the next period. Done means: cancellation sets `cancel_at_period_end`, no charge fires after cancellation, a new test in `subscribe.test.ts` covers the cancel-then-renew case, and the existing suite still passes. Do not change the public API of `BillingService`. Write the plan before the code.

The second one names the file, states the bug precisely, defines done as a set of checkable conditions, sets a constraint (do not change the public API), and demands a plan first. An agent can run that to completion and know whether it succeeded. So could a new hire. That is the test: if a competent stranger could check the result against your brief without asking you a single question, it is a real brief. If they would have to come back and ask what you meant, you wrote a mood, not a brief.

![A dark editor screenshot showing the anatomy of a delegable issue, with five labeled parts: the context and the precise problem with a file path, the done-means acceptance conditions as a checklist, a constraint of what not to change, the verification that tests must pass, and a do-not-implement-yet plan-first line](screenshot-issue.png)

The anatomy that works, every time:

- **Context and the precise problem,** with file paths. Point at where the work lives.
- **Done means,** as a checklist of conditions a machine or a colleague can verify.
- **Constraints,** the must-nots. What the change should not touch or break.
- **Verification,** the check the agent should run before calling it finished. Usually the tests.
- **Plan first.** Tell it not to implement until it has written the approach, so you get a checkpoint before any code exists.

Spend your time here. A brief that takes you five minutes to sharpen routinely saves an hour of reviewing and re-running a result that solved the wrong problem.

## Review gates: how to trust agent-authored changes

You delegated work you did not watch get made. Trust cannot come from having seen it happen, because you did not. It comes from gates: a sequence of checks each change passes before you believe it. The more of these you automate, the more you can delegate without losing sleep.

![A funnel of four review gates an agent-authored change passes through before merge. Gate one: an automated check that runs without you, tests and build and lint green. Gate two: a fresh-context reviewer that sees only the diff and the brief, told to find gaps. Gate three: a scope check, did it touch only what it should. Gate four: a human read at altitude, blast radius not line-by-line. Only changes that clear all four reach merge](diagram-gates.png)

**Gate one: a check that runs without you.** Tests, a build, a linter, a type check. The agent should run these itself and show you they pass before it hands the work over. This is the gate that does the most work, because it closes the loop without your attention. If you delegate nothing else, delegate the writing of tests first, so this gate exists.

**Gate two: a fresh-context reviewer.** Have a second agent, in a clean context that did not write the code, review the diff against the brief, with one instruction: find where this misses the bar, not where it is stylistically different. The agent that produced the change is the worst judge of it; a fresh one with no stake reviews it honestly. This catches the plausible-but-wrong changes that pass the tests but miss the intent.

**Gate three: a scope check.** Ask, in code or by eye, whether the change touched only what it should have. Agents sometimes fix the bug and also reformat three unrelated files. A diff that sprawls past the brief is a signal to look harder, even when the tests are green.

**Gate four: you, reading at altitude.** Not line by line, which does not scale to the volume delegation produces. You read for blast radius: what did this touch, what does it now depend on, what could it break that the tests do not cover, how confident am I to ship it. This is the review that stays human, and it is faster than old line-by-line review because the first three gates already filtered the obvious problems.

The point of the gates is that trust becomes a process instead of a feeling. You are not deciding whether you trust the agent. You are deciding whether the change cleared four checks, which is a question you can answer at the volume the new workflow produces.

## A one-week plan to switch

You do not need to rebuild your habits all at once. Here is a five-day plan that moves you from completion to delegation without betting anything important on it.

**Monday: pick three and write them.** Choose three tasks from your real backlog that pass the three filters, bounded, verifiable, low on taste. A test suite for an untested module, a small migration, a contained refactor. Write each one as a proper delegable brief using the anatomy above. Do not delegate yet. Just practice writing the briefs, and notice how much you have to think to make "done" checkable.

**Tuesday: delegate one, gate it by hand.** Hand off the first brief. When the agent returns, run it through the four gates manually: confirm the tests pass, read the diff against your brief looking for gaps, check the scope, then read at altitude. Merge it or send it back with a sharper brief. Notice where the brief was vague, because the review will show you.

**Wednesday: delegate two in parallel.** Hand off the other two briefs at the same time and work on something else while they run. This is the first taste of the real gain: two pieces of work happening that you did not type. Review both through the gates.

**Thursday: automate a gate.** Add the fresh-context reviewer. Before you read a change yourself, have a second agent review the diff against the brief and report gaps. Compare its findings to your own read. You are building the gate that lets you scale past what you can personally review.

**Friday: retro and template.** Look back at the week. Which handoffs came back clean, which needed a second pass, and what was different about the briefs? Turn what you learned into a reusable issue template, the five-part anatomy filled with your project's specifics, the files that matter, the test command, the constraints you keep repeating. That template is your delegation workflow, written down.

By the end of the week you will have shipped real work you did not type, built the gates that make it trustworthy, and have a repeatable way to write the briefs. That is the whole workflow in miniature, and it scales from there.

## The shape of the new day

The move from completion to delegation is not about working faster at the keyboard. It is about doing less at the keyboard and more at the two ends, deciding what should be built and confirming that it was. Your output goes up not because you type quicker but because you run several pieces of work in parallel that you never typed at all, each one specified clearly enough to hand off and gated tightly enough to trust.

The keystrokes were never the valuable part. They were just the only way to get the work out of your head and into the codebase. That bottleneck is gone. What is left is the part that was always the real work: knowing exactly what you want, saying it precisely, and checking honestly whether you got it. Write the brief, hand it off, hold the gate. That is the day now.

---

*Marco Kotrotsos, specializing in practical AI implementation for organizations ready to close the gap between AI hype and AI value. With 30 years of IT experience now focused purely on AI deployment, he works hands-on with companies to turn AI potential into measurable business outcomes.*

*This article is published in [Autocomplete](https://medium.com/autocomplete-real-world-ai), a Medium publication about real-world AI for practitioners and decision-makers. We're always looking for writers. If you're building with AI and have something worth sharing, reach out.*

*My free Substack newsletter, also called Autocomplete, can be found here: https://acdigest.substack.com.*

*My books on Amazon: [Claude Code for Everyone Else](https://www.amazon.com/dp/B0H35YY851) and [From Vibe to Production](https://www.amazon.com/dp/B0H34GK9VW).*
