# Becoming AI-Native Is Mostly About What You Stop Doing

*A talk from Anthropic's Claude Code lead reframed the whole thing for me. Becoming AI-native is not adoption. It is subtraction plus trust. Here is the map, and the one exercise to run on your team this week.*

![On the left a tall tower of process blocks tilting and dissolving, on the right a flat stable platform, a hand releasing a block between them](hero.png)

I have watched a lot of teams try to become "AI-native" by adding things. A new agent here, a dashboard there, a Slack bot that summarizes standups. Six months later they are slightly faster and a lot more cluttered, and nobody can say what actually changed.

Then I watched Fiona Fun's talk on running the Claude Code and Co-work teams at Anthropic, and it reframed the whole problem. She has built and led teams at Meta and Microsoft, and her point is the opposite of adding. Becoming AI-native is mostly about what you stop doing.

Hit reply and tell me which process on your team quietly stopped working. I am collecting these.

**Three things to hold onto:**

- AI-native is a control change, not a tooling upgrade. For twenty years we built planning, reviews, and ownership rituals to protect scarce engineering bandwidth. That bandwidth is no longer scarce. The rituals are now the tax.
- The hard part is letting go. You give up heavy planning, gatekept reviews, design-doc culture, and the reflex to ask "who touched this," and you trust verification, automation, and code-as-source-of-truth instead. The speed comes after you let go, not before.
- A short list of things you keep, and keeping them is what makes the letting-go safe: taste, risk and trust boundaries, deep system expertise, product sense.

## The bottleneck moved and the process did not notice

Fun keeps coming back to one line: what served you before may no longer serve you.

For years, engineering bandwidth was the expensive thing, so everything got built to protect it. Heavy planning to avoid wasting precious hours. Careful reviews because rework was costly. Ownership tracking because the person who knew the code was scarce.

She points out this is not the first time the bottleneck moved. Microsoft, early 2000s, building Visual Studio. No cloud. One server room. A build queue that merged six PRs at a time, and when a test failed you had to find which of the six broke it. Cloud and continuous build dissolved that. Nobody mourns the six-PR queue.

Same shift now. On the Claude Code team, coding is no longer the slow part. Writing code, writing tests, refactoring: all cheap. And when the expensive thing becomes cheap, the processes built to protect it quietly stop working.

That phrase is the whole problem. A process rarely fails loudly. It keeps running, eating time, long after its reason is gone. Someone set it up to solve a real problem. Nobody scheduled the meeting to check whether the problem still exists.

![Split screen showing coding as the bottleneck before, verification as the bottleneck now](diagram-2-bottleneck-moved.png)

## What to let go

**Heavy upfront planning.** When coding was expensive you planned hard. When it is cheap you build three versions and look at them. Fun describes a refactoring debate with a colleague where, in the old days, they would have booked a room and whiteboarded approaches. Instead she generated three versions of the PRs. The debate got better because they were comparing real implementations and real impact on callers.

**Design docs as the main artifact.** The team reduced in-depth design docs. Discussion happens in PRs and prototypes now. The doc was a proxy for the expensive thing you were about to build. When building is cheap, the proxy costs more than the thing.

**Gatekept review as the only safety net.** Review still matters. It is no longer a single human funnel everything waits behind. Claude handles style, lint, obvious bugs, and spec drift, which frees human review for the parts that need a human.

**The reflex to ask "who made this change."** Interrogate the question. Are you hunting a regression, looking for context, trying to learn the area? In most cases Claude answers faster, and you are not interrupting an engineer.

**Documentation as the source of truth.** High throughput means anything outside the update loop goes stale fast. Documentation that lives apart from the code is a liability. This is a real reversal. For years the advice was to document more. Now the advice is to document inside the thing that stays current, and treat anything else as a snapshot that will lie to you within weeks.

**Whiteboard-first debate.** Code wins. Building is cheap, argument is expensive. When you can generate the alternatives in the time it takes to schedule the debate, you generate them. Fun describes nearly walking a colleague to a meeting room to whiteboard a refactoring approach, then catching herself and generating three real versions of the PRs instead. The debate that followed was better, because it was about real code and real impact on callers, not sketches of code.

A pattern runs through all of these. Each thing you let go was a proxy. Planning was a proxy for confidence. Design docs were a proxy for the build. Ownership tracking was a proxy for knowledge. When the underlying work gets cheap, the proxy costs more than the real thing, and you can finally drop it.

## What to trust instead

![Three columns: let go, trust, keep](diagram-1-let-go-trust-keep.png)

Letting go without a replacement is chaos. The weight moves onto different supports.

**Verification, shifted left.** This is the new center of gravity. When more people check in more changes, correctness is the question that matters. Fun's framing: better than you catching the bug first is automation catching it closer to the source. The investment that protected code-writing time now goes into verification.

**Code as the source of truth.** When Fun onboarded, her first deep-dive was with Claude, not a person. She asked Claude to teach her the area around a bug before fixing it. Her advice: get your source of truth into the codebase. If it is a spec, make it a skill and check it in. Things in the codebase stay current. Things outside it rot.

**Generation over argument.** Generate the candidates and look at them instead of debating which is right.

**Prototypes you then scale.** The old fear was getting attached to throwaway code. Now you prototype to learn, then scale to production fast. The prototype stops being a trap.

**Claude for the cross-functional gaps.** Designers make their own polish fixes instead of red-lining and handing to an engineering queue. Fun, who writes too verbosely, uses Claude to tighten copy. It augments where each person is weak.

## What to keep

Some things do not move to the model, and protecting them is what lets you trust the rest.

Human judgment on risk and trust boundaries. Legal reviews, anything about risk tolerance, anything crossing a trust boundary. This is trust-but-verify territory, and the verify is human. The more you automate everywhere else, the more important it is to be precise about where a human still has to sign off. Drawing that line clearly is what lets you move fast on everything that sits inside it.

Taste and product sense. Fun's example is small and exactly right. Last December she wanted to give Claude a holiday theme in the CLI and turn it into a snowman. She coded it up, asked a designer to review, and got told it looked nothing like a snowman, it looked like Mr. Peanut. She looked again and it was unmistakably a peanut. A model will not make that call for you. Taste is a human input, and no amount of throughput substitutes for it.

Two engineering profiles to hire for: creative builders with product sense, and people with deep system expertise. As roles blur and Claude augments, these are the two she doubles down on. The hard parts still need humans who hold the whole system in their head and know what good looks like.

And build product sense deliberately, because people asked her how. Her answer: dogfood relentlessly. Anthropic calls it ant food. She joins a team and immediately starts using the product, because that is how you feel it in your bones and remember the problem you were trying to solve. For managers this is newly possible, because before Claude they often had no time to be in the codebase or the product at all. Then iterate, ship, and go talk to actual customers. Fun's passion project is small businesses; she onboarded restaurant-owner friends onto Co-work and got humbling feedback on an onboarding flow she thought was fine. The alternative is deciding off metrics, dashboards, and slide decks, which is how you slowly lose touch with what your product actually does.

## Org shape and rollout

Flat and agile, and every manager starts as an IC first. Before taking on supporting people, they learn what it is to be an engineer on the team. For managers this is the moment to get maker hours back, because onboarding is far less daunting now. Fun shipped her first bug fix with test-driven development, which she used to treat like eating broccoli; with Claude writing the test scaffolding, the broccoli tax came off.

![Forcing functions from the top meeting bottoms-up adaptation from the bottom](diagram-3-rollout.png)

The rollout has two halves. Align top-down on a few forcing functions: everyone uses Claude Code and Co-work daily, claudify everything you can, and give explicit permission to kill old processes. Then leave room for bottoms-up adaptation: each pod decides how Claude shows up in triage, standups, and which workflow gets claudified first.

The permission-to-kill function matters because people do not delete processes on their own. They pile up. Fun was once on a team with so many SLAs, one for P0 bugs, one for incident response, on and on, that engineers came to her asking which SLA they were even supposed to satisfy in a 24-hour day. Nobody had ever scheduled the audit. Removing a process has to be somebody's explicit, blessed job, or it never happens. And keep revisiting, because the models keep improving. What Claude could not do three months ago it may handle now, which means the audit is not one-time hygiene, it is how you catch the capabilities that quietly became available.

## How to know it is working

Onboarding ramp-up time drops. The span from a new engineer joining to landing their first PR shrinks, and so does the cost to existing teammates, because new joiners ask Claude the questions they used to ask a busy colleague. Fun points out a nice side effect: managers stop feeling guilty about getting back into the codebase, because they are no longer taking an engineer's time to learn an area, they are asking Claude.

PR cycle time drops, but measure it in pieces. This one has a trap. If your PR cycle time is not dropping, it does not automatically mean you are failing to adopt AI. The jam might be elsewhere in the queue, your build or CI failing to keep up with the new throughput. Break the cycle time into its funnel stages and find the stage that is actually the constraint now.

Claude-assisted commits climb. On the Claude Code team, nearly every commit in recent months has been Claude-assisted.

The signal that beats all the throughput numbers: measure whatever your product is actually trying to improve. Throughput going up is meaningless if the product is not getting better. Find the metric that maps to the thing you are building, and watch that.

She was also honest that the work is unfinished. Open questions the team is still sitting with: do you still need separate iOS and Android orgs when engineers can flex across platforms? How far do you push fully automated reviews before you lose something that mattered? These are not cracks in the approach. They are what the frontier looks like when you are standing on it.

## The exercise to run this week

Pick your noisiest workflow. The highest-tax one, or the meeting nobody enjoys where everyone is on their laptop except to give status. For that one workflow, ask two questions. Is it still serving its purpose? And if it is expensive, could Claude do it instead?

Then do the next one. One workflow at a time.

That is the method. AI-native is not a tool you install. It is a sequence of small acts of letting go, each backed by something you trust, with a short list of things you refuse to give up.

What is your noisiest workflow? Reply and tell me. I want to know which ones are hardest to give up.

Until next week,

Marco

---

*Marco Kotrotsos writes Autocomplete, a free newsletter on practical AI for people actually shipping it. Source talk: Running an AI-Native Engineering Org by Fiona Fun at Code with Claude. https://www.youtube.com/watch?v=IA5LWIGqnyM*
