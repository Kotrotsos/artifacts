# Stop Optimizing for Engineer Speed. Start Optimizing for Agent Speed.

*A real case study from PFF. 25x more deploys, 10x output, customer satisfaction up from 7.5 to 8.6. Two months. Two engineers. Scrum did not survive.*

![Two side-by-side isometric scenes showing a single bottleneck engineer in the before scene and a parallel grid of agent-powered modules in the after scene](hero.png)

I watched a conference talk this week that I want every engineering lead on this list to see. The speaker leads the post-engineering org at PFF, a 200-person sports data company that powers fantasy football and serves NFL and NCAA teams. They ran a case study from January to March 2026 with two engineers, ended up with 25x more deploys per day, 10x output, and a customer satisfaction lift from a 7 to 7.5 baseline up to 8.6 out of 10.

The reframe that drove the whole thing is the part I keep coming back to. The instinct in engineering for decades has been to ask how we help engineers go faster. The PFF team asked a different question. How do we make the agents go faster?

That sentence is small until you sit with it. It changes what you optimize for, what processes you keep, what tooling you build, and which engineers thrive on your team.

This newsletter is the operational walkthrough of what PFF actually did. Numbers, process changes, what got cut, what survived. Hit reply if you want me to look at your specific setup.

**Three things to know up front:**

The reframe that drove the case study was simple but consequential. Stop asking how to make engineers faster. Start asking how to make agents faster. The agile manifesto, the foosball tables, the sleeping pods, the whole optimized-for-engineer-productivity culture exists because engineers were the bottleneck. They are not the bottleneck anymore. Optimizing for the new bottleneck (agent throughput) produced a 25x deployment increase and a 10x output increase at PFF within two months.

The cuts were as important as the additions. Scrum did not survive contact with this approach. Sprint planning, daily standups, sprint refinement, and the project manager role all got cut at PFF. What remained: half-hour huddles every other day, customer satisfaction surveys as the primary success metric, and deployment metrics. The pipeline replaced the ceremony.

The actual workflow at PFF is four steps: spec, lightweight design document, auto-generated tickets, auto-generated PRs. Followed by auto-deploy to staging and an autonomous QA agent that checks acceptance criteria. The next step on the roadmap is closing the loop so a second agent fixes the QA failures and re-opens PRs, making the whole system self-healing.

## The reframe in one sentence

The speaker opened with the question that drove the experiment. The instinct in engineering for decades has been "how do we help engineers output more." That instinct shaped agile, software craftsmanship, the elaborate physical perks, and the entire culture of engineer-centric optimization. The premise underneath all of it was that engineers are the bottleneck.

The reframed question at PFF was: how do we make the agents quicker?

Once you accept that engineers are no longer the only bottleneck, every ceremony, perk, and tool built around "engineer is the constraint" becomes worth re-examining. Most of it did not survive.

## The numbers

The two-engineer tiger team versus the 10-engineer team produced the following:

- 25x more deploys per day (the tiger team deployed five times a day on average; the larger team deployed roughly once every five days)
- 10x output, measured as tickets shipped weighted by code complexity
- Customer satisfaction score climbed from a 7 to 7.5 baseline up to 8.6 out of 10 in a statistically significant survey
- Features that pre-AI would have been scoped at four months shipped in under two
- One of the two engineers became unblocked under a month in and started shipping additional features in parallel

![A 2 by 2 grid of stat cards showing 25x deploys, 10x output, 8.6 customer satisfaction, and 2 months versus 4 months ship time](diagram-3-results.png)

The honest caveat the speaker offered: small teams are always faster than large ones. Part of the multiplier is just team size. But the tiger team still had to coordinate every deploy with the larger team, and the 10x output gap was measured by tickets weighted by complexity, not raw deploy count.

The compounding bit is the part that gets understated. One engineer unblocked early went on to ship additional features in parallel while the other engineer continued on the main work. The throughput stacks. The output graph after that is geometric, not linear.

## What got cut

Scrum did not survive.

![Two columns showing what got cut from the process (sprint planning, daily standups, sprint refinement, project manager) versus what was kept (huddles, customer surveys, deployment metrics, retrospectives)](diagram-2-survived.png)

The PFF team removed sprint planning. Estimation games make no difference when the agent is doing the work.

They removed daily standups. Tickets auto-update based on PR status. If a PR opens, the ticket moves to in-progress. If it goes to review, the ticket updates. If it merges, the ticket closes. The standup as a status broadcast became redundant because the system already broadcasts.

They removed sprint refinement. Refinement now happens earlier, inside the spec and the lightweight design document phase, before tickets even exist.

They removed the project manager role. Without sprint planning, standups, refinement, or estimation games, the coordination work that role used to absorb is mostly gone.

What survived: huddles every other day, half an hour, with engineers plus product plus design science. Customer satisfaction surveys as the primary success measurement. Deployment metrics. Retrospectives. That is roughly it.

## The new workflow

Four autonomous stages plus two verification stages.

![A horizontal flow showing the autonomous workflow: spec, lightweight design document, tickets, pull request, staging deploy, QA agent, with a self-healing feedback loop from QA back to tickets](diagram-1-workflow.png)

**Stage 1: Spec.** Engineer describes the feature. Agent interviews them, surfacing the questions traditional product discovery would have surfaced. Output is a spec with the actual requirements rather than the simplified version that usually survives the telephone game from stakeholder to PM to engineer.

**Stage 2: Lightweight design document.** This is the part I want to highlight. PFF has a Skill that generates the LDD. The Skill is calibrated against the LDDs the team has written before, so every new design document is in the same shape and architectural style as everything else in the codebase. This prevents the agent from drifting toward generic patterns. The LDD is a discipline encoded as a Skill, not a Claude Code idiosyncrasy.

**Stage 3: Tickets.** Once the LDD passes review, tickets are auto-generated, structured so none of them block each other. Where blocking dependencies exist, they are flagged explicitly.

**Stage 4: Pull requests.** Agent picks up each ticket and writes the PR.

**Stage 5: Auto-deploy to staging.** Every merged PR triggers a deployment to staging without human intervention.

**Stage 6: QA agent.** After the staging deploy completes, a QA agent looks at the tickets that went into the deploy, reads the acceptance criteria for each, and runs verification. Pass: greenlit. Fail: failures flagged with specific pointers to what is missing.

The piece PFF has not yet built but is on the roadmap is the self-healing loop. The next agent in line would look at failed acceptance criteria, generate the fix, and open a new PR automatically. Once that wires up, the system runs end-to-end without human intervention for routine cases, leaving humans for the cases that need judgment.

## Where humans still matter

**Security.** Agents take shortcuts. Anything with a security implication gets human review.

**Product feel.** Every product can now be built in an hour by anyone. The brand and product feel of the output is what separates "looks like every other AI-generated app" from "looks like our product."

**Scale and engineering complexity.** Agents will produce a thousand lines of code where two hundred would do. The LDD is where you intervene. A tight, prescriptive design document at the start prevents sprawl later. Most of the engineering judgment now lives in writing a precise LDD, not in writing the code afterward.

## The cultural transition

The speaker said it directly: not everyone can drive a sports car. The engineering org transition is going to be hard for some engineers, and pretending otherwise is dishonest.

The engineers who thrive in this model are curious. When they hit something they do not understand, they spend time figuring out how it was built. They are comfortable with the agent doing the typing and themselves doing the orienting.

The engineers who struggle are the ones who need prescriptive specs. They were great at executing on detailed tickets. The detailed tickets are no longer how the work flows. The work flows through specs and LDDs and judgment calls about whether the agent's output matches the codebase style. That is a different skill, and not everyone has it or wants it.

For an engineering leader, the difficult truth is that the team you start with is not the team you end with. Some engineers will lean in and accelerate. Some will resist or struggle. Pretending the transition will be uniform across the team is the kind of optimism that costs you twelve months of misaligned hiring.

## How PFF actually started

The implementation pattern is worth copying because it is not a moonshot. It is incremental.

**Pick the engineers with the best system knowledge.** Every engineering team has one or two people whom others say "you should talk to them" when something is genuinely hard. Those are the engineers to put on this. They know the codebase well enough to spot when the agent is drifting, and they have the curiosity to figure out what the agent did that they did not understand.

**Go slowly and phase in.** The temptation to roll out coding assistants to the whole team on day one is strong and almost always wrong. PFF did proof-of-concept work in non-critical systems for two months before the tiger team ever touched the 100-million-page-view product.

**Experiment in non-critical systems first.** The speaker spent November and December building small features that did not get much traffic. When a bug or mistake happened, it did not matter. By the time they moved to high-traffic systems in January, the operational patterns were proven.

**Encode your patterns as Skills.** Every reusable engineering pattern in your codebase (your API style, your branch naming, your feature flag conventions, your trunk-based development discipline) gets encoded as a Skill. The agent then has access to your team's calibrated practices, not the average practices it picked up in training.

**Get the guardrails working before going autonomous.** The speaker was explicit: do not turn on the autonomous loop until the deterministic verification (tests, feature flags, deployment checks) is reliable. Otherwise the self-healing loop will heal its way into a worse state.

One opinion from the talk worth quoting: be skeptical of Skills with strong software design opinions that contradict your team's. They will pull your codebase toward someone else's defaults and create friction you do not want.

## What you should not do

A few specific anti-patterns the speaker called out.

**Do not roll this out to everyone at the same time.** The most common reason these initiatives fail in larger orgs is that they get treated as a deployment, not a transition. A demo hackathon plus a license rollout to the whole team is an event, not a strategy.

**Do not assume your culture transfers.** Every engineering org is different. The PFF playbook works because PFF has 20 engineers, a clear codebase, and the leadership backing to dismantle ceremonies that were not working. A 1,000-person org cannot run the same playbook unchanged. The principles transfer; the specific operational changes need re-derivation.

**Do not be too conservative.** This is the harder advice. The speaker said they felt a few months behind their peers in the broader industry even with what they had built. The compounding-impact logic is real. A few months behind today is six months behind in three months and twelve months behind by Q3. The competition is not slowing down.

## What I would do this month

If you run an engineering org and want to start moving toward this model, four concrete moves in the next thirty days.

**Run the Scrum audit.** For each current ceremony (planning, standup, refinement, retro), ask what specific value it provides and whether that value still applies when the bottleneck is agent throughput, not engineer coordination. Most teams will find two or three ceremonies still earning their place and the rest is inertia.

**Build the LDD Skill.** This is the single highest-return move. A Skill that generates lightweight design documents in your team's specific shape, calibrated against your existing LDDs, prevents the agent from sprawling on every feature. Without it, every PR is generic.

**Pick the two-engineer tiger team.** Strongest system knowledge, paired against a non-critical system, six to eight weeks, explicit license to question every ceremony. The patterns they discover are what scales.

**Survey customers, not stand-ups.** The number that tells you the work was worth doing is whether customers noticed. PFF moved from 7 to 7.5 baseline up to 8.6 in two months. That is the number worth chasing.

Hit reply if you want a closer look at your specific setup. I read everything.

## What is coming next

The next two newsletter issues will continue this thread:

- **Building your LDD Skill.** A walkthrough of how to encode your team's architectural patterns as a Skill that calibrates new designs against your existing codebase.
- **The Scrum audit.** A concrete pattern for going ceremony by ceremony through your current process and deciding what survives in an agent-first org.

**One open question for you, if you are willing to share in the comments:** which engineering ceremony are you most reluctant to cut, and what is it actually doing for you that the pipeline cannot do better?

Until next week.

Marco

---

*Autocomplete is a free weekly newsletter on practical AI implementation. If you are not subscribed yet, the button above this paragraph is the easiest way to fix that. If you are already subscribed, forwarding this to one engineering lead who would benefit is the single best thing you can do for me.*
