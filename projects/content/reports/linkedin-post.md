# LinkedIn Post: Spec-Driven Dev Belongs on the Board

Spec-driven development is one of the operational shifts that actually compounds in 2026. Writing a clear spec before writing code, letting the agent interview you to refine it, generating tickets from the spec.

The practice is right.

Putting the spec in a spec.md file in your repo is wrong.

A spec has five things a flat markdown file cannot do well:

LIFECYCLE. Draft, in review, approved, in progress, blocked, done. The state matters.

OWNERSHIP. Named human responsible, notified on changes, has authority to approve.

STATUS. Live operational status visible to the whole team this morning.

DEPENDENCIES. What blocks what. Real graph, queryable, updatable.

HISTORY. Not git blame. Audit log of who decided what and why.

Linear has all five as first-class concepts. So does Jira, Notion, Asana. They have spent ten to fifteen years getting good at exactly this.

The reason teams put specs in markdown files anyway is understandable. Agents read markdown natively. The fastest workaround for an integration problem was to put the spec where the agent could read it.

That workaround is no longer necessary. Linear has an MCP server. Jira has one. Every major project tool either has one or will within the quarter.

The right architecture is the round-trip:

Linear holds the spec, source of truth. The agent exports a markdown snapshot when it needs to work. The agent does its work against the file. Updates flow back to Linear via MCP.

The file is for travel. The board is for living.

Stop putting specs in markdown only. Stop hand-syncing files to tickets. Stop reinventing the project management tool you already pay for.

Use markdown for what it is good at (portable, agent-readable, version-controllable). Use Linear for what it is good at (lifecycle, ownership, status, dependencies, history).

Full breakdown with the round-trip pattern and a 30-day playbook:

[link to Medium article]

How many markdown files in your current repo are trying to be a system of record for something that has a real lifecycle?
