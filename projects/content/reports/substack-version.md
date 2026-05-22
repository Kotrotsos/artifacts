# The Claude Code Harness: A Practitioner's Read on Anthropic's Scale Guide

*Anthropic dropped a substantial guide on May 14 covering how Claude Code works in large codebases. It is mandatory reading. It is also dense enough that the most actionable points get lost in the prose. This is the annotated version, save it for your next harness audit.*

![A tall stacked tower of six labeled translucent teal layers on a white platform, with subagents floating beside as a separate capability, all in isometric perspective with the caption the model is one layer of seven](hero.png)

A lot of my client conversations this year have included some version of the question: "we are on the best Claude model. Why is our team not getting the productivity gains we hoped for?" The answer, when I look at the actual setup, is almost always the same. The model is fine. The harness around the model is the problem.

Anthropic just published the playbook for what to do about that. The guide is called "How Claude Code works in large codebases" and it lays out the patterns they have observed across their largest customer deployments. The load-bearing sentence in it is one line: *the harness matters as much as the model*. If you take only one thing from either article, take that.

This newsletter is the practitioner annotation. I am pulling each individual point out, giving it depth from my own deployment experience, marking the common mistakes I see, and ordering by what actually matters on day one.

Save it. Reread it after every major Claude release. The harness work is not a one-time setup.

Reply if you want me to look at your specific configuration. I read everything.

**Three things to know up front:**

Anthropic's guide contains one sentence the rest of the article unpacks: "the harness matters as much as the model." Performance at scale is determined by six layers around the model (CLAUDE.md, hooks, skills, plugins, LSP, MCP) plus one delegation capability (subagents). Teams that focus on the model alone are tuning the wrong knob.

Build order is not optional. The layers compose. CLAUDE.md comes first because nothing else has anywhere to live until it exists. Hooks next, because hooks are how the harness gets self-improving. Skills, plugins, LSP, MCP follow in that order. Subagents come in whenever you need them. Teams that build MCP integrations before CLAUDE.md is solid are building on sand.

Three configuration patterns travel across every successful deployment: make the codebase legible to Claude (layered CLAUDE.md, scoped commands, codebase maps, LSP), keep the harness current (review every 3 to 6 months because models evolve), and assign ownership from day one (a named DRI, a plugin marketplace, eventually an agent manager role). The technical and organizational layers cannot be separated.

## Why agentic search wins, and where it loses

The first technical claim in the Anthropic piece is that Claude Code uses agentic search instead of RAG. The difference is more important than it looks.

![Two side-by-side scenes showing RAG with a stale index returning a renamed function versus agentic search with grep against the live codebase returning the current state](diagram-1-rag-vs-agentic.png)

A RAG-powered AI coding tool embeds the entire codebase ahead of time and retrieves relevant chunks at query time. At small scale this is fine. At large scale it breaks, because the embedding pipeline cannot keep up with active engineering teams committing code. By the time a developer queries the index, the index reflects the codebase as it existed days or weeks ago. Retrieval returns a function the team renamed two weeks ago. The model confidently uses it. The build fails.

Agentic search avoids that failure mode by working from the live codebase. Claude traverses the file system, reads files, runs grep, follows references. No index. No staleness window. Every query operates on what is true right now.

Anthropic flags one tradeoff that the client teams I have worked with consistently underestimate: agentic search works best when Claude has enough starting context to know where to look. If you ask it to find all instances of a vague pattern across a billion-line codebase, you hit the context window before the work begins. The quality of the navigation is shaped entirely by how well the codebase is set up.

This is why the rest of the article matters. The setup is the load-bearing investment.

## The harness matters as much as the model

The capabilities of Claude Code are not the capabilities of the model. The capabilities of Claude Code are the capabilities of the model plus the harness wrapped around it. Six extension points and one delegation capability. The model is one of seven things.

Teams that fixate on model benchmarks are tuning the wrong knob. The benchmark differences between current frontier models are real but small. The differences in harness quality between teams are massive. I have seen two teams running the same model where one team finishes ten times the work because their harness is set up properly and the other is fighting against it.

The six extension points: CLAUDE.md, hooks, skills, plugins, LSP integrations, MCP servers. Plus subagents as a delegation capability. Each gets its own section below, in build order.

## The build order

![A horizontal flow of six numbered nodes from CLAUDE.md through hooks, skills, plugins, LSP, and MCP servers with a note that subagents are available throughout](diagram-2-build-order.png)

The order is not arbitrary. It is the order in which each layer requires the previous one.

## Layer 1: CLAUDE.md

CLAUDE.md files are the foundation. Nothing else has anywhere to live until these are right.

The Anthropic guidance is concise: root file for the big picture, subdirectory files for local conventions. The mistake I see most often is the inverse pattern. Teams pile every convention, every style rule, every "do not do this" rule into a single root CLAUDE.md that grows to 2,000 lines over six weeks. Claude tries to honor all of it, runs out of context space, and the team blames the model.

Three operational rules I have arrived at across client deployments:

**Lean.** The root CLAUDE.md should fit on one screen. Pointers, critical gotchas, and the highest-leverage conventions only. If the root file is more than 80 lines, it is doing too much.

**Layered.** Claude walks up the directory tree and loads every CLAUDE.md file it finds along the way. Put module-specific conventions in module-specific files. Keep the root for things that apply everywhere.

**Audited.** Anthropic recommends reviewing CLAUDE.md every three to six months. I would tighten that: audit after every major model release. A rule that helped Sonnet 4.0 stay on track may actively constrain Sonnet 4.6 from doing something it handles natively.

**Common mistake:** using CLAUDE.md for reusable expertise that belongs in a skill. If the same instruction would help across multiple projects, it is a skill. If it only applies to this codebase, it is CLAUDE.md.

## Layer 2: Hooks

The default use of hooks is guardrails. That is the boring use.

The valuable use, which the Anthropic guide flags clearly, is continuous improvement. A stop hook that runs at session end can reflect on what happened, propose CLAUDE.md updates while context is fresh, and surface skill candidates. A start hook can load team-specific context dynamically.

Three hook patterns worth implementing in your first week:

**The reflection hook.** At session end, ask the agent to summarize what was learned and propose updates. After two weeks of running this in a client deployment, the typical backlog is five to ten proposed CLAUDE.md changes worth reviewing.

**The dynamic context hook.** At session start, detect which module the developer is working in and load the relevant skills. Replaces "every developer configures manually" with "the environment knows itself."

**The enforcement hook.** For deterministic checks like linting and formatting, hooks beat instructions. Telling Claude to run the linter and Claude forgets 15% of the time means that 15% is your bug rate. A hook running unconditionally is 100% reliable.

**Common mistake:** putting things in CLAUDE.md that should be hooks. Instructions get forgotten under context pressure. Hooks do not.

## Layer 3: Skills

Skills are the answer to "I keep needing this expertise but only sometimes." They load on demand, which means they do not compete with CLAUDE.md for the always-loaded budget.

The Anthropic frame is "progressive disclosure." Not everything needs to be in front of Claude at the same time. A security review skill loads when Claude is assessing code for vulnerabilities. A deployment skill loads when work is in the payments service. The path-scoping mechanism is the operationally useful detail: skills can be bound to specific directories so they only auto-load where they apply.

**Common mistake:** loading everything into CLAUDE.md instead of building skills. CLAUDE.md becomes the dumping ground, performance degrades because every session reads 2,000 lines of mostly-irrelevant context, and skills never get built. This is the single biggest configuration pathology I see in client teams.

The fix is mechanical. Any time you find yourself writing a chunk of CLAUDE.md that starts with "when doing X, do Y," ask whether X is a recurring task type. If yes, it is a skill. Move it.

## Layer 4: Plugins

Plugins are the distribution mechanism. They bundle skills, hooks, and MCP configurations into a single installable package.

The point that lands hardest: good setups tend to stay tribal. One engineer figures out a great configuration, the team next to them rebuilds the same thing from scratch six months later, the second version is slightly worse, the cycle continues.

Plugins solve this by making the setup itself a shareable artifact. New engineer day one installs the company plugin. Same context, same skills, same MCP connections as everyone who has been using Claude Code for six months.

**Common mistake:** trying to centrally author every plugin from one team. The pattern that works is letting teams build their own plugins, with a centralized curation function approving what goes into the company-wide marketplace.

## Layer 5: LSP integrations

This is the highest-value-per-effort layer for multi-language codebases. Anthropic flags one customer who deployed LSP integrations org-wide before their Claude Code rollout, specifically to make C and C++ navigation reliable at scale.

Without LSP, Claude pattern-matches on text. Grep for `processOrder` returns dozens of matches across languages, comments, deleted code in stale branches, and different functions with the same name. Claude opens files trying to figure out which one matters. Context burns.

With LSP, the search happens at the symbol level. The language server returns only the references that point to the same symbol. Claude reads what it needs and nothing else. For C, C++, Java, C#, Go, and Rust codebases especially, this is the single highest-leverage configuration change.

**Common mistake:** assuming LSP integration is automatic. It is not. The plugin layer activates it. If your team is on a typed language and not using LSP, the rollout is fighting against itself.

## Layer 6: MCP servers

MCP servers are how Claude connects to internal tools, data sources, and APIs it cannot otherwise reach. The most sophisticated teams build MCP servers that expose structured search as a tool Claude can call directly.

That detail is worth slowing down on. Structured search as an MCP tool means your team's existing code search infrastructure (Sourcegraph, internal indexers) becomes something Claude can query alongside its native grep and file traversal. This is not a replacement for agentic search. It is an addition.

The pattern at the most mature deployments: Notion MCP for product docs and specs. Linear MCP for ticket context. An internal search MCP for code search at scale. A custom MCP for the analytics platform. Each gives Claude access to a source of truth it could not otherwise reach.

**Common mistake the Anthropic guide flags:** building MCP connections before the basics are working. Teams discover MCP, get excited, spend two weeks wiring up integrations with Jira, Confluence, Datadog, Sentry, then wonder why Claude is not producing quality output. The model has access to every tool in the company and no idea what to do with any of it because CLAUDE.md, hooks, and skills were never set up.

MCP is the last layer. Build the others first.

## Subagents

Subagents work differently from the six layers above. They are a delegation capability, available whenever you need them, configured nowhere upfront.

The pattern Anthropic flags is "split exploration from editing." Spin up a read-only subagent to map a subsystem and write findings to a file. Then have the main agent edit with the full picture. The subagent burns its own context window on the mapping work without polluting the editor agent's context.

Two other patterns worth knowing: parallel exploration (three subagents map three different subsystems in parallel; main agent gets three small writeups) and specialized review (a subagent with a security-review skill reviews a change before the main agent commits).

## Three patterns that travel

![Three columns showing the three patterns: make the codebase legible to Claude, keep the harness current as models evolve, and assign ownership from day one](diagram-3-three-patterns.png)

**Make the codebase legible.** Layered CLAUDE.md files, initialization in subdirectories rather than at the repo root, test and lint commands scoped per directory, `.ignore` files for generated content, codebase maps when the directory structure does not do the work, LSP running so search happens at the symbol level. Every one of these is a small investment. The compound effect is what makes Claude Code work at scale.

**Keep the harness current.** Models evolve. Instructions written for the model you have today can work against the model you will have in six months. Anthropic recommends reviewing configurations every three to six months. I would add: do an audit after every major model release, not just on the calendar.

**Assign ownership.** This is the pattern most engineering organizations get wrong. Technical configuration alone does not drive adoption. The rollouts that spread fastest had a dedicated infrastructure investment before broad access. A small team, sometimes just one person, wired up the tooling so Claude already fit developer workflows when they first touched it. The Anthropic guide identifies an emerging role here: the agent manager. A hybrid PM-engineer function dedicated to managing the Claude Code ecosystem.

## What to do this week

If you are running Claude Code with a team and have not done the harness work yet, here is the order I would take it in.

**Day 1.** Audit your CLAUDE.md files. Are they lean? Are they layered? Is the root file under 80 lines? If not, this is the highest-leverage thing you can fix today.

**Day 2.** Identify one feedback hook to build. The reflection hook is the best starting point. Five to ten lines of shell. Run at session end. Output goes to a log the DRI reviews weekly.

**Day 3.** Build or audit your first plugin. If the team has any tribal configuration, package it.

**This month.** Wire up LSP for your primary language if you are on a typed language. Identify one MCP integration that would unblock workflow friction.

**This quarter.** Assign a DRI if you have not. Establish a plugin marketplace. Run the cross-functional working group Anthropic recommends.

## What is coming next

The next two newsletter issues will continue this thread:

- **The reflection hook walkthrough.** The actual shell script that surfaces CLAUDE.md update candidates after every session. Five to ten lines, working with Claude Code, output format that turns into a Monday morning review ritual.
- **Plugin marketplace patterns.** What the successful internal marketplaces look like, who owns curation, the approval flow, and the rollback mechanism. Pulled from three client deployments.

**One open question:** how many CLAUDE.md files are in your repo right now, and is the root one under 80 lines? Be honest. (Mine had 1,200 lines a month ago. Now it has 60. The other 1,140 lines are skill files.)

Until next week.

Marco

---

*Autocomplete is a free weekly newsletter on practical AI implementation. If you are not subscribed yet, the button above this paragraph is the easiest way to fix that. If you are already subscribed, forwarding this to one engineering lead who would benefit is the single best thing you can do for me.*

*Source article: Anthropic, "How Claude Code works in large codebases: Best practices and where to start." May 14, 2026.*
