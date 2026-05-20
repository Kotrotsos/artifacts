# Stop Putting Your Specs in Markdown Files

*Spec-driven development is right. Putting the spec in a spec.md file is wrong. Use Linear (or Jira, or Notion). Use markdown as the portable export. The file is for travel. The board is for living.*

![A small translucent spec.md card on the left and a larger translucent board with ticket cards arranged in three kanban columns on the right, connected by a bidirectional coral arrow, with a caption reading one for travel, one for living](hero.png)

I have spent the last six weeks watching three engineering teams quietly run into the same wall. They all adopted spec-driven development, all in the same shape: spec.md in the repo, agent reads the file, builds against the spec. All three felt great for the first three weeks. By month two, all three were running shadow project management in Slack to reconcile what was in the markdown file with what was actually happening on their Linear board.

The trend is right. The architecture is wrong.

This newsletter is the operational argument. Keep spec-driven development. Move the spec to the platform that was built for it. Use markdown as the portable exchange format, not the system of record. Stop reinventing your ticketing tool in flat files.

Hit reply if your team has been hitting this wall too. I want to hear how you have handled it.

**Three things to know up front:**

Spec-driven development is the right move. Writing a clear spec before writing code is one of the operational shifts that actually compounds in 2026. The debate is not whether to write specs. The debate is where the spec lives.

The default answer that has emerged in the last year (put it in a spec.md file, commit it to the repo, point the agent at it) gets the practice right and the architecture wrong. A spec has lifecycle, ownership, status, dependencies, and history. A flat markdown file has none of those, and reinventing them with prefixes and inline tables is a downgrade from the tools your team already pays for.

The clean architecture is the round-trip. Linear (or Jira, or Notion, or Asana) holds the spec as the source of truth. Agents export to spec.md when they need to read or hand off. Updates flow back. The file is for travel. The board is for living. You stop fighting your project management tool and your AI tooling stops accumulating shadow project management it was never built for.

## What spec-driven development gets right

Let me start by defending the practice, because the rest of this only makes sense if we agree the practice is worth keeping.

Spec-driven development works because agents are bad at intent and great at execution. An agent without a clear spec will produce something. It will probably be plausible. It will probably be wrong in subtle ways you cannot diagnose without reading the entire codebase. The act of writing the spec, ideally in a back-and-forth with the agent, is the part where the misunderstandings get surfaced before they get compiled into 800 lines of code.

The empirical case has been made repeatedly in 2026. PFF's case study from January to March (25x more deploys, 10x output) had the lightweight design document at the center. Anthropic's effective-agents guidance is built around clear goal specification before delegation. Every team I have watched produce real autonomous-agent wins has started by getting better at writing specs.

The practice is settled. The argument I am making is purely about where the spec lives once it exists.

## The unspoken assumption that is hurting you

The default move in the spec-driven community is: put it in a file. spec.md in the repo. CLAUDE.md alongside it. LDD.md for the lightweight design doc. tickets.md for the ticket list. Commit them all. Let the agent read them.

This works for the first three weeks. Then it starts decaying in predictable ways.

The spec.md file does not have a status. Is the spec approved? In review? Stale? Two engineers are reading it; one thinks it is signed off, the other thinks it is still a draft. The file does not tell them.

The spec.md file does not have an owner. Who decides when it changes? Who has authority to accept a change? Who is notified when the spec is updated? Without a system that holds ownership, every update becomes a Slack message: "hey, I changed the spec, can you look at it?"

The spec.md file does not track dependencies. The tickets.md has six things in it. Three of them block the other three. The order matters. The file lists them as bullet points. The dependency graph is in someone's head, or worse, scattered across three Slack threads and a Notion page.

The spec.md file does not have a real history. Yes, you have git history. Git is great for source code. Git history of a spec.md, by month four, is a 47-commit list of "update spec," "more updates," "fix spec." Try to figure out who approved the change from a one-sentence acceptance criterion to a two-paragraph one. Git does not know.

The spec.md file does not interact with the rest of your work surface. Customer issues live in Linear. Bugs live in Linear. Feature requests live in Linear. The spec for the feature also lives in Linear, except for the part of it that lives in spec.md, which is the part the agent reads. Your team now does double bookkeeping for the lifecycle of every piece of work.

I have watched this play out in three engineering orgs in the last two months. The pattern is the same in all three. The spec.md was beautiful in week one. By month three the file and the board had drifted, nobody was sure which was authoritative, and the team was running an unspoken third system in Slack to reconcile them.

## What a spec actually needs

![Five translucent ticket-shaped cards on an isometric platform showing lifecycle, ownership, status, dependencies, and history, with a caption that a flat markdown file does none of these well](diagram-1-needs.png)

**Lifecycle.** A spec moves through states. Draft, in review, approved, in progress, blocked, done, deprecated. If you cannot tell what state a spec is in by looking at it, you cannot act on it confidently.

**Ownership.** Every spec has a person responsible for it. They are notified when it changes, they approve changes, they are the named human someone can ask "is this still the plan." Diffuse ownership is no ownership.

**Status.** Even within a state, the spec has live operational status. Is this currently being worked on? Is it blocked? Is it waiting for review? The status is the thing the team looks at every morning to know what to pick up.

**Dependencies.** Specs do not exist in isolation. This spec depends on that one shipping first. This other spec is blocked on a design decision being made. The dependency graph is part of the spec's metadata, and it has to be visible to be useful.

**History.** Not git history of who typed what. Audit history of who decided what. The spec changed on March 12, proposed by Maya, approved by Tomas, scoped down from the original because of a customer concern raised in a different ticket. This history is what you use to figure out why a feature looks the way it does six months later when someone new joins.

A flat markdown file does none of these well. You can fake any one of them with prefixes and tables and YAML frontmatter. You cannot fake all five without rebuilding Linear, badly, on top of a file system.

## A small story about why this matters

One of the teams I worked with had a beautiful spec.md system in February. By April, the file in the repo said the feature was "in design review." The Linear ticket said it was "ready for development." Three different engineers were operating off three different states. The feature shipped two weeks later than it should have, not because of the work, but because of the time spent arguing about what state the spec was actually in.

That story is unremarkable. It is happening at engineering orgs everywhere right now. The pattern is reliable enough that I have started asking new clients about it as a diagnostic. If you have a spec.md anywhere in your repo that is over three weeks old, there is a 60% chance your team is silently in some version of this drift.

The fix is the round-trip pattern, which I will get to in a minute. First, a defense of the platforms most of you already pay for.

## Why the platforms exist

The ticketing tools we have been using for fifteen years are good at exactly the things spec.md is bad at.

Linear has lifecycle as a first-class concept. Changing the status updates the team automatically. Moving a ticket from In Progress to Blocked tells the assigned engineer's manager that the work is stuck and they should ask why.

Linear has ownership as a first-class concept. Every ticket has an assignee. Subscribing to a project means you get notified when specs you care about change. There is no "I missed the slack message" because the platform is the message.

Linear has dependencies as a first-class concept. You can block one ticket on another. The blocked ticket cannot be marked done until the blocker is. The chain is visible, queryable, and updatable by the whole team in real time.

Linear has audit history as a first-class concept. Every change is recorded, attributed, timestamped. You can reconstruct the decision path on any spec by reading its activity log.

Linear, Jira, Notion, Asana, and the rest have spent ten to fifteen years optimizing for the exact problems that a spec.md file struggles with. The decision to ignore them and reinvent project management as a folder of markdown files is, to be polite, an unforced error.

The reason the AI-engineering community made that choice is understandable. Agents read markdown natively. Agents did not have a Linear API connection by default. The path of least resistance was to put the spec where the agent could read it, which meant a file. That move was a fast workaround for an integration problem, not a principled architectural decision.

Now that the integration problem is largely solved (Linear has an MCP server, Jira has one, every major project tool either has one or will within the quarter), the workaround does not need to be the architecture anymore.

## The right architecture: the round-trip

![Three nodes connected by coral arrows: a Linear-style board labeled source of truth, a spec.md card labeled portable export, and an agent cube labeled executes work, with a caption reading file is for travel, board is for living](diagram-2-round-trip.png)

**Linear holds the spec.** The spec is a Linear project, or a parent ticket with sub-tickets. It has all the metadata Linear gives you for free: status, owner, dependencies, audit log, integration with the rest of your workflow.

**spec.md is generated from Linear.** When an agent needs to work on the spec, it pulls a markdown export from Linear. This is the snapshot. It is the portable, agent-readable, version-pinned representation of the spec at that moment. The file exists for the duration of the agent's work and gets discarded or refreshed when needed.

**The agent reads and writes spec.md.** The agent does its work against the file. It interviews you, refines, generates tickets, builds, tests. All of that happens with markdown as the working medium.

**Updates flow back to Linear.** When the work completes, the agent's output (new tickets, status changes, additional acceptance criteria) flows back to Linear via the MCP server or a small sync step. The board state is updated. Humans see the changes in their normal workflow.

This is the model. The file is for travel. The board is for living. The agent gets the speed and ergonomics of markdown. The team gets the lifecycle, ownership, and history of a real project management tool. Nobody is doing double bookkeeping.

## What this changes about your current setup

![Three columns showing what to stop, what to keep, and what to start: stop list includes spec.md as source of truth and hand-syncing files to tickets, keep list includes spec-driven development and agents that read markdown, start list includes board as system of record and round-trip export](diagram-3-stop-keep-start.png)

**Stop.**

- Treating spec.md as the source of truth. If you also have it in Linear, decide which is authoritative now. Running both creates the drift.
- Hand-syncing files to tickets. Manual sync between markdown and Linear is a workaround that does not need to exist anymore.
- Flat files for shared team work. They were never the right shape for collaborative, lifecycle-bearing artifacts.
- Reinventing Linear in markdown. YAML frontmatter for status, inline tables for dependencies, filename conventions for ownership. All workarounds.

**Keep.**

- Spec-driven development as a practice. The instinct to write things down before building them is correct.
- Agents that read markdown. LLMs read markdown natively and the format is right for agent consumption.
- Lightweight design docs as portable exports. The LDD as a concept is right.
- Markdown as portable exchange format. The format is great for travel.

**Start.**

- Treating the board as the system of record. Linear, Jira, Notion, Asana. Pick the one you already use and put the spec there with all its metadata.
- Round-trip export. Install the MCP server. Configure your agents to export from the board, work against the file, push updates back.
- Exports on demand. The spec.md is a snapshot, not a state. Generate it when you need it, discard or refresh after.
- spec.md as a snapshot. The file is never the truth. The board is the truth.

## What I would do this week

Three concrete moves.

First, take a working session with the team and inventory every markdown file in your repo that contains spec-shaped content. spec.md, LDD.md, tickets.md, requirements.md. For each, decide whether to promote it to Linear, deprecate it entirely, or keep it as a working artifact. Most teams will find they have three to seven of these, and at least half can move.

Second, install the Linear MCP server (or your equivalent). Configure your agents to read Linear projects when working on substantial features. The setup is short. The pattern of "agent reads Linear, agent works against an exported spec.md, agent updates Linear when done" is what makes the rest of this work.

Third, pick one feature your team is about to ship that currently has a spec.md in the repo. Migrate that one feature to the Linear-as-source pattern as a pilot. Run it through to completion. The team will know within two weeks whether the new architecture is better.

Hit reply if you want me to look at your specific setup. I read everything.

## What is coming next

The next two newsletter issues will continue this thread:

- **The Linear MCP setup, end to end.** A practical walkthrough of installing the Linear MCP server, configuring your agents to read and write, and the small sync patterns that make the round-trip reliable.
- **What to do with your CLAUDE.md files.** A related cleanup. Most teams have accumulated three to seven markdown files in the repo that are quietly trying to be different things. The audit pattern for sorting them out.

**One open question for you, if you are willing to share in the comments:** how many markdown files in your current repo are trying to be a system of record for something that has a real lifecycle? Be honest. Mine had four.

Until next week.

Marco

---

*Autocomplete is a free weekly newsletter on practical AI implementation. If you are not subscribed yet, the button above this paragraph is the easiest way to fix that. If you are already subscribed, forwarding this to one engineering lead who would benefit is the single best thing you can do for me.*
