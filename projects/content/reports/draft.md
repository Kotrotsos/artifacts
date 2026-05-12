# Uber Burned Its Annual AI Budget in Four Months

*Five thousand engineers got Claude Code in December. The pricing model, not the engineers, did this.*

**Key takeaways:**

- Uber gave 5,000 engineers Claude Code in December 2025. By April 15, the CTO told the company the entire 2026 AI budget was spent. Per-engineer cost ran $500 to $2,000 a month.
- This is not an engineering overreach story. It is a finance story. Consumption pricing and annual budgeting are structurally incompatible, and most enterprises are building their FY26 AI line item on assumptions that already broke at Uber.
- The fix is not to slow adoption. The fix is to tier engineers, move to quarterly elastic budgets, and measure cost per merged PR instead of cost per token. Three moves you can make in the next thirty days.

---

In December 2025, Uber rolled out Claude Code to 5,000 engineers. By February the usage had nearly doubled. By March, 84% of Uber's developers were classified as agentic coding users, up from 32% the prior December. On April 15, Chief Technology Officer Praveen Neppalli Naga told the company they had burned through the entire 2026 AI budget. Four months in.

The reaction in the press was predictable. AI is expensive. Engineers got carried away. Maybe the tools are not ready for production scale. Maybe the spend is irresponsible.

It is none of those things. Look at the same data with finance eyes instead of engineering eyes and a different story emerges. Uber did not blow its budget because engineers were wasteful. Uber blew its budget because the pricing model of the tool and the budgeting model of the company are structurally incompatible. And every enterprise reading the headlines about Uber has the same problem, sitting in their own FY26 plan, waiting to surface.

This is the article I would want to read if I were running a finance review next week.

## What actually happened at Uber

Start with the numbers, because the numbers tell the story better than the framing.

Uber's R&D budget for 2026 includes a $3.4 billion allocation, of which the AI line item is the fastest-growing slice. The CTO did not say AI costs were 6x what was forecast. He said the entire annual envelope, the full year of planned spend, was gone in four months. AI-related costs at Uber are up roughly 6x since 2024.

The per-engineer numbers are the interesting part. Monthly Claude Code spend per engineer ranged from $500 to $2,000. That four-fold variance inside a single workforce is the first signal something different is happening. With a SaaS seat, the per-engineer cost is bounded by the per-engineer license price. With a consumption-priced tool, the per-engineer cost is bounded only by how many agent loops that engineer chooses to run.

The adoption curve was the second signal. From December to March, the percentage of Uber engineers classified as agentic coders went from 32% to 84%. By April, 95% of all Uber engineers were using AI tools at least monthly, and 70% of committed code had some AI involvement. Approximately 11% of live backend code changes are now written by AI agents end-to-end. None of this is bad. All of it is the kind of adoption rate companies dream about when they sign a deal.

The third signal, the one I want to spend more time on, is what Uber did to drive that adoption. Engineers were ranked on internal dashboards by AI tool usage. Usage was a visible performance signal. Leadership was rewarding the people who used Claude Code the most.

Read those three signals together. The tool is consumption-priced. The workforce is incentivized to consume. The budget is annual and fixed. There is no version of those three statements that ends with the budget surviving the year.

## The pricing model mismatch

Software has had two budgeting eras. Both worked because the pricing model matched the budgeting model.

The license era had perpetual software. You paid once, you owned a copy, the budget was capital expense, and depreciation handled the rest. Predictable. The SaaS era introduced per-seat subscriptions. Per-seat is bounded. Each licensed user can consume at most one unit of the service per period. You count seats, multiply by price, add a buffer, and you have a number. Annual budgeting works fine.

Consumption pricing breaks both of those assumptions.

With Claude Code, the unit of consumption is not the engineer. It is the agent loop. One engineer running one well-crafted prompt against a small codebase might cost $100 a month. The same engineer running an agentic workflow that spawns sub-agents, calls tools, reads thousands of files into context, executes multiple iterations, and merges back into a parent task might cost $2,000 a month. The capacity for spend is not bounded by headcount. It is bounded by the engineer's willingness to launch agent loops, multiplied by the average complexity of those loops.

Agentic AI uses 5 to 30 times more tokens per task than a standard chatbot interaction, depending on the workload. That is the multiplier that makes consumption-priced agents structurally different from any SaaS product your finance team has ever budgeted for.

Annual budgets are built on the bounded assumption. You forecast a number based on expected seats, expected usage per seat, and a contingency. That math works when the per-seat ceiling is a hard ceiling. It does not work when the per-seat ceiling is the engineer's imagination plus the latency of a feedback loop they enjoy.

That is what Uber discovered. The engineers were not wrong. The budget was wrong.

## The internal leaderboard problem

This is the piece I want managers and execs to sit with.

Uber rolled out Claude Code with an explicit adoption push. Internal dashboards ranked engineers by AI tool usage. That is a textbook adoption strategy. Make the behavior visible, reward it, watch usage spread. It worked. 32% to 84% in three months is not a fluke. It is the result of well-designed internal incentives.

The problem is that the same lever that drives adoption drives cost. There is no separation between "engineer is being productive with AI" and "engineer is generating tokens." For a consumption-priced tool, productivity signal and cost signal are the same signal.

If your leaderboard rewards usage, your leaderboard is also rewarding spend. If your spend is uncapped per engineer, your leaderboard is an unhedged buy order on Anthropic's revenue, written in your engineering culture.

This is not a reason to drop the leaderboard. Cultural pressure works. The point is to acknowledge that the cultural lever and the financial lever are now the same lever, and most companies have not thought through what that means.

## Why every enterprise has this problem

Uber is not unusual. Uber is just the public version of what is happening everywhere.

The FinOps Foundation's 2026 State of FinOps report has the supporting numbers. 78% of IT leaders have experienced unexpected charges on a consumption-based SaaS bill in the past year. 98% of FinOps practitioners are now tasked with managing AI spend, up from 31% in 2024. The average enterprise AI budget went from $1.2 million in 2024 to $7 million in 2026, a 5.8x increase in two years.

The pattern is industry-wide. Companies are rolling out consumption-priced AI tools to broad engineering populations. The tools work. Adoption spreads. Token consumption scales with adoption. The annual budget, set in October of the previous year on assumptions that already do not match the tool, runs out somewhere in Q2 or Q3.

The companies talking publicly about this are doing the rest of us a favor. The ones not talking are either lucky, in a smaller pilot, or about to have the same conversation with their CFO in a less convenient context.

## A budgeting model that survives this

I am going to give you four specific changes. None of them require slowing adoption. All of them are about restructuring the cost side to match the new revenue side of the tool's pricing model.

**Move from annual fixed to quarterly elastic.** Annual envelopes were designed for predictable consumption. If your consumption shape is non-predictable, your envelope cadence has to match. Quarterly budgets with explicit re-forecast at each quarter close, ratified by finance, give you four chances to catch a runaway curve. Not one chance, with eight months of damage already done.

**Set a per-engineer monthly soft cap with explicit overage approval.** Not a hard cut-off. A signal. If an engineer crosses the cap, their manager gets a ping, the engineer gets a brief justification ask, and the additional spend is approved or rerouted. This is how cloud spend is managed already. It is not new infrastructure. It is just applying the same governance to AI tools that you already apply to AWS.

**Measure cost per merged PR or cost per deployed feature, not cost per token.** Cost per token is the wrong unit. It compares well to other tokens. It compares badly to everything you actually care about. Cost per merged PR is the unit that lets you have an honest conversation about ROI. If the cost per merged PR is dropping while the cost per engineer is rising, the tool is winning. If the cost per merged PR is flat or rising, you have an adoption problem dressed up as a productivity story.

**Tier engineers by usage profile.** Light user, heavy user, agent runner. The agent runner is a different cost center than the light user. Mixing them in the same budget bucket lets a small number of agent runners distort the per-capita number, which makes the budget look better than it is on average and worse than it should be at the heavy end. Tiered budgets give you accuracy and let you say honest things to your CFO about where the money is going.

These four moves are not novel. They are the standard FinOps playbook for any consumption-priced resource, applied to AI tooling. The novelty is that most companies have not yet realized AI tooling sits in the same category as cloud infrastructure for budgeting purposes, not the same category as Microsoft Office for budgeting purposes.

## What to do in the next thirty days

If you are reading this and you sit in front of a budget that includes an AI line item, three concrete moves.

First, find out what your current per-engineer consumption looks like, by engineer, by month, broken down by tool. Most companies do not have this data. Get it. If the answer is "we cannot get it without a quarter of engineering work," that is your first finding. Build the visibility before you build the controls.

Second, talk to whichever engineering leader is closest to the adoption push and find out what the cultural incentives look like. Are engineers being ranked by usage? Are managers being told to drive adoption to a specific percentage? Are there internal leaderboards? If yes, surface the cost implications now, while you can still adjust the metric, instead of waiting for the bill.

Third, run the numbers as if your AI line item is going to grow 5 to 10x over the next twelve months. That is the magnitude Uber and the FinOps data suggest is plausible. If a 5x growth in your AI budget is survivable, you have time. If a 5x growth would crater the rest of your tooling budget, you have a planning problem to solve before the curve catches you.

None of these require slowing adoption. None require punishing engineers. All of them give you better data and better controls than the median Fortune 500 has right now.

## Closing

The Uber number is not a warning about Claude Code specifically. It is a warning about an entire generation of consumption-priced AI tooling meeting a generation of annual-budgeting practices that were designed for a slower, more bounded category of software.

Uber is the canary. The companies that read the canary correctly will spend the rest of 2026 quietly tuning their cost models while their competitors are stuck in emergency budget reviews. The companies that read it as an Uber problem will have their own version of the conversation by Q3.

You can adopt aggressively and budget sanely. The two are not in conflict. They just require you to update the budgeting model at the same speed you are updating the tooling.

---

*Marco Kotrotsos, specializing in practical AI implementation for organizations ready to close the gap between AI hype and AI value. With 30 years of IT experience now focused purely on AI deployment, he works hands-on with companies to turn AI potential into measurable business outcomes.*

*This article is published in [Autocomplete](https://medium.com/autocomplete-real-world-ai), a Medium publication about real-world AI for practitioners and decision-makers. We're always looking for writers. If you're building with AI and have something worth sharing, reach out.*

*My free Substack newsletter, also called Autocomplete, can be found here: https://acdigest.substack.com.*
