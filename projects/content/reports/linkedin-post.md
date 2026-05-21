# LinkedIn Post: Workspace Authored, Files Generated

Three weeks ago I argued spec.md was the wrong container for your specs.

Three engineering teams worth of follow-up conversations later, the sharper version is one sentence:

Workspace is authored. Files are generated.

Spec.md is not the problem. Treating it as the source of truth is.

The pattern that holds up is three tiers, not one:

TIER 1: source of truth. Lives in Notion or Linear. Continuously evolving. Read by humans and by agents via MCP.

TIER 2: working snapshot. Lives in .spec-cache/ (gitignored). Pulled from the workspace at session start. Used by the agent during the session. Regenerated next session. Never committed.

TIER 3: frozen snapshot. Lives in docs/specs/ (committed). Frozen at PR open. Evidence of what spec the work was built against when it landed.

The three tiers answer three different questions:
- What does the team currently believe this feature should be?
- What is the agent looking at right now?
- What spec was true when this code shipped?

A single spec.md at the root of your repo cannot be all three things at once. That is what made the original setup drift.

The workflow:
1. Session start: pull from workspace via MCP into .spec-cache/
2. Agent reads, executes, may refine the cache
3. Session end: sync changes back to workspace, discard cache
4. PR open: freeze a snapshot to docs/specs/, commit alongside the code

Notion's May 13 Developer Platform launch made this easier. Ivan Zhao: "Use your Notion database as a sheer canvas to power both your workflows and your agents." First-class integrations with Claude Code, Cursor, Codex, Decagon.

If you remember one line: workspace authored, files generated.

Full breakdown with the round-trip pattern and the three-tier architecture:

[link to Medium article]

How many markdown files in your current repo are still being treated as a source of truth for something with a real lifecycle?
