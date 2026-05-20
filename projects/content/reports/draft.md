# Spec-Driven Development Is Right. Spec.md Is the Wrong Container.

*Markdown is great as a portable exchange format between agents. It is a poor system of record for things that have lifecycle, ownership, status, and history. Stop reinventing Linear in flat files.*

![A small translucent spec.md card on the left and a larger translucent board with ticket cards arranged in three kanban columns on the right, connected by a bidirectional coral arrow, with a caption reading one for travel, one for living](hero.png)

**Key takeaways:**

- Spec-driven development is the right move. Writing a clear spec before writing code is one of the operational shifts that actually compounds in 2026. The debate is not whether to write specs. The debate is where the spec lives.
- The default answer that has emerged in the last year (put it in a spec.md file, commit it to the repo, point the agent at it) gets the practice right and the architecture wrong. A spec has lifecycle, ownership, status, dependencies, and history. A flat markdown file has none of those, and reinventing them with prefixes and inline tables is a downgrade from the tools your team already pays for.
- The clean architecture is the round-trip. Linear (or Jira, or Notion, or Asana) holds the spec as the source of truth. Agents export to spec.md when they need to read or hand off. Updates flow back. The file is for travel. The board is for living. You stop fighting your project management tool and your AI tooling stops accumulating shadow project management it was never built for.

---

There is a quiet trend in the AI-engineering community I want to push back on, while keeping the part of it that is correct.

The trend is spec-driven development. Write a clear specification, get the agent to interview you about it, refine the spec, then let the agent build against it. PFF's post-engineering org built their 25x deployment story on this practice. Anthropic's own internal teams do something similar. So do most of the engineering shops I have advised in the last six months. The practice is real, the gains are real, and skipping the spec is the single most expensive mistake I see teams making with autonomous agents.

So the practice is right. The architecture most teams have landed on is wrong.

The architecture most teams have landed on is putting the spec in a file called spec.md, committing it to the repo, and pointing the agent at it. Then a sibling LDD.md. Then a tickets.md. Then the inevitable manual sync from these files to whatever ticketing system the team actually uses for everything else. Then the slow accumulation of "is the file the truth or is the ticket the truth?" confusion. Then the realization six months later that nobody can tell what state any given spec is actually in.

Markdown is a great format. It is a poor system of record. The two roles are not the same role, and conflating them is the trap.

This article is the argument for the cleaner architecture. Keep the spec-driven practice. Move the spec to the platform that was built for it. Use markdown as the portable exchange format, the snapshot, the thing your agent reads and writes. Stop pretending markdown is your ticketing system.

## What spec-driven development gets right

Let me start by defending the practice, because the rest of the article only makes sense if we agree the practice is worth keeping.

Spec-driven development works because agents are bad at intent and great at execution. An agent without a clear spec will produce something. It will probably be plausible. It will probably be wrong in subtle ways you cannot diagnose without reading the entire codebase. The act of writing the spec, ideally in a back-and-forth with the agent, is the part where the misunderstandings get surfaced before they get compiled into 800 lines of code.

The empirical case has been made repeatedly in 2026. PFF's case study from January to March (25x more deploys, 10x output) had the lightweight design document at the center. Anthropic's effective-agents guidance is built around clear goal specification before delegation. Every team I have watched produce real autonomous-agent wins has started by getting better at writing specs.

So the practice is settled. You write the spec first. You let the agent interview you to surface gaps. You distribute the spec for review. You let the agent generate tickets and PRs from it. This is how 2026 engineering teams work, and it is genuinely better than the pre-agent process.

The argument I am making is purely about where the spec lives once it exists.

## The unspoken assumption that is hurting you

The default move in the spec-driven community is: put it in a file. spec.md in the repo. CLAUDE.md alongside it. LDD.md for the lightweight design doc. tickets.md for the ticket list. Commit them all. Let the agent read them. Update them as the work progresses.

This works for the first three weeks. Then it starts decaying in predictable ways.

The spec.md file does not have a status. Is the spec approved? In review? Stale? Two engineers are reading it; one thinks it is signed off, the other thinks it is still a draft. The file does not tell them.

The spec.md file does not have an owner. Who decides when it changes? Who has authority to accept a change? Who is notified when the spec is updated? Without a system that holds ownership, every update becomes a Slack message: "hey, I changed the spec, can you look at it?"

The spec.md file does not track dependencies. The tickets.md has six things in it. Three of them block the other three. The order matters. The file lists them as bullet points. The dependency graph is in someone's head, or worse, scattered across three Slack threads and a Notion page.

The spec.md file does not have a history. Yes, you have git history. Git is great for source code. Git history of a spec.md, by month four, is a 47-commit list of "update spec," "more updates," "fix spec," "address feedback." Try to figure out who approved the change from a one-sentence acceptance criterion to a two-paragraph one. Git does not know. The blame line tells you who typed the change, not who authorized it.

The spec.md file does not interact with the rest of your work surface. Customer issues live in Linear. Bugs live in Linear. Feature requests live in Linear. The roadmap lives in Linear. The spec for the feature also lives in Linear, except for the part of it that lives in spec.md, which is the part the agent reads. Your team now does double bookkeeping for the lifecycle of every piece of work.

These problems are not theoretical. I have watched them play out in three engineering orgs in the last two months. The pattern is the same in all three. The spec.md was beautiful in week one. By month three the file and the Linear board had drifted, nobody was sure which was authoritative, and the team was running an unspoken third system in Slack to reconcile them.

## What a spec actually needs

If you list the things a spec needs in order to function as a real artifact in a real engineering org, the list looks like this.

![Five translucent ticket-shaped cards on an isometric platform showing lifecycle, ownership, status, dependencies, and history, with a caption that a flat markdown file does none of these well](diagram-1-needs.png)

**Lifecycle.** A spec moves through states. Draft, in review, approved, in progress, blocked, done, deprecated. The state matters. If you cannot tell what state a spec is in by looking at it, you cannot act on it confidently.

**Ownership.** Every spec has a person responsible for it. They are notified when it changes, they approve changes, they are the named human someone can ask "is this still the plan." Diffuse ownership is no ownership.

**Status.** Even within a state, the spec has live operational status. Is this currently being worked on? Is it blocked on something? Is it waiting for review? The status is the thing the team looks at every morning to know what to pick up.

**Dependencies.** Specs do not exist in isolation. This spec depends on that one shipping first. This other spec is blocked on a design decision being made. The dependency graph is part of the spec's metadata, and it has to be visible to be useful.

**History.** Not git history of who typed what. Audit history of who decided what. The spec changed on March 12, the change was proposed by Maya, approved by Tomas, the change was scoped down from the original because of a customer concern raised in this ticket. This history is what you use to figure out why a feature looks the way it does six months later when someone new joins.

A flat markdown file in a repo does none of these well. You can fake any one of them with prefixes and tables and YAML frontmatter. You cannot fake all five without rebuilding Linear, badly, on top of a file system.

## What Linear (and Jira, and Notion, and Asana) are good at

This is the part the spec-in-markdown community has under-credited. The ticketing tools we have been using for fifteen years are good at exactly the things spec.md is bad at.

Linear has lifecycle as a first-class concept. The ticket has a status. Changing the status updates the team automatically. Moving a ticket from In Progress to Blocked tells the assigned engineer's manager that the work is stuck and they should ask why.

Linear has ownership as a first-class concept. Every ticket has an assignee. Subscribing to a project means you get notified when specs you care about change. There is no "I missed the slack message" because the platform is the message.

Linear has dependencies as a first-class concept. You can block one ticket on another. The blocked ticket cannot be marked done until the blocker is. The chain is visible, queryable, and updatable by the whole team in real time.

Linear has audit history as a first-class concept. Every change is recorded, attributed, timestamped. You can reconstruct the decision path on any spec by reading its activity log.

Linear, Jira, Notion, Asana, and the rest of the platforms have spent ten to fifteen years optimizing for the exact problems that a spec.md file in a repo struggles with. The decision to ignore them and reinvent project management as a folder of markdown files in your repo is, to be polite, an unforced error.

The reason the AI-engineering community made that choice is understandable. Agents read markdown natively. Agents do not have a Linear API connection by default. The path of least resistance was to put the spec where the agent could read it, which meant a file. That move was a fast workaround for an integration problem, not a principled architectural decision.

Now that the integration problem is largely solved (Linear has an MCP server, Jira has one, every major project tool either has one or will within the quarter), the workaround does not need to be the architecture anymore.

## The right architecture: the round-trip

The clean version is the round-trip.

![Three nodes connected by coral arrows: a Linear-style board labeled source of truth, a spec.md card labeled portable export, and an agent cube labeled executes work, with a caption reading file is for travel, board is for living](diagram-2-round-trip.png)

**Linear holds the spec.** The spec is a Linear project, or a parent ticket with sub-tickets, or whatever structure your team uses for substantial work. It has all the metadata Linear gives you for free: status, owner, dependencies, audit log, integration with the rest of your workflow.

**spec.md is generated from Linear.** When an agent needs to work on the spec, it pulls a markdown export from Linear. This is the snapshot. It is the portable, agent-readable, version-pinned representation of the spec at that moment. The file exists for the duration of the agent's work and gets discarded or refreshed when needed.

**The agent reads and writes spec.md.** The agent does its work against the file. It interviews you, refines, generates tickets, builds, tests. All of that happens with markdown as the working medium. The agent never directly modifies Linear in the middle of a long autonomous loop, because Linear is the source of truth and you do not want an agent partial-writing into it.

**Updates flow back to Linear.** When the work completes, the agent's output (new tickets, status changes, additional acceptance criteria) flows back to Linear via the MCP server or a small sync step. The board state is updated. Humans see the changes in their normal workflow. The next agent run pulls the fresh markdown from the updated board.

This is the model. The file is for travel. The board is for living. The agent gets the speed and ergonomics of markdown. The team gets the lifecycle, ownership, and history of a real project management tool. Nobody is doing double bookkeeping.

The piece that makes this work is the bidirectional MCP layer. Linear's MCP server is the obvious one for Linear shops. Jira has equivalent. Notion has equivalent. The pattern is: the agent never owns the source of truth. The agent operates on exports. The exports are first-class artifacts produced and consumed on demand.

## What this changes about your current setup

If you are running spec-driven development today and your spec lives in a markdown file in the repo, three things to do this month.

![Three columns showing what to stop (spec.md as source of truth, hand-syncing files to tickets), what to keep (spec-driven development, agents that read markdown), and what to start (board as system of record, round-trip export)](diagram-3-stop-keep-start.png)

**Stop treating spec.md as the source of truth.** Audit your repo. Every spec.md, every LDD.md, every tickets.md. For each one, ask: is this also tracked in Linear, Jira, or Notion? If yes, decide which is authoritative. If the answer feels uncomfortable, that is because you have been running two systems and nobody admitted it.

**Stop hand-syncing files to tickets.** If your current workflow involves a human reading the markdown spec, going to Linear, and copying the structure across, you have built a manual sync layer that exists only because the agent could not talk to Linear. That constraint is gone. Stop paying the manual sync tax.

**Stop reinventing project management in flat files.** YAML frontmatter for status. Inline tables for dependencies. Filename conventions for ownership. These are workarounds for not using the right tool. The right tool exists, you already pay for it, you are just not pointing your agent at it.

**Keep spec-driven development as a practice.** Writing a clear spec before writing code is correct. Letting the agent interview you to refine the spec is correct. Distributing the spec for review is correct. None of this changes. The practice is the part that compounds.

**Keep agents that read markdown.** Markdown is the right input format for an agent. LLMs read it natively, it is portable, version-controllable, and human-readable. The agent should keep reading markdown. The change is that the markdown is now generated from the board, not authored as a standalone file.

**Keep lightweight design docs as portable exports.** The LDD as a concept is correct. PFF's case study showed why. The lightweight design doc is a great working artifact. It just does not need to live in a file in your repo as the authoritative version. Generate it from the board, work against it, push updates back.

**Start using the board as the system of record.** If you use Linear, the spec is a Linear project. If you use Jira, it is an epic with sub-tasks. If you use Notion, it is a database row with linked sub-pages. Pick the tool you already use and put the spec there with all the metadata it deserves.

**Start using the round-trip pattern.** Install the MCP server for your project tool. Configure your agents to export from the board to spec.md when they need to work, and to push updates back when the work changes the spec. This is one afternoon of setup that pays back every week thereafter.

**Start treating spec.md as a snapshot, not a state.** When the file exists, it is a snapshot of the board at a moment in time. The agent's job is to make that snapshot useful for the next chunk of work, then discard it or refresh it. The snapshot is never the truth. The board is the truth.

## What I would do this week

Three concrete moves.

First, take a working session with the team and inventory every markdown file in your repo that contains spec-shaped content. spec.md, LDD.md, tickets.md, requirements.md, anything with structured task or feature content. For each, decide whether it should be promoted to Linear, deprecated entirely, or kept as a working artifact. Most teams will find they have three to seven of these, and at least half can move.

Second, install the Linear MCP server (or your equivalent). Configure your agents to read Linear projects when working on substantial features. The setup is short. The pattern of "agent reads Linear, agent works against an exported spec.md, agent updates Linear when done" is what makes the rest of this work.

Third, pick one feature your team is about to ship that currently has a spec.md in the repo. Migrate that one feature to the Linear-as-source pattern as a pilot. Run it through to completion. Observe. The team will know within two weeks whether the new architecture is better. In every case I have done this, the answer has been yes, by a wide margin, and the team never wants to go back.

## Closing

The spec-driven trend is correct. The instinct to write things down before building them, to let the agent interview you, to generate tickets from a clear specification, is the operational shift that actually compounds in 2026. None of that is in question.

The mistake is the architectural one of confusing the spec with the file that holds it. Markdown is a great working medium and a poor system of record. Linear (and the other platforms) have been good at being a system of record for over a decade. Use both for what they are good at, with the round-trip between them as the integration pattern, and you stop running shadow project management in your repo.

There is still a place for markdown in spec-driven development. The place is for travel, not for living. The file moves between systems and agents. The board is where the spec actually lives.

Stop putting your specs in markdown only. Use the platform you already pay for. Your future self, the next engineer who joins your team, and the auditor who shows up six months from now will all thank you.

---

*Marco Kotrotsos, specializing in practical AI implementation for organizations ready to close the gap between AI hype and AI value. With 30 years of IT experience now focused purely on AI deployment, he works hands-on with companies to turn AI potential into measurable business outcomes.*

*This article is published in [Autocomplete](https://medium.com/autocomplete-real-world-ai), a Medium publication about real-world AI for practitioners and decision-makers. We're always looking for writers. If you're building with AI and have something worth sharing, reach out.*

*My free Substack newsletter, also called Autocomplete, can be found here: https://acdigest.substack.com.*
