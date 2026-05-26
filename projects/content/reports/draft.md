# Becoming AI-Native Is an Act of Subtraction: What to Let Go, What to Trust

*Most teams trying to become AI-native start by adding. More tools, more agents, more dashboards. The teams that actually get there start by removing. They let go of the processes they built to protect a resource that stopped being scarce, and they trust verification and human judgment to hold the line instead. Here is the full map, drawn from how Anthropic runs the Claude Code and Co-work teams.*

![On the left a tall tower of process blocks tilting and dissolving, on the right a flat stable platform, a hand releasing a block between them](hero.png)

**Three key takeaways:**

- AI-native is not a tooling upgrade. It is a control change. For two decades, engineering organizations built planning, reviews, ownership tracking, and documentation rituals to protect scarce engineering bandwidth. That bandwidth is no longer the bottleneck. The processes that protected it are now the tax.
- The hard part is not adopting AI. The hard part is letting go. You have to give up heavy upfront planning, gatekept reviews, design-doc culture, and the instinct to ask "who touched this," and trust verification, automation, and code-as-source-of-truth to do the job instead. Only after you let go do you get the speed.
- Some things you keep, and keeping them is what makes the letting-go safe: taste, risk and trust boundaries, deep system expertise, and product sense. AI augments where you are weak. It does not replace the judgment that decides what "correct" means.

---

## The bottleneck moved, and your process did not notice

Fiona Fun leads engineering and product for Claude Code and Co-work at Anthropic. Before that she built and led teams at Meta and Microsoft. Her framing for the whole shift is one line she keeps coming back to: what served you before may no longer serve you.

For many years, engineering bandwidth was the expensive thing. Think about how software got built around that fact. Heavy planning, because you wanted high confidence before you spent precious engineering hours. Careful reviews, because rework was costly. Ownership tracking, because the person who knew the code was a scarce resource you had to route around. Every one of those rituals existed to protect engineering time.

This is not the first time the bottleneck has moved. Fun takes you back to Microsoft in the early 2000s, building Visual Studio with no cloud, one server room, a build queue that could merge six PRs at a time. When a test failed you had to work out which of the six broke it. That was a real bottleneck. Cloud and continuous build dissolved it. Nobody mourns the six-PR queue now.

This is the same kind of shift. On the Claude Code team, coding is no longer the slow part. Writing code, writing tests, refactoring: the work that used to define the constraint is now cheap. And when the expensive thing becomes cheap, every process built around protecting it quietly stops working.

![Split screen showing coding as the bottleneck before, verification as the bottleneck now](diagram-2-bottleneck-moved.png)

That phrase, quietly stops working, is the whole problem. A process rarely fails loudly. It just keeps running, consuming time, long after the reason for it is gone. Somebody set it up to solve a real problem. Nobody scheduled the meeting to ask whether the problem still exists.

## What to let go

Becoming AI-native starts with subtraction. Here is what comes off the table.

**Heavy upfront planning.** When coding was expensive, you planned hard to avoid building the wrong thing. When coding is cheap, you can build three versions and look at them. Fun tells a story about a refactoring debate with Boris, a colleague. In the old days they would have booked a room and whiteboarded the approaches. Instead she generated three different versions of the PRs. The debate got better, because they were arguing about real implementations and real impact on callers, not about diagrams of implementations.

**Design docs as the main artifact.** The Claude Code team reduced in-depth design docs. Most discussion now happens in PRs or prototypes. The doc was a proxy for the expensive thing you were about to build. When building is cheap, the proxy costs more than the real thing.

**Gatekept code review as the only safety net.** Review still matters, but it is no longer a single human funnel that everything waits behind. Claude handles the style, the lint, the obvious bugs, and spec drift. That frees the human review for the parts that actually need a human.

**The instinct to ask "who made this change."** This used to be the reflex question: who is the code owner, who touched this last. Fun's advice is to interrogate the question itself. What are you actually trying to answer? Are you hunting a regression? Looking for context? Trying to learn the area? In most of those cases, Claude can answer faster than a person, and you are not interrupting a busy engineer to do it.

**Documentation as the source of truth.** When coding bandwidth was limited, documentation drifting out of date was annoying but survivable. Now that throughput is high, anything outside the update loop goes stale fast. Documentation that lives apart from the code is a liability.

**Whiteboard-first technical debate.** On this team, code wins. Building is cheap, argument is expensive. When you can generate the alternatives in the time it would take to schedule the debate, you generate the alternatives.

None of this is reckless. Each thing you let go of is replaced by something you trust instead.

## What to trust

![Three columns: let go, trust, keep](diagram-1-let-go-trust-keep.png)

Letting go without a replacement is just chaos. The reason a team can drop heavy planning and gatekept review is that it moves its weight onto a different set of supports.

**Trust verification, and shift it left.** This is the new center of gravity. When bandwidth increases and more people are checking in changes, the question that matters is whether the change is correct. Fun's framing: what is better than you running into the bug first? Having automation catch it closer to the source. The investment that used to go into protecting code-writing time now goes into verification and automation. You move the catch as early in the pipeline as you can.

**Trust the code as the source of truth.** When Fun onboarded to Claude Code, the code was the source of truth, and her first technical deep-dive was with Claude, not a human. She asked Claude to teach her the surface area around a bug before she fixed it. Her advice for teams: whatever your source of truth is, get it into the codebase. If it is a spec, turn it into a skill and check it in. Things in the codebase stay in the update loop. Things outside it rot.

**Trust generation over argument.** Instead of debating which approach is right, generate the candidates and look at them. The model makes the alternatives cheap enough that producing them beats arguing about them.

**Trust prototypes, then scale them.** The old worry about prototyping was that you would get attached to throwaway code and ship something that was not built to scale. With Claude, you prototype to learn, then scale the prototype to production far faster than before. The prototype stops being a trap.

**Trust Claude to fill the cross-functional gaps.** This one applies to every role, not just engineering. Designers on the Claude Code team make their own polish and UX fixes with Claude instead of red-lining and handing off to an engineering queue. That closes the iteration loop. On the flip side, Fun, an engineer who tends to write too verbosely, uses Claude as a content-design partner to tighten copy. Claude augments the area where each person is weak.

## What to keep, because keeping it makes the rest safe

Subtraction has a floor. Some things do not move to the model, and protecting them is exactly what lets you trust everything else.

**Keep human judgment on risk and trust boundaries.** Legal reviews. Anything that comes down to risk tolerance. Anywhere a decision crosses a trust boundary. This is trust-but-verify territory, and the verify is human.

**Keep taste and product sense.** Fun's example is perfect because it is small and real. Last December she wanted to give Claude a holiday theme in the CLI and turn it into a snowman. She coded it up and asked a designer to review. The feedback: that looks nothing like a snowman, you turned Claude into Mr. Peanut. She looked again, and it was unmistakably a peanut. That is the kind of call a model will not make for you. Taste is a human input.

**Keep two engineering profiles.** As roles blur and Claude augments, Fun focuses hiring on two types. Creative builders with product sense, and people with deep system expertise. The hard parts still need humans who can hold the whole system in their head and who know what good looks like.

**Build the product-sense muscle deliberately.** When she gave this talk in San Francisco, engineers asked how to build product sense. Her answer: dogfood relentlessly. Use the product you build. Anthropic calls it ant food. She joins a team and immediately starts using the product, because that is how you feel it in your bones and remember the problem you were trying to solve. Then iterate, ship, and talk to actual customers. Her passion project is small businesses; she onboarded restaurant-owner friends onto Co-work and got humbling feedback about an onboarding flow she thought was fine. The alternative is making product decisions off metrics, dashboards, and slide decks, which is how you slowly lose touch with what your product does.

## The org shape that makes this work

A few structural choices made the rest possible.

The org stays flat and agile. And every manager on Claude Code starts as an IC first. Fun is direct about why this works: before a manager takes on the responsibility of supporting people, they roll up their sleeves and learn what it is like to be an engineer on the team. There is a bonus here for managers specifically. This is the moment to get maker hours back, because onboarding is far less daunting than it used to be. When Fun onboarded to Claude, she did her first tech deep-dive with Claude and shipped a bug fix using test-driven development, which she had previously treated like eating broccoli. With Claude writing the test scaffolding, the broccoli tax came off, and TDD became something she actually enjoyed.

The point underneath the org shape: keep the people who make decisions close to the work. A manager who cannot get into the codebase makes decisions off proxies. A manager who can is making them off the real thing.

## How to actually roll it out

![Forcing functions from the top meeting bottoms-up adaptation from the bottom](diagram-3-rollout.png)

The rollout has two halves, and you need both.

**Align top-down on a few forcing functions.** These are the non-negotiables that set the culture. On the Claude Code team they are short. Every teammate uses Claude Code and Co-work, every day. Claudify everything you can: any time you are about to do something, ask whether Claude could do it instead, because every task it takes frees your bandwidth for harder problems. And, the one that does the heavy lifting, explicit permission to kill old processes. Even the team's own principles get revisited when they stop serving their purpose.

**Leave room for bottoms-up adaptation.** How Claude shows up in a team's triage, its standups, its planning rituals, which workflow gets claudified first: all of that is left to each pod. Different teams have different toolchains and different pain. The forcing functions set the floor. The pods choose how to build on it.

The reason the "permission to kill processes" forcing function matters so much: people do not delete processes on their own. They pile up. Fun was once on a team with so many SLAs, one for P0 bugs, one for incident response, on and on, that engineers came to her asking which SLA they were even supposed to satisfy in a 24-hour day. Nobody had ever scheduled the audit. Processes accrete unless removing them is somebody's explicit, blessed job.

There is a second reason to keep revisiting: the models keep improving. Something Claude was not good at three months ago may be solid after a model update. A growth-mindset audit cadence is not just good hygiene, it is how you catch the capabilities that quietly became available.

## How to know it is working

Fun shared the signals the team watches.

**Onboarding ramp-up time goes down.** The span from a new engineer joining to landing their first PR shrinks. So does the cost to existing team members, because new joiners ask Claude the questions they used to ask a busy colleague.

**PR cycle time goes down, but measure it in pieces.** This one has a trap. If your PR cycle time is not dropping, it does not necessarily mean you are failing to adopt AI. It might mean a different part of the queue is jammed, your build or CI cannot keep up with the new throughput. Break PR cycle time into its funnel stages and find which stage is the actual constraint now.

**Claude-assisted commits go up.** On the Claude Code team, nearly every commit in recent months has been Claude-assisted.

And the signal that matters more than any throughput number: whatever your product or problem actually is, find a way to measure that. Throughput going up is meaningless if the product is not getting better. Measure the thing you are actually trying to improve.

## The honest open questions

Fun was transparent that this is unfinished. The questions the Claude Code team is still working out:

Do you still need separate iOS and Android orgs, when Claude lets engineers flex across mobile platforms? How far do you push fully automated reviews before you lose something important? As roles blur, how do you make sure everyone stays productive and confident that their changes are correct?

These are not signs that the approach is shaky. They are what the frontier looks like when you are actually on it.

## Where to start on Monday

The takeaway exercise Fun leaves with audiences is small enough to do this week. Pick your noisiest workflow. The one with the highest tax on the team, or the meeting nobody enjoys, the one where everyone is on their laptop except when they pop up to give status. Count the people in that room and the cost becomes obvious.

For that one workflow, ask the only two questions that matter. Is it still serving its purpose? And if it is expensive, could Claude help with it instead?

Then do the next one. One workflow at a time.

That is the whole method. Becoming AI-native is not a tool you install. It is a sequence of small acts of letting go, each one backed by something you have decided to trust, with a short list of things you refuse to give up. The teams that get there are not the ones who added the most. They are the ones who were willing to remove what stopped working, and trust what replaced it.

---

*Marco Kotrotsos, specializing in practical AI implementation for organizations ready to close the gap between AI hype and AI value. With 30 years of IT experience now focused purely on AI deployment, he works hands-on with companies to turn AI potential into measurable business outcomes.*

*This article is published in [Autocomplete](https://medium.com/autocomplete-real-world-ai), a Medium publication about real-world AI for practitioners and decision-makers.*

*My free Substack newsletter, also called Autocomplete, can be found here: https://acdigest.substack.com.*

*Source talk: Running an AI-Native Engineering Org by Fiona Fun at Code with Claude. https://www.youtube.com/watch?v=IA5LWIGqnyM*
