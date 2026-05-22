# How Claude Code Actually Works at Scale: The Six Layers and the Build Order

*Anthropic published the playbook for Claude Code at scale on May 14. Here is the practitioner annotation, with the build order, the common mistakes, and the calls that have to be made on day one. Save this if you are setting up Claude Code for a team larger than one.*

![A tall stacked tower of six labeled translucent teal layers on a white platform, with subagents floating beside as a separate capability, all in isometric perspective with the caption the model is one layer of seven](hero.png)

**Key takeaways:**

- Anthropic's guide contains one sentence the rest of the article unpacks: "the harness matters as much as the model." Performance at scale is determined by six layers around the model (CLAUDE.md, hooks, skills, plugins, LSP, MCP) plus one delegation capability (subagents). Teams that focus on the model alone are tuning the wrong knob.
- Build order is not optional. The layers compose. CLAUDE.md comes first because nothing else has anywhere to live until it exists. Hooks next, because hooks are how the harness gets self-improving. Skills, plugins, LSP, MCP follow in that order. Subagents come in whenever you need them. Teams that build MCP integrations before CLAUDE.md is solid are building on sand.
- Three configuration patterns travel across every successful deployment: make the codebase legible to Claude (layered CLAUDE.md, scoped commands, codebase maps, LSP), keep the harness current (review every 3 to 6 months because models evolve), and assign ownership from day one (a named DRI, a plugin marketplace, eventually an agent manager role). The technical and organizational layers cannot be separated.

---

Anthropic dropped a substantial guide on May 14 called "How Claude Code works in large codebases." It is the first installment of a series on Claude Code at enterprise scale. If you are running Claude Code in any environment larger than a side project, the guide is mandatory reading. It is also, like most Anthropic technical writing, dense enough that the most actionable insights get lost in the prose.

This is the annotated version. I am pulling each individual point out, giving it depth, marking the common mistakes I see in client deployments, and ordering the whole thing by what actually matters on day one.

You will get more from this article if you read Anthropic's piece first or alongside it. The original is at the Anthropic news site under "How Claude Code works in large codebases." This piece is the practitioner read.

## Why agentic search wins, and where it loses

The first technical claim in the Anthropic piece is that Claude Code uses agentic search instead of RAG. The difference is more important than it looks.

![Two side-by-side scenes showing RAG with a stale index returning a renamed function versus agentic search with grep against the live codebase returning the current state](diagram-1-rag-vs-agentic.png)

A RAG-powered AI coding tool embeds the entire codebase ahead of time and retrieves relevant chunks at query time. At small scale this is fine. At large scale it breaks, because the embedding pipeline cannot keep up with active engineering teams committing code. By the time a developer queries the index, the index reflects the codebase as it existed days or weeks ago. Retrieval returns a function the team renamed two weeks ago. The model confidently uses it. The build fails.

Agentic search avoids that failure mode by working from the live codebase. Claude traverses the file system, reads files, runs grep, follows references. No index. No staleness window. Every query operates on what is true right now.

Anthropic flags one tradeoff that the client teams I have worked with consistently underestimate: agentic search works best when Claude has enough starting context to know where to look. If you ask it to find all instances of a vague pattern across a billion-line codebase, you hit the context window before the work begins. The quality of the navigation is shaped entirely by how well the codebase is set up.

This is why the rest of the article matters. The setup is the load-bearing investment.

## The harness matters as much as the model

This is the central claim of the Anthropic piece. Everything else in the harness rollout flows from it.

The capabilities of Claude Code are not the capabilities of the model. The capabilities of Claude Code are the capabilities of the model plus the harness wrapped around it. Six extension points and one delegation capability. The model is one of seven things.

Teams that fixate on model benchmarks ("which model scores best on HumanEval?") are tuning the wrong knob. The benchmark differences between current frontier models are real but small. The differences in harness quality between teams are massive. I have seen two teams running the same model where one team finishes ten times the work because their harness is set up properly and the other is fighting against it.

The harness has six extension points:

1. **CLAUDE.md** files (context that loads every session)
2. **Hooks** (scripts triggered by events)
3. **Skills** (packaged expertise loaded on demand)
4. **Plugins** (bundles of the above, distributable)
5. **LSP integrations** (symbol-level code intelligence)
6. **MCP servers** (connections to external tools and data)

Plus one delegation capability:

7. **Subagents** (isolated Claude instances with their own context)

Each of these gets a section below, in build order. Build order matters. Skipping ahead breaks things.

## Layer 1: CLAUDE.md

CLAUDE.md files are the foundation. Nothing else has anywhere to live until these are right.

![A horizontal flow of six numbered nodes from CLAUDE.md through hooks, skills, plugins, LSP, and MCP servers with a note that subagents are available throughout](diagram-2-build-order.png)

The Anthropic guidance is concise: root file for the big picture, subdirectory files for local conventions. The mistake I see most often is the inverse pattern. Teams pile every convention, every style rule, every "do not do this" rule into a single root CLAUDE.md that grows to 2,000 lines over six weeks. Claude tries to honor all of it, runs out of context space, and the team blames the model.

Three operational rules for CLAUDE.md that I have arrived at across client deployments:

**Lean.** The root CLAUDE.md should fit on one screen. Pointers, critical gotchas, and the highest-leverage conventions only. If a rule applies to one subdirectory, it belongs in that subdirectory's CLAUDE.md. Anthropic calls this "layered." I would go further: if the root file is more than 80 lines, it is doing too much.

**Layered.** Claude walks up the directory tree and loads every CLAUDE.md file it finds along the way. That means a CLAUDE.md in `src/payments/` loads automatically when Claude is working in that directory, and the root CLAUDE.md still loads too. Use this. Put module-specific conventions in module-specific files. Keep the root for things that apply everywhere.

**Audited.** Anthropic recommends reviewing CLAUDE.md every three to six months because models evolve. I would tighten that: audit after every major model release. A rule that helped Sonnet 4.0 stay on track may actively constrain Sonnet 4.6 from doing something it handles natively. The "do not refactor across multiple files in one go" rule that helped early models is exactly the kind of thing that hurts current models.

**Common mistake the Anthropic guide flags:** using CLAUDE.md for reusable expertise that belongs in a skill. If the same instruction would help across multiple projects, it is a skill. If it only applies to this codebase, it is CLAUDE.md. The distinction matters because skills load on demand, while CLAUDE.md loads every session. Stuffing skill-shaped content into CLAUDE.md is the most common reason teams hit context limits early.

## Layer 2: Hooks

The default use of hooks is guardrails: block the agent from running `rm -rf`, prevent commits to main. That is the boring use.

The valuable use, which the Anthropic guide flags clearly, is continuous improvement. A stop hook that runs at the end of every session can reflect on what happened, propose CLAUDE.md updates while the context is fresh, and surface skill candidates. A start hook can load team-specific context dynamically so every developer gets the right setup for their module without manual configuration.

The shift in framing: hooks are not a constraint layer, they are a feedback layer. The harness becomes self-improving when you use them correctly.

Three hook patterns worth implementing in your first week:

**The reflection hook.** At session end, run a small script that asks the agent to summarize what was learned during the session and propose updates to CLAUDE.md or new skill candidates. After two weeks of running this in a client deployment, the typical backlog is five to ten proposed CLAUDE.md changes worth reviewing.

**The dynamic context hook.** At session start, detect which module the developer is working in and load the relevant skill set or extra context. This replaces "every developer manually configures their environment" with "the environment knows itself."

**The enforcement hook.** Anthropic is explicit on this one: for deterministic checks like linting and formatting, hooks beat instructions. If you tell Claude to run the linter and Claude forgets 15% of the time, that 15% is your bug rate. A hook that runs the linter unconditionally is 100% reliable.

**Common mistake:** putting things in CLAUDE.md that should be hooks. If the rule is "always run the linter before committing," that is not a CLAUDE.md instruction. That is a hook. Instructions get forgotten under context pressure. Hooks do not.

## Layer 3: Skills

Skills are the answer to "I keep needing this expertise but only sometimes." They load on demand, which means they do not compete with CLAUDE.md for the always-loaded budget.

The Anthropic frame is "progressive disclosure." The same idea applies to Claude that applies to good documentation: not everything needs to be in front of the reader at the same time. A security review skill loads when Claude is assessing code for vulnerabilities. A document processing skill loads when documentation needs updating. A deployment skill loads when the work is in the payments service.

The path-scoping mechanism Anthropic mentions is the operationally useful detail. Skills can be bound to specific directories, so they only auto-load in the part of the codebase they apply to. A team that owns a payments service can bind their deployment skill to that directory. It never wakes up when someone is working elsewhere in the monorepo.

This solves the "everyone has the right context, automatically" problem cleanly. You do not have to tell developers which skills to enable. The path activates the right ones.

**Common mistake the Anthropic guide flags:** loading everything into CLAUDE.md instead of building skills. The downstream effect is that CLAUDE.md becomes the dumping ground, performance degrades because every session reads 2,000 lines of mostly-irrelevant context, and skills never get built because "we already have it in CLAUDE.md." This is the single biggest configuration pathology I see in client teams.

The fix is mechanical: any time you find yourself writing a chunk of CLAUDE.md that starts with "when doing X, do Y," ask whether X is a recurring task type. If yes, that is a skill. Move it.

## Layer 4: Plugins

Plugins are the distribution mechanism. They bundle skills, hooks, and MCP configurations into a single installable package. The Anthropic point that lands hardest: good setups tend to stay tribal. One engineer figures out a great configuration, the team next to them rebuilds the same thing from scratch six months later, the second team's version is slightly worse, and the cycle continues.

Plugins solve this by making the setup itself a shareable artifact. New engineer day one installs the company plugin. They have the same context, the same skills, the same MCP connections as everyone who has been using Claude Code for six months.

The example Anthropic gives is concrete and worth pulling out: a large retail organization built a skill that connects Claude to their internal analytics platform so business analysts could pull performance data without leaving their workflow. They distributed it as a plugin before the broader rollout. That sequence (build the skill, distribute as plugin, then roll out broadly) is the pattern.

The corollary I would add: every organization needs a plugin marketplace, or at minimum a curated internal registry. Without that, you are back to tribal knowledge with extra steps. The plugin format helps but is not sufficient if every team is publishing to their own random location.

**Common mistake:** trying to centrally author every plugin from a single team. The pattern that works is letting teams build their own plugins, with a centralized curation function approving what goes into the company-wide marketplace. This is the same pattern that has worked for VS Code extensions, Slack apps, and every other plugin ecosystem at scale.

## Layer 5: LSP integrations

This is the highest-value-per-effort layer for multi-language codebases. Anthropic flags one customer who deployed LSP integrations org-wide before their Claude Code rollout, specifically to make C and C++ navigation reliable at scale.

Without LSP, Claude pattern-matches on text. Grep for `processOrder` in a large codebase returns dozens of matches across languages, comments, deleted code in stale branches, and different functions with the same name in different services. Claude opens files trying to figure out which `processOrder` matters. Context burns.

With LSP, the search happens at the symbol level. The language server returns only the references that point to the same symbol. Claude reads what it needs and nothing else. For C, C++, Java, C#, Go, and Rust codebases especially, this is the single highest-leverage configuration change you can make.

The setup detail Anthropic does not emphasize but matters: LSP servers run locally and need to be installed and running before Claude can use them. The plugin layer is where you wire this up. Most teams install the language server plugin for their primary language as part of their company plugin. Done once, available everywhere.

**Common mistake:** assuming LSP integration is automatic. It is not. The plugin layer activates it. If your team is on a typed language and not using LSP integration, the rollout is fighting against itself.

## Layer 6: MCP servers

MCP servers are how Claude connects to internal tools, data sources, and APIs it cannot otherwise reach. Anthropic flags that the most sophisticated teams build MCP servers that expose structured search as a tool Claude can call directly.

That detail is worth slowing down on. Structured search as an MCP tool means your team's existing code search infrastructure (Sourcegraph, internal indexers, whatever you use) becomes something Claude can query alongside its native grep and file traversal. This is not a replacement for agentic search. It is an addition. Claude gets a faster, smarter search tool for the queries that benefit from one, while still falling back to agentic traversal for everything else.

The pattern at the most mature deployments looks like this: Notion MCP for product docs and specs. Linear MCP for ticket context. An internal search MCP for code search at scale. A custom MCP for the analytics platform. Each gives Claude access to a source of truth it could not otherwise reach.

The Anthropic guide also flags a common mistake here: building MCP connections before the basics are working. This is exactly right and exactly what I see. Teams discover MCP, get excited, spend two weeks wiring up integrations with Jira, Confluence, Datadog, and Sentry, then wonder why Claude is not producing quality output. The model has access to every tool in the company and no idea what to do with any of it because CLAUDE.md, hooks, and skills were never set up.

MCP is the last layer. Build the others first.

## Capability: Subagents

Subagents work differently from the six layers above. They are a delegation capability, available whenever you need them, configured nowhere upfront.

The pattern Anthropic flags is "split exploration from editing." Spin up a read-only subagent to map a subsystem and write findings to a file. Then have the main agent edit with the full picture. The subagent burns its own context window on the mapping work without polluting the editor agent's context.

This pattern is more powerful than it looks. The main reason teams hit context limits on complex changes is that they are doing exploration and editing in the same conversation. Every file the agent reads to understand the system stays in context, even after the work has moved on. Subagents fix this by making exploration disposable.

Two other subagent patterns worth knowing:

**Parallel exploration.** Three subagents map three different subsystems in parallel. The main agent gets three small writeups instead of trying to read all three subsystems itself. Throughput multiplies, context cost stays low.

**Specialized review.** A subagent with a security-review skill reviews a change before the main agent commits. Findings come back as a file. The main agent reads the findings, addresses them, and ships. The security expertise is encapsulated and reusable across sessions.

## The build order matters

The Anthropic guide is explicit that the layers build on each other. The order matters, and skipping ahead breaks things.

The order:

1. **CLAUDE.md.** Without this, nothing else has anywhere to anchor.
2. **Hooks.** Once CLAUDE.md exists, hooks make it self-improving.
3. **Skills.** Once hooks are surfacing patterns, skills externalize the recurring ones.
4. **Plugins.** Once skills exist worth sharing, plugins distribute them.
5. **LSP.** Once the plugin layer is mature, LSP integrations belong there.
6. **MCP servers.** Once the rest of the harness is solid, MCP extends to external tools.

Subagents are available throughout. Use them when the work needs delegated exploration.

Teams I have advised who tried to build in a different order all hit the same wall. MCP-first teams have great tool integration and terrible context management. Skill-heavy teams without CLAUDE.md discipline produce skills that conflict with each other and bloat the harness. The order is not arbitrary. It is the order in which each layer requires the previous one.

## Three configuration patterns that travel

The second half of the Anthropic guide identifies three patterns that show up consistently across successful deployments. These are worth treating as a checklist.

![Three columns showing the three patterns: make the codebase legible to Claude, keep the harness current as models evolve, and assign ownership from day one](diagram-3-three-patterns.png)

**Pattern 1: Make the codebase legible.** Layered CLAUDE.md files, initialization in subdirectories rather than at the repo root, test and lint commands scoped per directory, `.ignore` files for generated content, codebase maps when the directory structure does not do the work, LSP running so search happens at the symbol level. Every one of these is a small investment. The compound effect is what makes Claude Code work at scale.

**Pattern 2: Keep the harness current.** Models evolve. Instructions written for the model you have today can work against the model you will have in six months. Anthropic recommends reviewing configurations every three to six months. I would add: do an audit after every major model release, not just on the calendar. The signal that you need an audit is also operational: when performance plateaus after a model release where the model itself clearly got better, the harness is probably the thing holding you back.

**Pattern 3: Assign ownership.** This is the pattern most engineering organizations get wrong. Technical configuration alone does not drive adoption. The rollouts that spread fastest had a dedicated infrastructure investment before broad access. A small team, sometimes just one person, wired up the tooling so Claude already fit developer workflows when they first touched it.

The Anthropic guide identifies an emerging role here: the **agent manager**. A hybrid PM-engineer function dedicated to managing the Claude Code ecosystem. Most organizations do not have this yet. The minimum viable version is a named DRI with authority over settings, permissions policy, the plugin marketplace, and CLAUDE.md conventions. Without that, knowledge stays tribal and adoption plateaus.

## What to do this week

If you are running Claude Code with a team and have not done the harness work yet, here is the order I would take it in.

**Day 1.** Audit your CLAUDE.md files. Are they lean? Are they layered? Is the root file under 80 lines? If not, this is the highest-leverage thing you can fix today. Move module-specific content to subdirectory files. Move task-specific patterns to skills (you may need to create the skill files; do it as you go).

**Day 2.** Identify one feedback hook to build. The reflection hook is usually the best starting point. Five to ten lines of shell. Run at session end. Output goes to a log file the DRI reviews weekly. Within two weeks you will have a list of CLAUDE.md updates and skill candidates surfaced from real sessions.

**Day 3.** Build or audit your first plugin. If the team has any tribal configuration, package it. Make it installable for new developers in one command. This is the move that prevents the next six months of every new hire reinventing the harness.

**This month.** Wire up LSP for your primary language if you are on C, C++, Java, C#, Go, or Rust. Identify one MCP integration that would unblock the most workflow friction (usually Notion or Linear, occasionally an internal data source). Build it after the basics, not before.

**This quarter.** Assign a DRI if you have not. Establish a plugin marketplace or curated registry. Run the cross-functional working group Anthropic recommends with information security and governance.

## Closing

The framing in the title of this section is the whole claim of the article: the harness matters as much as the model. Teams that act on this and invest in the six layers in the right order will run circles around teams running the same model with worse setup.

Save this article. Save Anthropic's. Reread both every quarter, after every major model release, and whenever performance feels like it has plateaued. The harness is the part of your engineering organization's AI tooling that you will continuously evolve for as long as Claude Code is in production. The teams that do this work get the gains. The teams that skip it stay frustrated and blame the model.

The model is one layer of seven.

---

*Marco Kotrotsos, specializing in practical AI implementation for organizations ready to close the gap between AI hype and AI value. With 30 years of IT experience now focused purely on AI deployment, he works hands-on with companies to turn AI potential into measurable business outcomes.*

*This article is published in [Autocomplete](https://medium.com/autocomplete-real-world-ai), a Medium publication about real-world AI for practitioners and decision-makers. We're always looking for writers. If you're building with AI and have something worth sharing, reach out.*

*My free Substack newsletter, also called Autocomplete, can be found here: https://acdigest.substack.com.*

*Source article: Anthropic, "How Claude Code works in large codebases: Best practices and where to start." May 14, 2026. Read it at anthropic.com.*
