# Skills, MCP, and Tool Calling: The Three Layers of Agent Extension

*Skills are loud right now. Building agents on Skills alone gives you half an agent. Here is what each of the three layers actually does, when to use which, and why "just write a Skill" is the most common architectural mistake I see in 2026.*

![Three stacked layers, the bottom labeled tools, the middle labeled MCP, the top labeled Skills, rendered as a clean isometric stack of teal slabs with coral side icons](hero.png)

**Key takeaways:**

- Three different extension mechanisms get conflated in 2026 because they all touch the same problem: making an agent do something useful. They are not interchangeable. Tool calling is the agent's hands. MCP is its outlets to external systems. Skills are its playbooks. Each one solves a problem the other two cannot.
- The Skills hype is loud because Skills shipped most recently (October 2025) and they are easy to write. The hype is also misleading. Skills cannot authenticate to your database. They cannot maintain state. They cannot expose one capability to every agent in your fleet at once. The teams treating Skills as the universal answer are quietly hitting walls.
- The 2026 architecture that actually works is hybrid. Skills provide the procedure. MCP provides the connectors with auth, state, and isolation. Tools provide the deterministic local operations. The decision is not "Skill or MCP" but "Skill for the how, MCP for the where, Tool for the what."

---

If you have spent any time inside the Claude or AI-engineering communities in the last six months, you have watched a quiet civil war play out. One camp is shipping Skills for everything. Another camp is wiring up MCP servers. A third camp is hand-rolling tools through the agent's tool-use API and ignoring both of the new shiny things. Most of the writing about this conflict is either tribal (one camp arguing the other is wrong) or vague (one paragraph each on three things, with no decision framework).

The actual answer is mundane. The three mechanisms are complementary, not competing. They solve different problems, and teams that have shipped real agents in production are using all three, often in the same agent.

This article gives you the clean version of the mental model, the decision tree, and the answer to the question I get most: "Why can't I just write a Skill for everything?"

The short answer is that Skills are instructions. They are not capability. If your agent needs to authenticate to a real database, talk to a SaaS API, run a sandboxed script, or expose one piece of logic to ten different agent flows, a Skill on its own will not get you there.

Let me walk through what each layer is, and then how to decide which one to reach for.

## What each layer actually is

The conflation happens because all three of these get pitched the same way in the docs. "Extend your agent." "Give Claude new capabilities." "Plug in to your stack." The pitch is the same. The mechanics are entirely different.

![Three columns showing the anatomy of tool calling (a JSON-like rectangle), MCP server (a box with outlet and data arrows), and Agent Skill (a manila folder with SKILL.md), each with a one-line description of what it is](diagram-1-anatomy.png)

### Tool calling: the agent's hands

A tool, in the modern agent sense, is a function the model can decide to invoke. It has a name, a description, an input schema (usually JSON Schema), and a returned result. The agent reads the description, decides the situation matches, calls the tool with arguments, gets a response back, and continues reasoning.

Tools are the foundation. Every other mechanism on this list is built on top of tool calling, or wraps it. When Claude Code reads a file, it is calling the built-in `Read` tool. When it runs a shell command, that is the `Bash` tool. Tool calling has been the agent extensibility primitive since GPT-4 introduced function-calling in mid-2023, and it remains the lowest-level interface to "the agent can do this specific thing."

Tools are bounded. They run a defined function with defined inputs and return a defined output. They are local in the sense that they run in the agent's runtime, not on a remote server. When you write a tool, you are writing code the agent can call, not a server the agent can connect to.

### MCP: the agent's outlets

Model Context Protocol is the protocol Anthropic published in November 2024 and open-sourced under Apache 2.0. The mental model is plumbing. MCP defines how a server (a program that exposes some capability) talks to a client (the AI application that wants to use that capability). The protocol is JSON-RPC 2.0 over either stdio (for local servers) or HTTP and SSE (for remote ones).

An MCP server exposes three kinds of things: tools (functions the client can call), resources (data the client can read), and prompts (templates the client can use). The agent on the other end gets a standardized way to discover what is available and how to call it.

The point of MCP is that the integration is built once and works for any client that speaks the protocol. Build a GitHub MCP server, and every agent that speaks MCP can use GitHub. Build a Notion MCP server, and you can wire it up to Claude Desktop, Claude Code, Cursor, Continue, and any future agent that adopts MCP. The protocol turns "we integrated GitHub into our agent" into "we exposed GitHub through a standard outlet."

MCP also handles three things that pure tool calling does not. Authentication and credentials live inside the server boundary, which means the agent never sees your API keys. State can be maintained across calls, because the server is its own process. Multi-tenant isolation is built in, because each tenant can have its own server instance with its own credentials.

### Agent Skills: the agent's playbooks

Anthropic shipped Agent Skills in October 2025. A Skill is a folder. Inside the folder is a markdown file called SKILL.md with YAML frontmatter (name and description) and a markdown body with instructions. Optionally, the folder includes reference files, scripts, and example outputs.

The Skill is loaded lazily. At session start, the agent reads only the metadata (the description). The full content loads only when the agent decides the situation matches. This progressive disclosure is the design trick that lets you have hundreds of Skills installed without bloating context.

Skills are not capability. They are configured procedural knowledge. A Skill named `pr-review` does not give the agent the ability to read code or post comments on GitHub. It gives the agent a calibrated procedure for reviewing pull requests: what to look for, what to flag, how to format the feedback, what voice to use. The agent already had to be able to read files and post comments (via Tools or MCP). The Skill tells it how to use those capabilities well for this specific task.

The mental model that lands cleanly: a Skill teaches Claude what to do. A Tool gives Claude something to do it with. An MCP server gives Claude somewhere to do it.

## When to use which

Here is the actual decision tree. Three questions, in order.

![A decision tree showing three questions: does the agent already have this capability, does it need to reach an external system, does it need a new local capability, with arrows pointing to USE A SKILL, USE MCP, or USE A TOOL respectively](diagram-2-decision.png)

**Question 1: Does the agent already have this capability built in?**

If yes, you do not need a new capability layer. You need a procedural layer. Write a Skill. Example: Claude Code already has `Read`, `Write`, `Edit`, `Bash`, and `WebFetch` tools built in. If the task is "review my pull requests against the style guide," the agent has everything it needs already. The job is to encode the procedure (what to check, what to flag, what format to output). That is a Skill.

**Question 2: Does the agent need to reach an external system?**

If yes, reach for MCP. Example: the agent needs to read Jira tickets, query your production database, post to Slack, or look up customer records in your CRM. Each of those is an external system with its own authentication, its own schema, its own rate limits. The right answer is an MCP server that wraps that system and exposes it to the agent through the protocol. The agent gets a standardized way to call it. The credentials stay on the server side. If the API changes, you update the server and every agent that uses it benefits.

**Question 3: Does the agent need a new bounded local capability?**

If yes, write a tool. Example: the agent needs to parse a specific file format you have, run a calculation, or perform a deterministic transformation that does not warrant a full external server. A custom tool, defined in the agent's tool list at runtime, is the right shape.

The hidden fourth question, which most teams skip: **does this workflow need multiple of the above?** It almost always does. A real workflow rarely fits cleanly into one layer.

## The hybrid pattern most real workflows use

Take a concrete example: "Fix a bug reported in Linear, in our production repo, with a tested PR."

![A diagram showing an agent in the center connected to three clusters: a bug-fixer Skill at the top, GitHub and Linear MCP servers to the right, and Read/Write/Bash tools at the bottom, with annotations showing how each layer contributes to the workflow](diagram-3-architecture.png)

You need three things. You need the procedure (read the issue, reproduce the bug, write the fix, verify with tests, open a PR with a proper description). You need access (the Linear issue, the repo, the PR system). You need execution (read files, run tests, write the fix).

If you build this with Skills alone, you cannot read the Linear ticket, because Skills cannot authenticate to Linear. You also cannot post a PR, because Skills cannot post to GitHub.

If you build this with MCP alone, the agent has access but no procedure. It can hit the GitHub API and the Linear API all day, but without a calibrated workflow it will write inconsistent PRs, miss test runs, and produce different bug fixes for the same root cause depending on which prompt triggers it.

If you build this with tools alone, you have to hand-roll an integration for every external system, and your tool list grows linearly with every new system the agent touches.

The right architecture for this workflow uses all three:

- A `bug-fixer` Skill that defines the procedure: read, reproduce, fix, verify, open PR. It also encodes your conventions: how branch names are formatted, what the PR description must include, when to wait for review versus self-merge.
- A GitHub MCP server and a Linear MCP server that provide authenticated access to the relevant systems.
- The built-in Read, Write, Edit, and Bash tools that give the agent the local execution hands.

Skill for the how. MCP for the where. Tools for the what.

## Why "just Skills" is the most common architectural mistake

Skills are loud right now for understandable reasons. They are the newest of the three mechanisms, they look easy because they are just markdown, and the file format makes great LinkedIn content. The teams that have only just discovered Skills tend to overcorrect and try to push everything through them.

Five specific things that Skills cannot do, which I see teams discover the hard way.

**Skills cannot authenticate.** A Skill is an instruction pack. It does not hold credentials, it cannot refresh tokens, it cannot make authenticated API calls on its own. The moment your workflow involves a real external system with real auth, you are out of Skills territory.

**Skills cannot maintain state.** A Skill is loaded into the agent's context each time it activates. There is no persistent process. If you need to remember session state across calls, you need either a Tool that manages it locally or an MCP server that holds it.

**Skills cannot enforce isolation.** If you have multiple tenants, multiple environments, or multiple credentials, you cannot use a Skill as your security boundary. Skill instructions can be ignored by a sufficiently distracted agent. MCP servers run in their own process and can enforce hard isolation.

**Skills do not centralize.** Every Skill file has to be installed on every machine that runs an agent. If your team has fifty engineers and a Skill needs updating, you have fifty manual updates. An MCP server is one deployment, and every agent reading from it gets the new behavior immediately.

**Skills are static prompts.** They cannot dynamically discover what is available, query a database for the current schema, or adapt to runtime state. They tell the agent what to do, not what is reachable.

The teams I have advised who tried to push everything through Skills had the same three symptoms. Their Skill files grew long and hard to maintain. Their agent started doing things it should not (because the Skill said "do X" but had no mechanism to enforce it). And every external integration was a custom bespoke job that nobody else on the team could reuse.

The fix is to use Skills for the thing Skills are good at (procedural knowledge, voice, conventions, calibrated judgment), use MCP for the thing MCP is good at (capability extension with auth and isolation), and use Tools for the thing Tools are good at (bounded deterministic local ops). Skills do not get abandoned. They get scoped.

## The hidden cost: tokens

There is a real tradeoff most write-ups gloss over. The three layers have different context costs.

**Tool calling**, at the schema layer, is cheap. A well-defined tool with a tight schema and a short description costs maybe 100 to 300 tokens. Ten of these is fine.

**MCP**, historically, has been expensive. One MCP server can expose 90 or more tools (the GitHub server is a famous case), and connecting three popular MCP servers on a 200K-token model could consume over 70% of the available context before the agent did anything. This was the legitimate basis for the "MCP is heavy" criticism that circulated through 2025.

Anthropic addressed this in January 2026 with MCP Tool Search, which dynamically loads tools on demand when they would consume more than 10% of context. The penalty has dropped substantially. The criticism is now mostly historical. But you should still audit how many tools a given MCP server exposes before connecting it.

**Skills** are the most context-efficient by design. The progressive disclosure pattern means only the descriptions are loaded at session start, and only the relevant Skill body loads when the agent decides it matches. You can have hundreds of Skills installed without a noticeable context hit. This is the design move that makes Skills genuinely useful for procedural knowledge at scale.

The token math has moved fast enough in the last year that any benchmarks older than three months are probably stale. Treat token efficiency as a tiebreaker, not a primary criterion.

## The architecture I would build in 2026

If I were starting a serious agent build today, here is what the stack looks like.

**The runtime.** Claude Code or Claude API with the Agent SDK. Both speak MCP natively, both load Skills natively, both expose the built-in tools that cover 80% of local operations.

**The MCP layer.** One server per external system. GitHub, Linear or Jira, Notion or your wiki, Slack, your primary database, the SaaS tools you actually use. Most of these have official or community-maintained servers already. Run them locally or in your infrastructure depending on your security posture.

**The Skill layer.** One Skill per repeatable workflow. The PR review skill, the bug fixer, the customer feedback theme extractor, the consultant-translator, the contract redline against your playbook. The Skills directory grows with use. By month six, you should have ten to twenty Skills covering the most common procedural work in your team.

**The Tool layer.** Built-in tools cover most of what you need. Custom tools are rare and tightly bounded: a parser for a specific file format, a calculator for a domain-specific number, a transformer for a specific input. If you find yourself writing a custom tool every week, your work probably belongs in an MCP server instead.

The split that works in practice: 70% of your extensibility energy goes into Skills (procedural know-how is the highest-return thing you can encode), 25% goes into MCP servers (one solid server per external system), 5% goes into custom Tools (only when nothing else fits).

If your distribution looks very different from that, it is worth asking why. Teams that go 100% on Skills are usually missing the external-systems layer. Teams that go 100% on MCP are usually missing the procedural-knowledge layer. Teams that go 100% on custom Tools are usually reinventing what MCP gives you for free.

## Closing

The three layers are not in competition. They cover different surface area. The teams that get this right are pragmatic: Skills for the playbooks, MCP for the connectors, Tools for the deterministic native ops. The teams that get it wrong are usually the ones being loud on one of these and quiet on the others.

The most common mistake in 2026 is not "they picked the wrong layer." It is "they only picked one." The right architecture spans all three deliberately, and the question to ask of any new agent capability is the same one in three forms: is this a procedure, an outlet, or a hand?

Get the question right and the layer falls out almost automatically.

---

*Marco Kotrotsos, specializing in practical AI implementation for organizations ready to close the gap between AI hype and AI value. With 30 years of IT experience now focused purely on AI deployment, he works hands-on with companies to turn AI potential into measurable business outcomes.*

*This article is published in [Autocomplete](https://medium.com/autocomplete-real-world-ai), a Medium publication about real-world AI for practitioners and decision-makers. We're always looking for writers. If you're building with AI and have something worth sharing, reach out.*

*My free Substack newsletter, also called Autocomplete, can be found here: https://acdigest.substack.com.*
