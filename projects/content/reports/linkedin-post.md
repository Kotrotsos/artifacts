# LinkedIn Post: The Claude Code Harness

Anthropic dropped a substantial guide on May 14: "How Claude Code works in large codebases."

One sentence in it carries the whole argument:

The harness matters as much as the model.

Performance at scale is determined by six layers around the model, plus one delegation capability:

1. CLAUDE.md (context every session)
2. Hooks (event-triggered scripts)
3. Skills (on-demand expertise)
4. Plugins (distribute what works)
5. LSP integrations (symbol-level navigation)
6. MCP servers (external tools and data)
+ Subagents (delegation, available anytime)

The model is one of seven things.

Teams that fixate on model benchmarks are tuning the wrong knob. The benchmark differences between current frontier models are real but small. The differences in harness quality between teams are massive.

Build order matters.

CLAUDE.md first, because nothing else has anywhere to live until it exists. Hooks next, because hooks make the harness self-improving. Skills, then plugins, then LSP, then MCP. Subagents whenever you need them.

The biggest configuration pathology I see in client teams: loading everything into CLAUDE.md instead of building skills. CLAUDE.md becomes the dumping ground, performance degrades, skills never get built. The fix is mechanical: any time you find yourself writing a chunk of CLAUDE.md that starts with "when doing X, do Y," ask whether X is a recurring task type. If yes, it is a skill. Move it.

The second-biggest mistake: building MCP integrations before the basics are working. Teams get excited about MCP, spend two weeks wiring up Jira and Datadog, then wonder why Claude is not producing quality output. The model has access to every tool in the company and no idea what to do with any of it.

Three configuration patterns travel across every successful deployment:

- Make the codebase legible (layered CLAUDE.md, scoped commands, LSP)
- Keep the harness current (audit every 3 to 6 months, after every major model release)
- Assign ownership from day one (a named DRI, a plugin marketplace, eventually an agent manager role)

If you remember one line from this post: the model is one layer of seven.

Full annotated breakdown of each layer, the build order, and what to do this week:

[link to Medium article]

What does your CLAUDE.md root file look like? Mine had 1,200 lines a month ago. Now it has 60.
