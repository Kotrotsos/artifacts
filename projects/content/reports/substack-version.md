# Skills, MCP, and Tool Calling: Why You Need All Three

*Skills are loud right now. Building agents on Skills alone gives you half an agent. Here is how I think about the three layers, and how the teams I have advised this year are actually wiring them up.*

![Three stacked layers, the bottom labeled tools, the middle labeled MCP, the top labeled Skills, rendered as a clean isometric stack of teal slabs with coral side icons](hero.png)

The most common architectural question I have answered in the last two months is some version of this: should I write a Skill for this, build an MCP server, or just use the agent's built-in tools? Half the time the question is framed as "Skill or MCP" with no awareness that tool calling is also a distinct mechanism. The other half assumes the three are interchangeable.

They are not. They solve different problems. And the teams I have watched try to push everything through Skills have all hit the same set of walls within about three weeks.

This is the article version of the answer I keep giving in client calls. Hit reply if you want me to look at a specific architecture decision your team is making.

**Three things to know up front:**

The three mechanisms are not competing. Tool calling is the agent's hands (bounded, deterministic local ops). MCP is its outlets to external systems (auth, state, multi-tenant access). Skills are its playbooks (procedural know-how, voice, conventions). Each one solves a problem the other two cannot.

The Skills hype is real but misleading. Skills cannot authenticate to your database. They cannot maintain state. They cannot expose one capability to every agent in your fleet at once. The teams treating Skills as the universal answer are quietly hitting walls a few weeks in.

The 2026 architecture that actually works is hybrid. Skills provide the procedure. MCP provides the connectors with auth, state, and isolation. Tools provide the deterministic native operations. The right question is not "Skill or MCP?" but "Skill for the how, MCP for the where, Tool for the what."

## Why the confusion exists

Three different things, all pitched the same way in the docs. "Extend your agent." "Give Claude new capabilities." "Plug in to your stack." Same marketing, entirely different mechanics.

It does not help that they shipped in the order they did. Tool calling came first, in mid-2023, when GPT-4 introduced function calling. MCP arrived in November 2024 from Anthropic. Agent Skills launched in October 2025, also from Anthropic. The newest one is the loudest one because it shipped most recently and has the most viral file format (a folder with a markdown file is great LinkedIn content).

So in the noise, teams who are just now arriving at "how do I customize my agent" hear about Skills first and try to push everything through Skills. The other two layers exist. Most of the writing about Skills does not bother explaining when not to use them.

This piece does both.

## What each layer actually is

![Three columns showing the anatomy of tool calling (a JSON-like rectangle), MCP server (a box with outlet and data arrows), and Agent Skill (a manila folder with SKILL.md), each with a one-line description of what it is](diagram-1-anatomy.png)

### Tool calling: the agent's hands

A tool is a function the model can decide to invoke. It has a name, a description, an input schema, and a returned result. The agent reads the description, decides the situation matches, calls the tool with arguments, gets a response, continues reasoning.

Tools are the foundation. Every other mechanism on this list is built on top of tool calling or wraps it. When Claude Code reads a file, it is calling the built-in `Read` tool. When it runs a shell command, that is the `Bash` tool. Tool calling has been the agent extensibility primitive since GPT-4 introduced function-calling, and it remains the lowest-level interface to "the agent can do this specific thing."

Tools are bounded. They run a defined function with defined inputs and return a defined output. They are local in the sense that they run in the agent's runtime, not on a remote server.

### MCP: the agent's outlets

Model Context Protocol is the protocol Anthropic published in November 2024 and open-sourced under Apache 2.0. The mental model is plumbing. MCP defines how a server (a program that exposes some capability) talks to a client (the AI application). The protocol is JSON-RPC 2.0 over either stdio (for local servers) or HTTP and SSE (for remote ones).

An MCP server exposes three kinds of things: tools (functions the client can call), resources (data the client can read), and prompts (templates). The agent on the other end gets a standardized way to discover what is available and how to call it.

The point of MCP is that the integration is built once and works for any client that speaks the protocol. Build a GitHub MCP server, and every agent that speaks MCP can use GitHub. Build a Notion MCP server, and you can wire it up to Claude Desktop, Claude Code, Cursor, Continue, and any future agent that adopts MCP.

MCP also handles three things that pure tool calling does not. Authentication and credentials live inside the server boundary. State can be maintained across calls. Multi-tenant isolation is built in.

### Agent Skills: the agent's playbooks

Anthropic shipped Agent Skills in October 2025. A Skill is a folder. Inside the folder is a markdown file called SKILL.md with YAML frontmatter (name and description) and a markdown body with instructions. Optionally, the folder includes reference files, scripts, and example outputs.

The Skill loads lazily. At session start, the agent reads only the metadata. The full content loads only when the agent decides the situation matches. This progressive disclosure is what lets you have hundreds of Skills installed without bloating context.

Skills are not capability. They are configured procedural knowledge. A Skill named `pr-review` does not give the agent the ability to read code or post comments on GitHub. It gives the agent a calibrated procedure for reviewing pull requests: what to look for, what to flag, how to format the feedback, what voice to use. The agent already had to be able to read files and post comments via Tools or MCP. The Skill tells it how to use those capabilities well for this specific task.

A clean mental model: a Skill teaches Claude what to do. A Tool gives Claude something to do it with. An MCP server gives Claude somewhere to do it.

## The decision tree

Three questions, in order.

![A decision tree showing three questions: does the agent already have this capability, does it need to reach an external system, does it need a new local capability, with arrows pointing to USE A SKILL, USE MCP, or USE A TOOL respectively](diagram-2-decision.png)

**Question 1: Does the agent already have this capability built in?**

If yes, you do not need a new capability layer. You need a procedural layer. Write a Skill. Example: Claude Code already has `Read`, `Write`, `Edit`, `Bash`, and `WebFetch` tools built in. If the task is "review my pull requests against the style guide," the agent has everything it needs already. The job is to encode the procedure. That is a Skill.

**Question 2: Does the agent need to reach an external system?**

If yes, reach for MCP. Example: the agent needs to read Jira tickets, query your production database, post to Slack, or look up customer records in your CRM. Each of those is an external system with its own authentication, its own schema, its own rate limits. The right answer is an MCP server that wraps that system and exposes it to the agent through the protocol.

**Question 3: Does the agent need a new bounded local capability?**

If yes, write a tool. Example: the agent needs to parse a specific file format you have, run a calculation, or perform a deterministic transformation that does not warrant a full external server. A custom tool, defined in the agent's tool list at runtime, is the right shape.

The hidden fourth question, which most teams skip: does this workflow need multiple of the above? It almost always does.

## A real workflow uses all three

Concrete example: "Fix a bug reported in Linear, in our production repo, with a tested PR."

![A diagram showing an agent in the center connected to three clusters: a bug-fixer Skill at the top, GitHub and Linear MCP servers to the right, and Read/Write/Bash tools at the bottom, with annotations showing how each layer contributes to the workflow](diagram-3-architecture.png)

You need three things. You need the procedure (read the issue, reproduce the bug, write the fix, verify with tests, open a PR with a proper description). You need access (the Linear issue, the repo, the PR system). You need execution (read files, run tests, write the fix).

If you build this with Skills alone, you cannot read the Linear ticket, because Skills cannot authenticate to Linear. You also cannot post a PR, because Skills cannot post to GitHub.

If you build this with MCP alone, the agent has access but no procedure. It can hit the GitHub and Linear APIs all day, but without a calibrated workflow it will write inconsistent PRs, miss test runs, and produce different bug fixes for the same root cause depending on which prompt triggers it.

If you build this with tools alone, you hand-roll an integration for every external system, and your tool list grows linearly with every new system the agent touches.

The right architecture uses all three:

- A `bug-fixer` Skill that defines the procedure: read, reproduce, fix, verify, open PR. It also encodes your conventions.
- A GitHub MCP server and a Linear MCP server that provide authenticated access to the relevant systems.
- The built-in Read, Write, Edit, and Bash tools that give the agent the local execution hands.

Skill for the how. MCP for the where. Tools for the what.

## Why "just Skills" is the most common mistake

Skills are loud right now for understandable reasons. They are the newest, they look easy because they are just markdown, and the file format makes great LinkedIn content. Teams that have only just discovered Skills tend to overcorrect and push everything through them.

Five specific things that Skills cannot do, which I see teams discover the hard way.

**Skills cannot authenticate.** A Skill is an instruction pack. It does not hold credentials, it cannot refresh tokens, it cannot make authenticated API calls on its own. The moment your workflow involves a real external system with real auth, you are out of Skills territory.

**Skills cannot maintain state.** A Skill is loaded into the agent's context each time it activates. There is no persistent process. If you need session state across calls, you need either a Tool that manages it locally or an MCP server that holds it.

**Skills cannot enforce isolation.** If you have multiple tenants, multiple environments, or multiple credentials, you cannot use a Skill as your security boundary. Skill instructions can be ignored by a sufficiently distracted agent. MCP servers run in their own process and can enforce hard isolation.

**Skills do not centralize.** Every Skill file has to be installed on every machine that runs an agent. If your team has fifty engineers and a Skill needs updating, you have fifty manual updates. An MCP server is one deployment, and every agent reading from it gets the new behavior immediately.

**Skills are static prompts.** They cannot dynamically discover what is available, query a database for the current schema, or adapt to runtime state.

The fix is to use Skills for the thing Skills are good at (procedural knowledge, voice, conventions, calibrated judgment), use MCP for the thing MCP is good at (capability extension with auth and isolation), and use Tools for the thing Tools are good at (bounded deterministic local ops). Skills do not get abandoned. They get scoped.

## The hidden cost: tokens

There is a real tradeoff most write-ups skip. The three layers have different context costs.

**Tool calling** is cheap at the schema layer. A well-defined tool with a tight schema costs 100 to 300 tokens. Ten of these is fine.

**MCP** has historically been expensive. One MCP server can expose 90 or more tools (the GitHub server is a famous case), and connecting three popular MCP servers on a 200K-token model could consume over 70% of the available context before the agent did anything. Anthropic addressed this in January 2026 with MCP Tool Search, which dynamically loads tools on demand when they would consume more than 10% of context. The penalty has dropped substantially.

**Skills** are the most context-efficient by design. Only descriptions are loaded at session start; only the relevant Skill body loads when activated. You can have hundreds of Skills installed without a noticeable context hit.

The token math has moved fast enough in the last year that any benchmarks older than three months are probably stale. Treat token efficiency as a tiebreaker, not a primary criterion.

## What I would build in 2026

If I were starting a serious agent build today, here is what the stack looks like.

**The runtime.** Claude Code or Claude API with the Agent SDK. Both speak MCP natively, both load Skills natively, both expose the built-in tools that cover 80% of local operations.

**The MCP layer.** One server per external system. GitHub, Linear or Jira, Notion or your wiki, Slack, your primary database, the SaaS tools you actually use. Most of these have official or community-maintained servers already.

**The Skill layer.** One Skill per repeatable workflow. The PR review skill, the bug fixer, the customer feedback theme extractor, the consultant-translator, the contract redline. By month six, you should have ten to twenty Skills covering the most common procedural work in your team.

**The Tool layer.** Built-in tools cover most of what you need. Custom tools are rare and tightly bounded. If you find yourself writing a custom tool every week, your work probably belongs in an MCP server instead.

The split that works in practice: 70% of your extensibility energy goes into Skills, 25% into MCP servers, 5% into custom Tools.

If your distribution looks very different, it is worth asking why. Teams that go 100% on Skills are usually missing the external-systems layer. Teams that go 100% on MCP are usually missing the procedural-knowledge layer. Teams that go 100% on custom Tools are usually reinventing what MCP gives you for free.

## What I would do if your team is just starting

Three concrete moves for the next thirty days, regardless of where you are.

First, audit your existing extensions. List every customization you have made to your agent. Tag each one as "instruction" (Skill territory), "external system" (MCP territory), or "bounded native op" (Tool territory). The audit alone surfaces most of the misallocation.

Second, install one MCP server this week. Pick the external system your agent reaches for most often (probably GitHub or your wiki). The community-maintained servers cover most of what you need. The hands-on familiarity will sharpen your judgment on when MCP is the right answer.

Third, look at your Skills directory and ask of each Skill: is this a procedure, or is this trying to be a capability? The Skills that are quietly trying to be capabilities are the ones that will break first under real use. Convert those to MCP-backed workflows.

Hit reply on this email if you want a closer look at your specific stack. I read everything.

## What is coming next

The next two newsletter issues will continue this thread:

- **Building your first MCP server in an afternoon.** A walkthrough from zero, with the auth pattern, the tool list, and the gotchas that catch most first-time builders.
- **The Skill audit.** A concrete pattern for cleaning up an existing Skills directory and migrating the misallocated ones to MCP.

**One open question for you, if you are willing to share in the comments:** what is the most miscast Skill in your current setup? The one that is trying to do something Skills were never meant to do? I am curious which patterns are showing up most across teams right now.

Until next week.

Marco

---

*Autocomplete is a free weekly newsletter on practical AI implementation. If you are not subscribed yet, the button above this paragraph is the easiest way to fix that. If you are already subscribed, forwarding this to one person who would benefit is the single best thing you can do for me.*
