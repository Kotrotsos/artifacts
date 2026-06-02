# Plan Mode Is the Feature: How to Stop Claude Code From Solving the Wrong Problem

![An isometric drafting table seen from above, a coral blueprint laid out and pinned, a small teal build rising from it on one side, parallel guide lines running from the plan into the structure](hero.png)

The most expensive thing Claude Code does is build the wrong thing quickly. It reads your prompt, decides it understands, and sprints. Ten minutes later you have two hundred lines of clean, well-tested code that solves a problem next to the one you actually had. Now you are not editing a sentence, you are unwinding a decision.

Everyone who has used a coding agent for real has watched this happen. The reflex is to blame the prompt, so people write longer prompts. That is not where the fix is. The teams I see getting reliable work out of Claude Code are not better at phrasing requests. They are disciplined about one thing: they refuse to let the agent write a line of code until the problem is pinned down. The mechanism for that refusal is plan mode, and it is the most underused feature in the tool.

One widely shared writeup put the success rate of unguided Claude Code runs at about a third. I would not treat that number as gospel, but it matches what I see. The gap between a third and reliable is almost never model quality. It is whether there was a plan before there was code.

**Three things to take away:**

- Plan mode exists to separate deciding what to build from building it. Those are different jobs, and letting them blur is the single biggest cause of fast, confident, wrong output.
- The official workflow is four phases in order: explore, plan, implement, commit. Most people skip the first two because skipping them feels faster. It is not. It just moves the cost to the end, where it is more expensive.
- The thing that makes a plan trustworthy is a check the agent can run without you: a test, a build, a screenshot diff. A plan tells Claude what to build. A check tells it when it is wrong. You want both.

## The failure mode, named

Claude stops when the work looks done. That sounds fine until you sit with it. "Looks done" is the only signal the agent has unless you give it another one. If the thing it built looks complete, it stops, regardless of whether complete and correct are the same thing here. You become the verification loop. Every wrong assumption waits for you to notice it, and you notice it after the code exists, which is the worst time.

This is why longer prompts do not save you. A longer prompt is still a description of a destination written by someone who has not seen the terrain. Claude has not read your files yet. You are both guessing. The fix is not to guess more precisely. It is to look before you commit to a route.

## The four phases, in order

The official Claude Code guidance is a sequence, and the order is the whole point: explore, then plan, then implement, then commit. Skipping the first two is the default mistake.

**Explore.** Enter plan mode first. In plan mode Claude can read files and answer questions but cannot make changes. Cycle into it with Shift+Tab before you ask for anything. Then point it at the actual code: "read /src/auth and explain how we handle sessions and login. Look at how we manage environment variables for secrets." No edits. You are getting the agent and yourself onto the same map before anyone decides where to go.

**Plan.** Now ask for the plan, while still in plan mode. "I want to add Google OAuth. What files need to change? What is the session flow? Write a plan." Claude produces an actual plan, grounded in the files it just read instead of the files it imagined. Press Ctrl+G to open that plan in your editor and change it directly. This is the highest-leverage minute in the whole session. You are editing a decision while it is still cheap, before a single line of code has been written against it.

**Implement.** Switch out of plan mode and let it build, against the plan it just made. "Implement the OAuth flow from your plan. Write tests for the callback handler, run the suite, fix any failures." The agent now has a route and a way to know it arrived.

**Commit.** When the check passes, "commit with a descriptive message and open a PR." The boring phase, and the one that goes smoothly precisely because the first three were not skipped.

![A four-phase flow: Explore reads the code, Plan writes the route, Implement builds against it, Commit ships, with a coral loop from Implement back to a verification check](four-phases.png)

Phases one and two look like this in practice. The agent reads your actual files, then hands back a plan grounded in them, with the plan-mode indicator showing it has not touched a thing yet.

![A Claude Code session in plan mode: the user asks it to read src/auth, Claude reports reading four files and explains the session flow, then produces a numbered Add Google OAuth plan, with a plan-mode bar showing ctrl+g to edit and shift+tab to implement](cc-s1.png)

## "Don't implement yet" is a real instruction

The single phrase that changes the most is the one that tells Claude not to start. Without something like "don't implement yet" or "write a plan first, do not change any code," the agent treats your description as a green light and begins. Plan mode enforces this structurally, but the habit matters even outside it. You are explicitly buying yourself a checkpoint between intent and action.

For anything larger than a small change, go further and let Claude interview you before it writes anything. Start with a minimal prompt and hand it the questions: "I want to build [short description]. Interview me in detail. Ask about technical implementation, edge cases, and tradeoffs. Do not ask obvious questions, dig into the hard parts I might not have considered. When we are done, write a complete spec to SPEC.md." It will surface decisions you had not made yet, which is the entire value. The decisions you have not made are the ones the agent will otherwise make for you, silently, in code.

Then start a fresh session to execute the spec. The new session has clean context aimed entirely at building, and you have a written artifact to check the result against. The official guidance puts it plainly: time spent making the spec precise pays off more than time spent watching the implementation. One circulating account described two hours on a twelve-step spec returning six to ten hours of implementation time. Treat the exact ratio as anecdote, but the direction is right and it is the direction that matters.

![A Claude Code session where the user asks it to interview them and write a spec instead of coding; Claude asks two decision questions about billing states and proration, then reports writing SPEC.md with 12 steps and 3 items out of scope](cc-s2.png)

## A plan without a check is half the system

A plan tells Claude what to build. It does not tell Claude when it has gone wrong. For that you need something the agent can run and read without you in the loop: a test suite, a build exit code, a linter, a script that diffs output against a fixture, a browser screenshot compared to a design. Give it one and the loop closes on its own. Claude builds, runs the check, reads the result, and iterates until it passes, instead of stopping at "looks done" and handing the verifying back to you.

The difference shows up in the prompt. "Implement a function that validates email addresses" leaves the agent to decide what correct means. "Write a validateEmail function. user@example.com is true, invalid is false, user@.com is false. Run the tests after implementing" gives it a fact to satisfy. The second one can fail honestly and fix itself. The first one can only look done.

![A Claude Code session running the test suite after implementing: two tests fail, Claude identifies the root cause as the unverified state param, fixes the handler, reruns, and all 24 tests pass](cc-s3.png)

For work you walk away from, harden the check. A `/goal` condition gets re-evaluated by a separate model after every turn, so the session keeps going until the condition holds. A Stop hook runs your check as a script and blocks the turn from ending until it passes. A verification subagent reviews the diff in a fresh context, seeing only the result and your criteria, not the reasoning that produced it, so the agent grading the work is not the one that wrote it. Each of these trades a little setup for a lot less of your attention. The plain prompt version works on any task today.

## Show it a real reference, not a description

The other habit that separates clean output from a guessing match is pointing Claude at an example instead of describing one. "Add a calendar widget" makes the agent invent your conventions. "Look at how existing widgets are implemented on the home page, HotDogWidget.php is a good example, follow that pattern to build a calendar widget" makes it match what you already have. Claude works better from a real reference than an abstract description, every time. Reference files with `@` so it reads them first, paste a screenshot when the target is visual, and when you report a bug, give the symptom, the likely location, and what fixed looks like: "login fails after session timeout, check token refresh in src/auth/, write a failing test that reproduces it, then fix it."

All of this is the same move as plan mode. You are replacing the agent's guess about your world with the actual contents of your world, before it acts on the guess.

## When not to plan

Planning has a cost, and pretending it does not is its own mistake. For a typo, a log line, a variable rename, a one-file change you could describe in a single sentence, skip it and just ask. The official rule of thumb is clean: if you could describe the diff in one sentence, you do not need a plan. Planning earns its overhead when you are unsure of the approach, when the change spans several files, or when you do not know the code you are about to touch. Forcing a ceremony onto a trivial fix is how people decide planning is a waste and stop doing it for the cases that need it.

## The real constraint underneath all of this

Every one of these habits is downstream of a single fact: Claude's context window fills fast, and performance degrades as it fills. Every file read, every command output, every failed attempt sits in that window. When it gets crowded, the agent starts forgetting your earlier instructions and making more mistakes. Plan mode helps here too, because exploring and deciding up front means the implementation phase runs on a clean, focused context instead of one already polluted by trial and error.

The corollary is that you should manage context aggressively. Run `/clear` between unrelated tasks. If you have corrected Claude twice on the same point, stop correcting, the context is now full of failed approaches that are dragging the next attempt down. Clear it and start fresh with a sharper prompt that bakes in what you just learned. A clean session with a better prompt beats a long session with accumulated corrections almost every time. Delegate big investigations to subagents so reading two hundred files happens in their context, not yours.

## The skill is restraint

None of this is about prompting harder. It is about resisting the pull to let a capable, eager agent start building before anyone has decided what to build. Plan mode is the tool that makes the restraint structural, but the underlying skill is older than the tool: look first, decide on paper, give yourself a way to know when you are wrong, then build.

The agent got very good at the building. The part that is still yours is making sure it is building the right thing. Plan mode is where you do that, and it is the difference between a third of your runs working and most of them working. The minute you spend in plan mode is the cheapest minute in the session. Spend it.

---

*Marco Kotrotsos, specializing in practical AI implementation for organizations ready to close the gap between AI hype and AI value. With 30 years of IT experience now focused purely on AI deployment, he works hands-on with companies to turn AI potential into measurable business outcomes.*

*This article is published in [Autocomplete](https://medium.com/autocomplete-real-world-ai), a Medium publication about real-world AI for practitioners and decision-makers. We're always looking for writers. If you're building with AI and have something worth sharing, reach out.*

*My free Substack newsletter, also called Autocomplete, can be found here: https://acdigest.substack.com.*

*My books on Amazon: [Claude Code for Everyone Else](https://www.amazon.com/dp/B0H35YY851) and [From Vibe to Production](https://www.amazon.com/dp/B0H34GK9VW).*
