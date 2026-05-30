# Five Ambitious Things to Build With Ultracode, From an Email Client to an ERP

![An isometric workshop bench seen from above, five separate coral-and-teal constructions in progress across the surface, each fed by its own fan of parallel tracks, one finished object glinting at the front](hero.png)

A few weekends ago I let Claude Code build a working SaaS while I made coffee. It picked the idea itself (a no-code platform for hosting MCP servers), wired up a landing page, auth, Stripe in test mode, and a deploy config, and the thing ran. That experiment changed how I think about what is worth attempting.

For two years the constraint on a software idea was build cost. You had an idea, and then you counted the months. Most ideas died in that count. Dynamic workflows, and the `ultracode` setting that turns them on for a whole session, move the constraint. When Claude can write its own orchestration script and run a fleet of subagents in parallel, each one checked by another agent before anything reaches you, the build is no longer the thing that kills the idea. Ambition is the scarce resource now.

So this is a list of five things I would point a session of ultracode at, ordered from believable to genuinely wild. For each one: why it is ambitious, a short version of how you would actually start it, what the money looks like, and an honest read on how crowded the field already is. None of these is a guaranteed business. All of them are now buildable by one person with a plan and a budget for tokens.

**Three things to hold onto:**

- The build cost of ambitious software just dropped to near zero for anything that is partitionable and verifiable. That reshapes which ideas are worth starting, not just how fast you ship them.
- The winning move on every one of these is the same: chain several workflows, each with a hard bar, instead of asking for the whole thing in one shot. Map, then build in parallel, then verify, then harden.
- The agent builds the software. It does not build your distribution, your trust, or your taste. Pick ideas where you can own those three, because that is the part no workflow ships for you.

![A grid of the five builds: AI-native email client, vertical ERP, continuous compliance, AI-native observability, and a live browser game with agent-built worlds, each with its business model and how crowded the field is](idea-grid.png)

## 1. An AI-native email client that actually triages

**The ambition.** Email is the oldest productivity surface and still the worst-run one. The clients people pay for are fast (Superhuman) or tied to an ecosystem (Notion Mail), but almost none of them genuinely run your inbox for you: read everything overnight, draft the twelve replies that are obvious, surface the three that need a human, and archive the rest with a reason you can audit. That is an agent product wearing an email client's clothes, and it is now a one-person build.

**Getting started.** This decomposes cleanly, which is exactly what makes it a workflow target. One workflow builds the IMAP/Gmail sync and the data model. A second builds the triage engine as a set of classifiers, each one a verifiable unit (does this rule fire correctly on a labeled test set). A third builds the compose-and-draft flow. A fourth builds the keyboard-driven UI. You name the bar for each (the sync passes its tests, the classifier hits a precision threshold on your own labeled inbox) and let the parallel agents grind. Start with your own email as the test harness. You are the acceptance test.

**The money.** The market is segmented, not saturated, and it is priced. Superhuman sits at $33 a month, Shortwave and MailOver around $7 to $7.50, and the open-source Inbox Zero undercuts everyone near $2. The clear gaps are reply quality, multilingual handling, and price. A genuinely agentic client at $10 to $15 a month, aimed at people drowning in email who find Superhuman expensive and everything else dumb, has room. Prosumer subscription, land through one vertical (founders, recruiters, customer-success leads) where inbox volume is brutal.

## 2. A vertical ERP for an industry that still runs on paper

**The ambition.** This is the boring one that prints money. Vertical software is on track from roughly $169 billion in 2025 to around $549 billion by 2035, and the reason is that the giants never went deep enough. In field service alone, the incumbents like ServiceTitan and ServiceTrade have barely penetrated one percent of the market, and a lot of what they did ship just digitized pen-and-paper without automating anything. Pick one underserved trade (commercial fire-and-safety, specialty fabrication, industrial laundry, agricultural co-ops) and build the ERP that actually fits how that trade works, with AI doing the scheduling, quoting, and compliance paperwork that the generic tools punt on.

**Getting started.** An ERP is the canonical "too big for one conversation" task, which is the whole point of workflows. You stage it. One workflow models the domain (work orders, inventory, scheduling, invoicing, the specific compliance forms that trade lives and dies by). One builds each module in parallel against that schema. One builds the integrations (accounting, payments, the two or three industry tools they already use). One runs a security and data-integrity pass over the whole thing. The hard bar is the domain model and the compliance forms, so get a real operator in that trade to define "correct" before you start. Their definition of done is your prompt.

**The money.** Vertical ERP is high-ACV B2B. You are not selling $15 a month, you are selling seats and modules at hundreds to thousands per customer per year, with real switching costs once their operations live inside it. The catch is that this market is sold, not marketed, so the agent building the software is the easy half. The hard half is one operator who trusts you enough to be customer zero. If you have that relationship in a specific trade, this is the most defensible thing on the list.

## 3. A continuous compliance platform that undercuts the incumbents

**The ambition.** Getting SOC 2, ISO 27001, or HIPAA is a tax every B2B software company pays, and the tools that automate it are expensive. Vanta starts around $10,000 a year and runs past $80,000. Drata runs $7,500 to over $100,000 and stacks $10,000 to $25,000 of onboarding on top. Mid-market programs cluster at $40,000 to $75,000. There is a real opening underneath that: a continuous-evidence platform for companies under fifty people who need one or two frameworks, done well, without the enterprise price tag or the onboarding fee. Connect to their cloud, collect evidence automatically, flag what is failing, and keep them audit-ready year round.

**Getting started.** Compliance automation is a giant pile of integration-and-check work, which fans out beautifully. One workflow builds the connectors (cloud providers, identity, code hosting, HR). One builds the control library as a set of checks, each independently verifiable against a fixture. One builds the evidence-collection scheduler. One builds the auditor-facing reporting. The bar is unambiguous here, which is rare and valuable: a control either has its evidence or it does not. That makes the verifier agents genuinely useful, because correctness is a fact, not a taste.

**The money.** B2B SaaS with strong retention, because nobody rips out their compliance tool mid-audit. Price it as the honest alternative: a few hundred a month for a single framework, no five-figure onboarding, aimed at the seed-to-Series-A companies that Vanta and Drata price roughly but do not love. Cheaper challengers already exist in this slot, so your wedge is a specific framework done better and an AI that actually explains what is failing and how to fix it, not just a red dashboard.

## 4. A self-hostable, AI-native observability suite

**The ambition.** Observability is where companies quietly bleed money. Mid-sized teams spend $50,000 to $150,000 a year on Datadog, and enterprise bills clear a million. The open-source alternatives (SigNoz, OpenObserve, the Grafana stack) cut the software cost to near zero but convert it into headcount, because someone has to run them and someone has to actually read the dashboards at 3am. The ambitious build is the open-core suite that closes both gaps: it self-hosts on object storage so the bill stays sane, and it ships an agent that does the on-call triage, correlates the spike to the deploy, and writes the incident summary before a human opens the laptop.

**Getting started.** This is a systems build, so the workflow plays to its strengths. One workflow builds the ingestion pipeline on the OpenTelemetry standard. One builds the storage and query layer (ClickHouse or Parquet-on-S3 are the proven patterns). One builds the dashboards and alerting. One builds the triage agent that reads the telemetry and proposes a root cause. Verify each stage against replayed production traces. The grunt work of wiring a hundred integrations is exactly the bounded, parallelizable labor a fleet of agents eats for breakfast.

**The money.** Open-core is the proven model here: the self-hostable core is free and earns you adoption and trust, the managed cloud and the support contracts are the revenue. You are selling against a Datadog bill that everyone already hates, so the pitch writes itself. The moat is not the dashboards, which are commodity. It is the triage agent being genuinely good, which is the part the existing open-source tools do not have and the part workflows let one team build.

## 5. A live multiplayer browser game with worlds the agents build

**The ambition.** Here is the wild one. Indie games on Steam pulled over $4.5 billion last year, a quarter of the platform's revenue, and the browser is having a genuine renaissance because the technical barrier finally dropped low enough for tiny teams. The audacious version is a persistent multiplayer browser game whose content (levels, quests, items, the economy) is generated and balanced by agents rather than hand-authored by a studio of forty. One person directs the vision. A fleet of agents builds and tunes the world.

**Getting started.** Games are less obviously verifiable than the other four, so you build the bar yourself. One workflow builds the netcode and the core loop. One generates content (maps, encounters, items) in parallel, with a second set of agents playing the content to check it is winnable and not broken, which is your verification loop. One builds the live-ops and economy. The trick is making "fun" measurable enough that the verifier agents have something to push against (completion rates, balance metrics, exploit detection), because an agent cannot grade taste but it can grade whether a level is beatable.

**The money.** Be honest that this is the riskiest slot. Game revenue is real but fickle. The models that work in 2026 are a premium base with optional cosmetics, a battle pass, and a power-user subscription, not gating core play behind a wall. The realistic monetization may not even be the game itself but the engine underneath it: if your agents can generate and balance a playable world, that content pipeline is licensable to other studios, which is a calmer business than hoping for a hit.

## The honest part

Every one of these is now buildable, and that is genuinely new. But buildable is not the same as built into a business. The agent ships the software. It does not ship your first ten customers, the trust an operator places in you, or the taste that decides which of a hundred reasonable choices is the right one. Those are still yours, and on every idea above they are the actual hard part. The email client lives or dies on whether people trust it to send for them. The ERP lives or dies on one operator who vouches for you. The game lives or dies on whether the world is fun, which no verifier can fully judge.

The shift is real, though. A year ago, "I want to build an ERP" was a five-year sentence. Now it is a weekend of careful prompting and a budget for tokens, followed by the same hard, human work of finding people who will pay. The build stopped being the moat. What you choose to point the agents at, and who you can convince to use it, is the whole game now.

So pick the one on this list that you already have an unfair advantage for. The trade you used to work in. The inbox that ruins your day. The Datadog bill that makes your CFO wince. Scope a slice, write the bar like an acceptance test, turn on ultracode, and find out over a weekend what a fleet of agents can actually build for you.

---

*Marco Kotrotsos, specializing in practical AI implementation for organizations ready to close the gap between AI hype and AI value. With 30 years of IT experience now focused purely on AI deployment, he works hands-on with companies to turn AI potential into measurable business outcomes.*

*This article is published in [Autocomplete](https://medium.com/autocomplete-real-world-ai), a Medium publication about real-world AI for practitioners and decision-makers. We're always looking for writers. If you're building with AI and have something worth sharing, reach out.*

*My free Substack newsletter, also called Autocomplete, can be found here: https://acdigest.substack.com.*

*My books on Amazon: [Claude Code for Everyone Else](https://www.amazon.com/dp/B0H35YY851) and [From Vibe to Production](https://www.amazon.com/dp/B0H34GK9VW).*
