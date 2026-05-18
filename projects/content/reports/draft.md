# The Post-Engineering Org: How PFF Got 25x More Deploys by Optimizing for Agents, Not Engineers

*A real case study from PFF, a 200-person sports data company. Two engineers, two months, 25x deployment frequency, 10x output, and a customer satisfaction score that climbed from 7.5 to 8.6. The lessons are operational, not aspirational.*

![Two side-by-side isometric scenes showing a single bottleneck engineer in the before scene and a parallel grid of agent-powered modules in the after scene](hero.png)

**Key takeaways:**

- The reframe that drove the case study was simple but consequential: stop asking how to make engineers faster, start asking how to make agents faster. The agile manifesto, the foosball tables, the sleeping pods, the whole optimized-for-engineer-productivity culture exists because engineers were the bottleneck. They are not the bottleneck anymore. Optimizing for the new bottleneck (agent throughput) produced a 25x deployment increase and a 10x output increase at PFF within two months.
- The cuts were as important as the additions. Scrum did not survive contact with this approach. Sprint planning, daily standups, sprint refinement, and the project manager role all got cut at PFF. What remained: half-hour huddles every other day, customer satisfaction surveys as the primary success metric, and deployment metrics. The pipeline replaced the ceremony.
- The actual workflow at PFF is four steps: spec, lightweight design document, auto-generated tickets, auto-generated PRs. Followed by auto-deploy to staging and an autonomous QA agent that checks acceptance criteria. The next step on the roadmap is closing the loop so a second agent fixes the QA failures and re-opens PRs, making the whole system self-healing.

---

I watched a conference talk this week that I want more practitioners to see. The speaker leads the post-engineering org at PFF, a 200-person sports data company that serves NFL and NCAA teams plus a consumer arm doing fantasy football and sports betting. PFF has 100 million page views a year and runs 9 million drafts annually. It is not a hobbyist setup. The case study they ran from January to March 2026 is one of the most specific, numbers-backed reorganization stories I have seen in this entire AI-tooling wave.

The headline result is 25x more deploys per day in a two-engineer tiger team compared to their normal 10-engineer team. The honest caveat the speaker offered is also worth keeping in mind: small teams are always faster than large ones, so part of the multiplier is just from team size. But the tiger team still had to coordinate every deploy with the larger team, and the output gap measured by tickets weighted by complexity was still 10x.

The shift that produced these numbers is the one I want to walk through. Not the headline, the operational specifics.

## The reframe

The speaker opened with the question that drove the whole experiment. The instinct in engineering for decades has been "how do we help engineers output more." That instinct shaped agile, software craftsmanship, the elaborate physical perks (foosball tables, sleeping pods, premium snacks), and the entire culture of engineer-centric optimization. The premise underneath all of it was that engineers are the bottleneck.

The reframed question at PFF was: how do we make the agents quicker?

That sentence sounds small until you sit with it. It changes what you optimize for, what processes you keep, what tooling you build, and which engineers thrive on the team. It changes the org chart. Once you accept that engineers are no longer the only bottleneck, every ceremony, perk, and tool that was built around "engineer is the constraint" becomes worth re-examining.

This is what PFF re-examined. Most of it did not survive.

## The numbers

The two-engineer tiger team versus the 10-engineer team produced the following:

- 25x more deploys per day (the tiger team deployed five times a day on average; the larger team deployed roughly once every five days)
- 10x output, measured as tickets shipped weighted by code complexity
- Customer satisfaction score climbed from a 7 to 7.5 baseline up to 8.6 out of 10 in a statistically significant survey
- Features that pre-AI would have been scoped at four months shipped in under two
- One of the two engineers became unblocked under a month in and started shipping additional features in parallel, while the other engineer continued on the main work, producing a compounding effect

![A 2 by 2 grid of stat cards showing 25x deploys, 10x output, 8.6 customer satisfaction, and 2 months versus 4 months ship time](diagram-3-results.png)

The compounding bit is the part that gets understated. The benefit was not just speed on the main feature. The benefit was that one engineer unblocked early could go pick up additional work, and that work also moved fast, and the throughput stacked. The pre-AI baseline had both engineers blocked on the same critical path for the whole estimated four months. The new baseline freed one of them after thirty days. The output graph after that is geometric, not linear.

## What got cut

Scrum did not survive.

![Two columns showing what got cut from the process (sprint planning, daily standups, sprint refinement, project manager) versus what was kept (huddles, customer surveys, deployment metrics, retrospectives)](diagram-2-survived.png)

The PFF team removed sprint planning entirely. The reasoning is that estimation games make no difference when the agent is doing the work. The hour spent debating ticket points was not informing real decisions.

They removed daily standups. Tickets auto-update based on PR status. If a PR is open, the ticket moves to in-progress automatically. If it goes to review, the ticket updates. If it merges, the ticket closes. The standup as a status broadcast became redundant because the system already broadcasts.

They removed sprint refinement. Refinement now happens earlier in the workflow, inside the spec and the lightweight design document phase, before tickets even exist.

They removed the project manager role. Without sprint planning, standups, refinement, or estimation games, the coordination work that role used to absorb is mostly gone or has been redistributed.

What survived: huddles every other day, half an hour or so, with engineers plus someone from product plus someone from the design science team. Customer satisfaction surveys as the primary success measurement. Deployment metrics. Retrospectives. That is roughly it.

The throughline is that the process layer optimized for human coordination got compressed dramatically. The process layer optimized for fast feedback (huddles, customer surveys, deployment frequency) was kept.

## The new workflow

The actual flow at PFF runs in four autonomous stages plus two verification stages.

![A horizontal flow showing the autonomous workflow: spec, lightweight design document, tickets, pull request, staging deploy, QA agent, with a self-healing feedback loop from QA back to tickets](diagram-1-workflow.png)

**Stage 1: Spec.** The engineer describes the feature. The agent interviews them, surfacing the questions that traditional product-discovery would have surfaced. The output is a spec that contains the actual requirements rather than the simplified version that usually survives the telephone game from stakeholder to PM to engineer.

**Stage 2: Lightweight design document.** This is the part I want to highlight, because it is the architecturally important step. PFF has a Skill that generates the LDD. The Skill is calibrated against the LDDs the team has written before, so every new design document is in the same shape and architectural style as everything else in the codebase. This is how they prevent the agent from drifting toward generic patterns. The LDD is a discipline encoded as a Skill, not a Claude Code idiosyncrasy.

**Stage 3: Tickets.** Once the LDD passes review, tickets are auto-generated, structured so none of them block each other. Where blocking dependencies exist, they are flagged explicitly in the ticket metadata.

**Stage 4: Pull requests.** The agent picks up each ticket and writes the PR.

**Stage 5: Auto-deploy to staging.** Every merged PR triggers a deployment to staging without human intervention.

**Stage 6: QA agent.** After the staging deploy completes, a QA agent looks at the tickets that went into the deploy, reads the acceptance criteria for each, and runs verification. If everything passes, the build is greenlit. If criteria fail, the failures are flagged with specific pointers to what is missing.

The piece PFF has not yet built but is on the roadmap is the self-healing loop. The next agent in line would look at the failed acceptance criteria, generate the fix, and open a new PR automatically. Once that is wired up, the system runs end-to-end without human intervention for the routine cases, leaving humans to focus on the cases that actually need judgment.

## Where humans still matter

The speaker was specific about where the people stay in the loop, and worth listening to because the answer is not "everywhere by default."

**Security.** Agents take shortcuts. Anything with a security implication gets human review. This is the most important guardrail.

**Product feel.** Every product can now be built in an hour by anyone. The brand and product feel of the output is what separates "looks like every other AI-generated app" from "looks like our product." Engineers stay engaged on whether the output feels right, on whether it matches the rest of the company's product surface.

**Scale and engineering complexity.** Agents will produce a thousand lines of code where two hundred would do. They will over-engineer when not constrained. The LDD is where you intervene. A tight, prescriptive design document at the start prevents the agent from sprawling later. Most of the engineering judgment now lives in writing a precise LDD, not in writing the code afterward.

## The cultural transition

This is the section most posts about AI-engineering avoid because it is uncomfortable.

The speaker said it directly: not everyone can drive a sports car. The engineering org and the new era is going to be hard for some engineers, and pretending otherwise is dishonest.

The engineers who thrive in this model are curious. When they hit something they do not understand, they spend a little time figuring out how it was built. They are comfortable with the agent doing the typing and themselves doing the orienting. They are comfortable with abstraction at a higher level than the lines of code.

The engineers who struggle are the ones who need prescriptive specs. They were great at executing on detailed tickets. The detailed tickets are no longer how the work flows. The work now flows through specs and LDDs and judgment calls about whether the agent's output matches the codebase style. That is a different skill, and not everyone has it or wants it.

For an engineering leader reading this, the difficult truth is that the team you start with is not the team you end with. Some engineers will lean into this and accelerate. Some will resist or struggle. Pretending the transition will be uniform across the team is the kind of optimism that costs you twelve months of misaligned hiring and disappointed performance reviews.

## How they actually started

The implementation pattern PFF used is worth copying because it is not a moonshot. It is incremental.

**Pick the engineers with the best system knowledge.** Every engineering team has one or two people whom others say "you should talk to them" when something is genuinely hard. Those are the engineers to put on this. They know the codebase well enough to spot when the agent is drifting, and they have the curiosity to figure out what the agent did that they did not understand.

**Go slowly and phase in.** The temptation to roll out coding assistants to the whole team on day one is strong and almost always wrong. PFF did proof-of-concept work in non-critical systems for two months before the tiger team ever touched the 100-million-page-view product.

**Experiment in non-critical systems first.** The speaker spent November and December building small features that did not get much traffic. When a bug or mistake happened, it did not matter. By the time they moved to high-traffic systems in January, the operational patterns were proven.

**Encode your patterns as Skills.** Every reusable engineering pattern in your codebase (your API style, your branch naming, your feature flag conventions, your trunk-based development discipline) gets encoded as a Skill. The agent then has access to your team's calibrated practices, not the average practices it picked up in training.

**Get the guardrails working before going autonomous.** The speaker was explicit: do not turn on the autonomous loop until the deterministic verification (tests, feature flags, deployment checks) is reliable. Otherwise the self-healing loop will heal its way into a worse state.

The speaker had a strong opinion about consuming other people's Skills, worth quoting: be skeptical of Skills with strong software design opinions that contradict your team's. They will pull your codebase toward someone else's defaults and create friction you do not want.

## What you should not do

A few specific anti-patterns the speaker called out.

**Do not roll this out to everyone at the same time.** The most common reason these initiatives fail in larger orgs is that they get treated as a deployment, not a transition. A demo hackathon plus a license rollout to the whole team is an event, not a strategy.

**Do not assume your culture transfers.** Every engineering org is different. The speaker's playbook worked at PFF because PFF has 20 engineers, a clear codebase, and the leadership backing to dismantle ceremonies that were not working. A 1,000-person org cannot run the same playbook unchanged. The principles transfer; the specific operational changes need re-derivation.

**Do not be too conservative.** This is the harder advice. The case study speaker said they felt a few months behind their peers in the broader industry even with what they had built. The compounding-impact logic is real. A few months behind today is six months behind in three months and twelve months behind by Q3. The competition is not slowing down.

## What this means if you are running an engineering org in 2026

A few practical takeaways.

The Scrum review is the first concrete move. Audit your current ceremonies. For each one (planning, standup, refinement, retro, sprint cadence), ask what specific value it provides and whether that value still applies when the bottleneck is agent throughput, not engineer coordination. Most teams will find that two or three ceremonies are still earning their place and the rest are inertia.

The Skills layer is the architectural investment. The single highest-return move at PFF is the LDD Skill that calibrates new designs against the existing codebase. It is a Skill specific to PFF's patterns, voice, and conventions, not a generic template. Building your equivalent is the highest-return engineering work most teams should be doing right now. Without it, every PR from the agent is generic.

The two-engineer tiger team is the right experimental unit. Not the whole team. Not one engineer alone. Two engineers with strong system knowledge, paired against a non-critical system, with explicit license to question every ceremony, for six to eight weeks. The patterns they discover are what scales.

The customer satisfaction survey is the right success metric. Not deployment frequency by itself. Not lines of code. Not PR count. The number that tells you the work was worth doing is whether the customers noticed. PFF moved from a 7 to 7.5 baseline up to 8.6 in two months. That is the number worth chasing.

## Closing

PFF is not a moonshot company. It is a 200-person sports data shop with 20 engineers and a serious customer base. The case study they ran is reproducible. The specifics matter: the LDD Skill, the auto-generated tickets, the QA agent, the cut of sprint planning, the huddle cadence. None of these are speculative. They are operational decisions that produced a 25x deployment increase and a customer satisfaction lift in two months.

The reframe that produced all of this fits in one sentence. Stop optimizing for engineer speed. Start optimizing for agent throughput. The rest is the work of redesigning your process to actually live by that sentence.

The engineering orgs that internalize this in 2026 will compound away from the ones that do not. The case study is the proof. The playbook is here. The next question is which part of yours gets re-examined first.

---

*Marco Kotrotsos, specializing in practical AI implementation for organizations ready to close the gap between AI hype and AI value. With 30 years of IT experience now focused purely on AI deployment, he works hands-on with companies to turn AI potential into measurable business outcomes.*

*This article is published in [Autocomplete](https://medium.com/autocomplete-real-world-ai), a Medium publication about real-world AI for practitioners and decision-makers. We're always looking for writers. If you're building with AI and have something worth sharing, reach out.*

*My free Substack newsletter, also called Autocomplete, can be found here: https://acdigest.substack.com.*

*Source: conference talk from the PFF post-engineering org lead, recorded May 2026.*
