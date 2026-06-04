# The Bar Is the Work

*The hours of doing just moved to the agent. What is left for you sits at the two ends: setting the bar before it starts, and holding the bar after it finishes. Get good at the ends.*

![An isometric scene: a horizontal coral threshold bar held up at two ends, a knob on the left where a hand sets its height and a magnifying lens on the right where it gets checked, while a teal machine in the middle stacks blocks that have to clear the bar](hero.png)

Last week I handed an agent a task and it came back in eleven minutes with something clean, formatted, and complete-looking. My real work that day was the four minutes before I started it and the ten minutes after it finished. The middle, the writing, the wiring, the grinding, the part that used to be the entire job, I did none of it. And the quality of the result came almost entirely from those two short windows at the ends.

That is the shape of the work now, and it is worth naming, because once you can see it you can get deliberately good at it instead of accidentally bad at it.

The work did not disappear when agents got capable. It split. Two jobs stay on the human side of the line, and everything between them crossed over to the machine. The first is setting the bar: defining what done and correct mean, before the agent starts, in a form it can check itself against. The second is holding the bar: verifying the result against that definition after it finishes, especially when it looks finished. Together they are **the bar**, and the bar is the work now.

**Three things to take away:**

- Setting the bar is specification done up front, in a checkable form. When you did the work yourself, a vague goal cost you one course-correction along the way. When an agent does it, vagueness costs you the whole result, because there is nobody in the loop to fix it mid-run.
- Holding the bar is verification, and it runs against your instincts. A polished output reads as trustworthy at exactly the moment you should check it hardest. Make the check structural, a test, a fresh reviewer, a gate, instead of a feeling.
- The middle, the doing, is where your attention still wants to go. That pull is nostalgia for the old job. Every model release makes the agent better at the middle and no better at knowing your bar. The two ends are the part that stays yours.

## The work bifurcated

For your whole career the bar and the doing were the same activity. You held the standard in your head and enforced it continuously, line by line, as you built. When something drifted below the bar you felt it and corrected, often without naming what the bar even was. The specification lived inside the act of doing the work, which is why most good engineers could never fully explain their standards: they did not need to, they just applied them.

When the agent took over the doing, the bar got ripped out of that middle and exposed at the two ends, where it now has to be explicit. You can no longer enforce a standard by applying it as you go, because you are not the one going. You have to state the standard before the agent starts, and check the output against it after the agent stops. The thing that used to be implicit and continuous is now explicit and bracketed.

![A before-and-after diagram. Before: a single human figure does the work, the bar implicit and continuous inside the doing. After: the human sets the bar on the left, the agent does the work in the middle, the human holds the bar on the right, the bar now explicit at the two ends](diagram-bifurcation.png)

This is why the transition feels harder than it should. The skill did not get more complex. It got pulled out of your hands and turned into something you have to articulate, twice, at moments when you would rather just be building.

## Job one: set the bar

Setting the bar means defining done and correct before the agent runs, in a form that can be checked without you in the room. The strongest version reads like an acceptance test. "Build a subscription flow" is not a bar. "Build a subscription flow where a user can sign up, create a subscription in test mode, cancel it, and the existing tests still pass" is a bar, because every clause is something an agent or a colleague can verify against.

This is the same move as plan mode's "don't implement yet." Telling the agent to write the plan before it writes the code buys a checkpoint between intent and action, and the checkpoint is where the bar gets set while it is still cheap to change. The interview-me-then-write-a-spec pattern is the long-form version: you let the agent ask you the questions you had not thought to answer, and the answers become the bar.

The reason this is hard is that vagueness used to be nearly free. You would start with a fuzzy idea, notice halfway through that it was going sideways, and course-correct. The bar assembled itself as you worked. With an agent there is no halfway correction, because you are not there for the halfway. Whatever you specified at the start is the only standard the entire run has to meet. Vagueness stops being a small tax you pay once and becomes the full bill.

![A Claude Code session: a vague one-line prompt produces a confident but wrong-shaped result, while the same task written as an acceptance test (named files, explicit done conditions, tests must pass) produces a result that can be checked clause by clause](cc-set-the-bar.png)

The test of whether you set the bar well is simple. Could someone else check the result against your specification without asking you a single question? If yes, the bar is real. If they would have to come back and ask what you meant, you did not set a bar, you set a mood.

## Job two: hold the bar

Holding the bar means verifying the finished result against the standard you set, and the hard part is that the finished result will actively discourage you from doing it.

Anthropic measured this directly in its AI Fluency Index. When the output was a polished artifact, a document, a piece of code, an interactive tool, people put more care into directing the work and less into checking it. Fact-checking, gap-spotting, and questioning the result all dropped, at the same time, in the same conversations, exactly when the work looked most done. Polish reads as correctness to the human eye, and that surface signal quietly stands in for the work of confirming whether the thing is right.

So holding the bar is a discipline that fights a reflex. The defense is to stop trusting how finished something looks and attach a check that runs without your judgment in the loop. A test suite. A build that has to pass. A fresh reviewer who sees only the diff and the bar and is told to find the gap, not to confirm the work. A workflow's own adversarial verifier that tries to break the result before it reaches you. The common thread is that the agent which produced the answer is the worst possible judge of the answer, and a separate one with no stake in it is the right judge.

![A two-panel diagram of the two jobs. Left, Set the bar: define done and correct, write it like an acceptance test, before the agent runs. Right, Hold the bar: verify against the bar, distrust polish, use a check that runs without you, after the agent finishes](diagram-two-jobs.png)

The practical rule is to treat polish as the cue. When you are about to wave through a clean, well-formatted result, that is the moment to slow down and ask what it inferred, what it skipped, and what it is confident about that it should not be. The better it looks, the harder you check.

## The middle is a trap

The doing still pulls at you. You hover over the session, you watch the tokens stream, you tweak the prompt mid-run, you feel busy and useful. That feeling is the residue of the job you used to have, and it is the lowest-value place you can put your attention now.

The middle is where the agent has caught up to you and where you add the least. Every minute you spend supervising the doing is a minute you did not spend sharpening the bar or checking the output, which are the two places your judgment is still the scarce resource. The skill to build is noticing when you have drifted into the middle and pulling yourself back to the ends.

## Better models only improve the middle

Prompt engineering was the defining skill of the last two years, and it is depreciating, because each model release needs less of it. Setting and holding the bar does not depreciate the same way, and the reason is structural. A better model does the middle better. It writes cleaner code, makes fewer mistakes, drifts less. None of that improvement touches the question of what you wanted or whether the result fits your specific situation, because that information was never inside the model. It was inside you, and the model can only act on the part of it you managed to express.

So as the models climb, the value does not spread out evenly. It concentrates at the two ends, where a human still has to say what good means and confirm that this is it. That concentration is the whole reason to get deliberate about the bar now rather than later.

## It reshapes who is valuable on a team

A senior engineer's value was never really the typing. It was the judgment about what good looks like and whether the thing in front of them met it. That judgment is exactly the bar, which means the senior's job translates cleanly into the new shape: set and hold the bar across far more work than they could ever have produced by hand. Their leverage goes up.

The exposure lands on whoever's value was the middle, the person whose contribution was producing the output the agent now produces directly. The response is not to defend the middle, it is to move up to the ends, to get good at specifying and verifying. For teams, this changes what to hire for and how to review. Hiring tilts toward people who can define a sharp bar and verify against it. Review shifts from reading every line to checking the result against a stated standard, which is faster and catches different, often worse, problems.

## Building the two muscles

For setting the bar: write the prompt like an acceptance test, with named files, explicit done conditions, and a way to tell right from wrong. For anything large, let the agent interview you first and write the spec, then run it in a clean session. The standing test is whether a colleague could check the work against your spec without asking you anything.

For holding the bar: never accept a result on the strength of how it looks. Attach a check that runs without you, and for anything that ships, add a reviewer in fresh context whose only instruction is to find where the result misses the bar. Train yourself to read polish as a warning rather than an all-clear.

For both: catch yourself in the middle. When you notice that you are watching an agent work and nudging it along, ask whether the bar was set well enough that you could have walked away and trusted the check to catch the misses. If the answer is no, the fix is the bar, not the prompt.

The doing is gone, and it is not coming back. You will hand more and more of the middle to the agent, and it will keep getting better at it than you are. The part that stays yours is the bar, set before and held after. That is not a smaller job than the one you had. It is a harder one, because it runs on judgment instead of effort, and judgment does not get faster with reps the way typing did. Get good at the ends. The bar is the work now.

---

*Marco Kotrotsos, specializing in practical AI implementation for organizations ready to close the gap between AI hype and AI value. With 30 years of IT experience now focused purely on AI deployment, he works hands-on with companies to turn AI potential into measurable business outcomes.*

*This article is published in [Autocomplete](https://medium.com/autocomplete-real-world-ai), a Medium publication about real-world AI for practitioners and decision-makers. We're always looking for writers. If you're building with AI and have something worth sharing, reach out.*

*My free Substack newsletter, also called Autocomplete, can be found here: https://acdigest.substack.com.*

*My books on Amazon: [Claude Code for Everyone Else](https://www.amazon.com/dp/B0H35YY851) and [From Vibe to Production](https://www.amazon.com/dp/B0H34GK9VW).*
