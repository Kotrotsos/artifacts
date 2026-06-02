# The Polish Trap: What Anthropic's AI Fluency Index Says About How We Actually Use AI

![An isometric scene: a person-shaped abstract form and a glowing teal AI form working over a shared document, a coral magnifying glass resting unused to one side as the document gets more polished](hero.png)

Anthropic just published something more useful than another benchmark. The AI Fluency Index looked at how nearly ten thousand real conversations actually go, and its most important finding is also its most uncomfortable: the better the AI's output looks, the less we check it.

That is not a hypothesis. They measured it. When Claude produced a polished artifact, a document, a piece of code, an interactive tool, people got noticeably better at directing the work and noticeably worse at questioning it. The polish and the scrutiny moved in opposite directions. I have spent two years watching teams deploy AI, and this single result explains more failed rollouts than any model limitation I can name.

Here is what the index found, what the framework underneath it is, and what it means if you are the one responsible for AI work that has to be right.

**Three things to take away:**

- Iteration is the keystone. Conversations where people went back and forth carried twice as many good collaboration behaviors as one-shot exchanges. Staying in the conversation is the highest-leverage habit, and most of the rest follow from it.
- Polish suppresses scrutiny. When the output looked finished, people directed it better but fact-checked, questioned, and gap-spotted it less. The moment the work looks most done is the moment people stop guarding it, which is exactly backwards.
- The cheapest fix is the most ignored. Only about a third of conversations set any explicit terms for how the AI should behave. One sentence telling Claude to push back is nearly free and almost nobody spends it.

## What the index actually measured

The credibility of a finding depends on how it was produced, so it is worth being precise. Anthropic analyzed 9,830 anonymized Claude.ai conversations from a single week in January 2026, filtered to real multi-turn exchanges rather than greetings or one-word replies. Classification was done by Claude Sonnet 4, with Claude Haiku 3.5 handling language detection, and no personally identifiable information was collected.

The behaviors they looked for come from the 4D AI Fluency Framework, developed by professors Rick Dakan and Joseph Feller with Anthropic. It defines four competencies for working with AI well: Delegation, Description, Discernment, and Diligence. The framework lists 24 specific behaviors. Only 11 of those are visible inside a conversation, the other 13 happen in your head or on your machine, so the index measures a real but partial slice. The results held steady across all seven days of the week and across six languages, with most behaviors shifting only one to five percentage points between them. That consistency is what makes the headline findings hard to dismiss as noise.

![The 4D AI Fluency Framework: four labeled quadrants. Delegation, deciding what the AI should handle. Description, communicating the goal clearly. Discernment, evaluating the output critically. Diligence, owning the result and verifying it.](framework-4d.png)

The four competencies split cleanly into two halves. Delegation and Description are about directing the work: deciding what to hand over and explaining it well. Discernment and Diligence are about guarding the work: judging whether the output is actually good and taking responsibility for verifying it. Keep that split in mind, because the central finding is about what happens to the two halves when the output gets polished.

## Iteration is the behavior that pulls the others up

The most common fluency behavior by a wide margin was iteration and refinement, present in 85.7% of conversations. People treat a first answer as a starting point and push on it. That is the good news, and it matches the report's framing that the most common form of fluency is augmentative: people use Claude as a thought partner rather than handing off the work and walking away.

What makes iteration matter is not its frequency but its pull on everything else. Conversations with iteration carried 2.67 fluency behaviors on average. Conversations without it carried 1.33. That is roughly double. Iteration is not one habit among many, it is the habit that drags the others into the room. When people stayed in the conversation, they were 5.6 times more likely to question the model's reasoning and 4 times more likely to notice missing context.

![A comparison chart: conversations without iteration average 1.33 fluency behaviors, conversations with iteration average 2.67, with two callouts showing iteration makes users 5.6x more likely to question reasoning and 4x more likely to spot missing context](chart-iteration.png)

The practical reading is simple. If you only change one thing about how your team uses AI, make it this: do not accept the first answer. The back-and-forth is where the judgment lives. A one-shot prompt that returns a finished-looking answer is the lowest-fluency interaction there is, and it is also the most tempting, because it feels efficient. It is not. It is just fast.

## The polish trap, in the numbers

Now the finding that should change how you work. About 12.3% of the conversations produced an artifact: code, a document, an interactive tool, something concrete and finished-looking. Anthropic compared the behavior in those conversations against the rest, and the split is stark.

When an artifact was in play, the directing behaviors jumped. People clarified their goals 14.7 percentage points more often. They specified the format 14.5 points more. They provided examples 13.4 points more, and they iterated 9.7 points more. In other words, when people knew they were building something real, they put visibly more care into describing what they wanted.

And at the same time, the guarding behaviors fell. People were 5.2 points less likely to identify missing context, 3.7 points less likely to check facts, and 3.1 points less likely to question the model's reasoning. The better the output looked, the less people interrogated it.

![A diverging bar chart titled The Polish Trap. Top half in teal shows behaviors that rise with polished artifacts: clarify goals +14.7, specify format +14.5, provide examples +13.4, iterate +9.7. Bottom half in coral shows behaviors that fall: identify missing context -5.2, check facts -3.7, question reasoning -3.1.](chart-polish-trap.png)

Anthropic states it plainly: "Polished outputs coincide with lower rates of critical evaluation, even though users go to greater lengths to direct Claude's work at the outset." Read that twice. The effort goes up and the verification goes down, at the same time, in the same conversations. Polish is doing something to our judgment. A clean, well-formatted, confident-looking artifact reads as trustworthy, and that surface signal quietly stands in for the work of actually checking whether it is right.

This is the whole ballgame for anyone shipping AI work to production. The output that most needs scrutiny, the finished artifact you are about to commit or send or deploy, is precisely the output your instincts will wave through. The failure is not that the model is wrong sometimes. Every tool is wrong sometimes. The failure is that the form of the output is actively suppressing the human check that would have caught it.

## The one-sentence fix almost nobody uses

The index found that only about 30% of conversations set any explicit terms for the collaboration. Seventy percent of the time, people just start asking, with no instruction about how they want the AI to behave. No "push back if my assumptions are wrong," no "tell me what you are unsure about," no "flag anything you are inferring rather than reading."

That is a remarkably cheap lever sitting unused. Telling Claude up front to challenge you, to surface its uncertainty, to separate what it knows from what it is guessing, costs one sentence and measurably changes the interaction. It is the closest thing to a free upgrade in the whole report, and it is left on the table in two out of three conversations.

## The polish trap scales with the whole team

Most coverage of a report like this will land on individual advice: be more careful, check your work. Useful, but it misses the organizational shape of the problem. The polish trap is not a personal failing you can will away. It is a predictable response to a surface signal, and it scales.

When a team adopts AI, the artifacts get more polished over time as people get better at the directing behaviors, clarifying goals, specifying formats, giving examples. The index shows people genuinely improve at those. But the same data shows the guarding behaviors do not improve alongside them, they decay under polish. So the natural trajectory of an AI-adopting team is toward output that looks more and more professional while being checked less and less. That gap is invisible right up until something polished and wrong ships.

This maps exactly onto what separates teams that scale AI from teams that stall. The ones that succeed build the verification in as a process, not a personal virtue: a test the artifact has to pass, a second person who reviews in fresh context, an explicit "what did you infer here" step before anything ships. They do not rely on individual diligence holding firm against the pull of polish, because the data says it will not. They make the check structural so it fires whether or not anyone feels suspicious that day.

## The honest limitations

A good reading of this report includes its caveats, and Anthropic lists them. The sample is early adopters during one week, so it does not represent the general population. Only 11 of the 24 framework behaviors are observable in conversation, so a lot of real diligence, the fact-checking you do in your head, the code you test on your own machine, the colleague you ask, simply is not captured. The classification is binary, present or absent, which misses degree. And the findings are correlational, not causal: iteration is associated with more fluency behaviors, but the report cannot prove iteration causes them.

None of that undoes the core result. The artifact comparison is a within-dataset contrast, the same population behaving differently when polish enters the picture, and it held across languages and days. You can argue about the absolute numbers. The direction is solid, and the direction is the part that should change your behavior.

## What to actually do with this

Three things, in order of leverage.

Stay in the conversation. Treat the first answer as a draft no matter how good it looks. The single behavior most associated with everything else going right is refusing to accept one-shot output. This is mostly a matter of resisting the efficiency illusion, the finished-looking answer that invites you to stop thinking.

Distrust polish specifically. Build a habit, or better a process, that triggers harder scrutiny exactly when the output looks most finished. When you are about to accept a clean artifact, that is the cue to slow down, not speed up. Ask what it inferred, what it skipped, what it is confident about that it should not be. The polish is the warning sign, not the all-clear.

Set the terms up front. Spend the one sentence. Tell the AI to push back, to flag uncertainty, to mark what it is inferring. Two-thirds of people never do this, which means doing it puts you in a small minority for the price of a single instruction.

The deeper lesson of the index is that fluency is not about getting better answers out of the model. It is about not letting the model's growing polish erode your judgment. The skill that matters most is the one that gets harder to practice exactly as the output gets better, and the only reliable defense is to stop trusting how finished something looks and start checking whether it is true.

---

*Marco Kotrotsos, specializing in practical AI implementation for organizations ready to close the gap between AI hype and AI value. With 30 years of IT experience now focused purely on AI deployment, he works hands-on with companies to turn AI potential into measurable business outcomes.*

*This article is published in [Autocomplete](https://medium.com/autocomplete-real-world-ai), a Medium publication about real-world AI for practitioners and decision-makers. We're always looking for writers. If you're building with AI and have something worth sharing, reach out.*

*My free Substack newsletter, also called Autocomplete, can be found here: https://acdigest.substack.com.*

*My books on Amazon: [Claude Code for Everyone Else](https://www.amazon.com/dp/B0H35YY851) and [From Vibe to Production](https://www.amazon.com/dp/B0H34GK9VW).*
