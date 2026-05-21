# The Spec Lesson, V2: Workspace Is Authored, Files Are Generated

*Three weeks ago I argued spec.md was the wrong container for your specs. Three engineering teams worth of follow-up conversations later, here is the sharper version. Spec.md is not the problem. Treating it as the source of truth is.*

![A workspace board at the top of the image with two file shapes below it, one with a dashed outline labeled working snapshot, one with a solid outline labeled frozen snapshot, all in translucent teal on a pure white background, with the caption workspace is authored, files are generated](hero.png)

The original piece argued for moving the spec to Linear. The argument landed with the teams that had already hit the drift wall. It made the teams that had not yet hit the wall defensive.

The defensive response was always some version of "the file works fine, the agent reads it, why are you trying to take it away?" That objection is fair. The original article underweighted it.

The honest answer, which took me another three weeks of working with engineering teams to get to, is that the file is not the problem. The file is fine. What does not work is the file being the authored, evolving, source-of-truth artifact. The fix is not to delete the file. The fix is to make sure the file got generated, not typed.

This newsletter is the operational version of that argument. The phrase to remember is one sentence. Workspace is authored. Files are generated. The rest is what that means in practice.

Hit reply if your team has been hitting the drift wall too. I want to hear how you have been handling it.

**Three things to know up front:**

Spec-driven development is right. The mistake is not the file format. The mistake is putting the authored, evolving, source-of-truth spec into a flat markdown file in your repo. The file drifts, the board drifts, nobody knows which is authoritative, and your team is silently running a third reconciliation system in Slack.

The operational rule that holds up across the teams I have advised this quarter is one sentence: workspace is authored, files are generated. The spec lives in Notion (or Linear). The repo holds two kinds of generated snapshot, never an authored file.

The two snapshots have different homes and different mutability. A gitignored working snapshot in `.spec-cache/` that the agent reads during a session, regenerated every time. A committed PR-attached snapshot in `docs/specs/` that freezes the spec at the moment work shipped. Both are generated. Neither competes with the workspace for "is it the truth."

## The thing that actually broke last time

Three engineering teams I worked with in March and April adopted spec-driven development the same way. spec.md in the repo, agent reads the file, work happens, file gets updated. All three loved it for the first three weeks. All three hit the same wall in week six.

The wall was always: the file says one thing, the Linear ticket says another, three engineers are operating off three different states, the team is silently running a reconciliation system in Slack. Nobody decided this should happen. It happened anyway, because the file and the ticket were both being treated as the truth, and they drifted.

When you ask the team where this drift came from, the honest answer is: nobody knew which artifact was authoritative. The file was authoritative for the agent. The Linear ticket was authoritative for status. The Slack thread was authoritative for the why. Three sources, no precedence rule, predictable result.

The original article's response was: move the spec to Linear, kill the file. The defensive response to that argument was: "but the agent needs to read markdown." Both are right. The synthesis is the title of this piece.

## Workspace is authored

Specs are authored in a workspace. Notion, Linear, Jira, Asana, Coda, whatever your team already uses. The choice between them matters less than people pretend. The point is that the authored spec lives in a tool built for things that have lifecycle, ownership, status, dependencies, and history.

The case for Notion specifically got a lot stronger on May 13, 2026, when Notion launched their Developer Platform. CEO Ivan Zhao's framing was sharp: "Use your Notion database as a sheer canvas to power both your workflows and your agents." Pitch: "Any data, any tool, any agent." Notion now ships first-class integrations with Claude Code, Cursor, Codex, and Decagon, plus Workers (cloud-sandboxed code execution inside Notion) and Database Sync.

That announcement is a vendor explicitly competing for the spec-source-of-truth slot. It is the first time a workspace tool has made the agent-readability story directly part of the product positioning. Linear has had MCP for a while. Notion now has MCP plus the developer platform plus the explicit agent-targeted pitch.

What changes when you move the authored spec to a workspace:

- The spec has a status. Draft, in review, approved, in progress, done. Updates the team automatically.
- The spec has an owner. A named human responsible, notified on changes, with authority to approve.
- The spec has dependencies. This one blocks that one. The graph is visible and updatable.
- The spec has a real audit history. Not git blame. The record of who proposed what change, when, and why.
- The spec connects to the rest of your work surface. Customer issues, bugs, design docs, in the same tool.

None of these are theoretical. They are the things a flat markdown file is bad at, and they are the things that caused the drift in the original three teams.

## Files are generated

The agent still needs markdown. That has not changed. What changes is where the markdown comes from.

![A vertical stack of three rounded cards showing the three tiers of spec artifacts: source of truth at the top in Notion or Linear, working snapshot in the middle in gitignored cache with a dashed outline, frozen snapshot at the bottom in docs slash specs committed](diagram-1-three-tiers.png)

The right pattern is three tiers, not one. The tiers have different homes and different mutability rules.

**Tier 1: source of truth.** Authored in Notion or Linear. Continuously evolving. Read by humans (and by agents through the MCP server).

**Tier 2: working snapshot.** Lives in `.spec-cache/spec.md` in the repo. Gitignored. Pulled from the workspace at session start via MCP. Used by the agent during the session. Regenerated next session. Never committed.

**Tier 3: frozen snapshot.** Lives in `docs/specs/<feature>-shipped.md`. Committed alongside the PR. Frozen at PR open and never updated. Evidence of what spec the work was built against when it landed.

The three tiers answer three different questions. The source of truth answers "what does the team currently believe this feature should be." The working snapshot answers "what is the agent looking at right now." The frozen snapshot answers "what spec was true when this code shipped, six months from now when someone is debugging."

A single file at the root of your repo cannot be all three things at once. That is what made the original setup drift.

## The actual workflow

The flow that ties the three tiers together is concrete enough to write down.

![A horizontal flow diagram with five labeled stages: workspace as source of truth, pull via MCP into a gitignored cache file, agent reads and executes, sync changes back to the workspace, freeze a snapshot into docs slash specs for PR evidence](diagram-2-workflow.png)

**Session start.** Pull the current spec from Notion or Linear via the MCP server. Write it to `.spec-cache/spec.md` (gitignored). The cache directory exists for the session and gets discarded after.

**During the session.** The agent reads from the cache file. The agent may also write to the cache file as the work surfaces new requirements or refinements. This is the agent's working medium. Behaves exactly like the spec.md you are used to, with one important difference: nobody else on the team is editing this file. It is yours for this session.

**Session end.** Diff the cache against the workspace. Sync the meaningful changes back. New acceptance criteria, refined requirements, surfaced edge cases, new sub-tasks. The workspace updates. The team sees the changes in their normal Notion or Linear workflow. The cache gets discarded.

**At PR open.** Generate a frozen snapshot of the spec as it stood when the work was finalized. Commit it to `docs/specs/<feature>-shipped.md`. This snapshot is the evidence in the PR review. It is the answer to "what spec did the agent build against."

That is the whole workflow. Five stages, two directions of flow, no ambiguity about which artifact is authoritative.

The reason this works where the original spec.md pattern broke: there is exactly one source of truth. The cache cannot drift because it gets regenerated. The frozen snapshot cannot drift because it is frozen. The team only edits one thing. The agent only reads one thing per session.

## The Marmelab response (steel-manning the dissent)

The strongest published dissent against spec-driven development is Marmelab's "Waterfall strikes back" piece. The argument is that big design up front fails because software development is non-deterministic. Specs go stale. Reality outruns the document.

The workspace-authored, files-generated pattern is not Waterfall. The workspace evolves continuously. Every session that surfaces a new requirement updates the workspace. Every PR review that catches an edge case updates the workspace. The team is iterating on the spec the whole time the work is happening.

What is Waterfall is putting a 4,000-word spec.md in your repo and then refusing to touch it for two months because "the spec is locked." That is the failure mode Marmelab is rightly objecting to. The two-snapshot pattern explicitly avoids it.

Lightweight specs in a workspace, iterated continuously, snapshotted for evidence: that is not Waterfall. That is what agile actually looks like when the team includes an AI agent that needs a written reference.

## What stops, what stays, what starts

![Three columns showing what to stop including human-edited spec.md at root and arguing which is the truth, what to keep including markdown as agent input format and version control for evidence, and what to start including workspace as source of truth and gitignored working cache](diagram-3-stop-keep-start.png)

**Stop.**

- Human-edited spec.md at the root of your repo. If a human typed it directly and committed it, it is in the wrong tier.
- Committing the working spec the agent uses during a session. That file should be ephemeral, not version-controlled.
- Hand-syncing files to tickets. The manual sync layer was a workaround that no longer needs to exist.
- Arguing which is the truth in Slack. If you find yourself in that conversation, you have not done the cleanup yet.

**Keep.**

- Markdown as the agent's input format. The agent reads markdown best.
- Version control for evidence. The frozen PR snapshot still lives in git.
- Spec-driven discipline. Writing things down before building them is correct.
- Lightweight written specs. Short specs in the workspace are better than long ones nobody updates.

**Start.**

- Workspace as source of truth. Notion or Linear.
- Gitignored working cache. `.spec-cache/` in the repo, never committed.
- Frozen PR snapshots in `docs/specs/`. Committed evidence at landing time.
- Generate, never author. If a file got typed by a human into git, it is in the wrong tier.

## What I would do this week

Three concrete moves for any team running the older pattern.

First, run the inventory. List every markdown file in your repo that contains spec-shaped content. For each, ask: is this also tracked in Notion or Linear? If yes, decide which is authoritative. If the answer feels uncomfortable, that is the answer.

Second, install the MCP server. Notion MCP is hosted and works with Claude Code, Cursor, VS Code, ChatGPT, and the major coding agents. Linear MCP is mature. Pick whichever workspace your team uses and wire it up.

Third, pick one feature your team is about to ship and run it through the new pattern as a pilot. Workspace authored, cache generated, PR snapshot frozen. Run it end to end. The team will know within two weeks whether the architecture is better.

Hit reply if you want me to look at your specific setup. I read everything.

## What is coming next

The next two newsletter issues will continue this thread:

- **The .spec-cache skill.** A walkthrough of the actual session-start and session-end scripts that implement the round-trip. About 40 lines, works with Claude Code or any Coding agent that speaks MCP.
- **Notion vs Linear for spec destination.** A practitioner-grade decision tree. Some teams will be better served by Notion (long-form, multi-stakeholder), some by Linear (lifecycle-first, lighter spec content). The deciding question is shape of work, not vendor preference.

**One open question for you, if you are willing to share in the comments:** how many markdown files in your current repo are still being treated as a source of truth for something with a real lifecycle? Be honest. Mine had four. (After last issue, mine had two. Better, not done.)

Until next week.

Marco

---

*Autocomplete is a free weekly newsletter on practical AI implementation. If you are not subscribed yet, the button above this paragraph is the easiest way to fix that. If you are already subscribed, forwarding this to one engineering lead who would benefit is the single best thing you can do for me.*
