# 10x Report: an evidence-based plan to multiply your output and keep it enjoyable

Prepared 2026-06-03. Author: productivity and systems analysis over your own Claude Code record.

## The one-line thesis

You have already built the machine. You drive it by hand. The leverage is not new capability, it is wiring, defaults, and focus. Almost every fix below is "connect two things you already own," not "go build something new."

---

## 1. What I actually analyzed (so you can trust or challenge this)

This is mined from the real record, not from assumptions.

- **880 Claude Code session transcripts** in `~/.claude/projects/`, 481 MB of JSONL, spanning **2026-04-15 to 2026-06-03** (about 7 weeks), across **38 projects**.
- I wrote a deterministic extractor (`10x/_data/extract.py`) that distilled every session into a per-project digest: timing, tool histograms, normalized shell-command frequencies, slash-command and skill usage, a session-by-session timeline, and every verbatim message you typed plus every correction you made. Digests are in `10x/_data/`.
- I then ran a multi-agent workflow (`10x/_data/workflow.js`): 14 per-project analysts plus 6 cross-cutting miners produced 66 raw findings, a synthesis pass merged them to 23, and an adversarial verifier re-checked each one against the evidence and deflated the optimistic estimates. Raw verified output is in `10x/_data/_findings.json`.
- I read the highest-signal digests myself to sanity-check the agents.

### Honest caveats about the numbers

I am flagging these up front because they change how you should read the rest.

- **"Active hours" is the real time number, not "wall hours."** Total active time (events with gaps under 5 minutes) is about **168 hours**. The "8,452 wall hours" figure is meaningless: it counts idle time and overlapping background sessions. Where I cite hours, I mean active.
- **The friction percentage is a noisy proxy.** It counts user messages matching correction or frustration patterns. It is inflated in build-heavy projects because your own review subagents send prompts containing words like "broken," "wrong," and "fix" (the "Review this change for security vulnerabilities" and "impartial code reviewer" prompts). So `projectultra` at 43 percent and `mailplus` at 41 percent are mostly false positives. I lead with verbatim quotes instead, which are trustworthy.
- **`PERSONAL-print` is not 1,539 hand-typed prompts.** It is a shoeprint-matching app for first responders, and most of those "messages" are automated `task-notification` events from a `ScheduleWakeup(270s)` loop babysitting a Zalando scraper. The poll-loop finding is real, the mechanism is "agent self-polling," not "human typing 1,539 times."
- **The workflow had failures.** Many per-project analyst agents (run as read-only `Explore` agents) finished without emitting structured output, so 9 of the spawned agents produced the usable findings. The 6 cross-cutting miners, which carry most of the value, all succeeded, and I covered the failed project slices by reading the digests myself.

---

## 2. The portfolio: where the 168 hours went

Time is concentrated. The **top 7 projects hold about 71 percent of active time**; 31 projects have under 2 hours each.

| Project | What it is | Active h | Real signal |
|---|---|---:|---|
| PERSONAL-Content | Blog (gloss.run), Substack, LinkedIn, books, the content factory | 30.9 | Editing-heavy, multi-step pipeline run by hand |
| projects-project7 ("askwell") | AI-interview/questionnaire SaaS, end-to-end TS, Railway/Postgres | 18.2 | 605 `cd` calls; deploy plumbing + launch-kit done by hand |
| PERSONAL-print | Shoeprint-match app for first responders + Zalando scraper | 17.6 | Poll-loop babysitting via ScheduleWakeup |
| montferland-virtuele-assistent | Client: municipality virtual assistant | 16.9 | Long idle gaps, context rebuilt on return |
| NXTPHASE-Rechtzaak (+ go) | Legal-case document processing (Dutch) | 16.6 | 104 Linear `save_issue`, manual issue logging |
| projects-subradar | Subscription Radar, local-first Electron app | 10.9 | Design-after-build rework; OAuth secret pasted in chat |
| NXTPHASE-pool | "pool" project | 9.2 | Low friction, fairly smooth |
| projectultra ("Prism") + sound | Local-first generative brand studio + sibling | 8.6 | "continue, computer crashed"; review-subagent churn |

The rest (`aside`, `elastofirm-profiler`, `kyssbook-kdp`, `reverse-engineer`, `ecoscrape`, `mailplus`, and ~30 thin folders) fill the tail.

What you build, in plain terms: **client work** (legal and municipal document processing), a **content factory** (blog, Substack, books), a stream of **local-first apps** spun up via `/ultracode` dynamic workflows (subradar, Prism, mailplus, askwell), and **security/recon** work. You are prolific. The cost of being prolific is the same setup ritual paid over and over.

---

## 3. Findings, grouped by what is actually costing you

Each finding cites the record. Confidence is labelled. Where the adversarial verifier corrected an estimate, I use the corrected number and say so. Estimates without a verifier pass are marked "est."

### Theme A: You rebuild context every session because memory is mandated but not wired

**Evidence.** Your global `CLAUDE.md` mandates Memento, but the SessionStart hook only prints a reminder, it never calls `memory_search` or `memory_store`. Memento search was used 78 times in 880 sessions (under 1 percent of tool calls), read-only. Meanwhile `Read` is your top-after-Bash tool at 4,032 calls, much of it re-establishing context (557 Reads in subradar, 650 in project7). You re-run `/init` and "analyze this codebase and create a CLAUDE.md" on projects that already have one, and you re-ask for state you already had (in subradar you re-paste the Google OAuth client id and secret to reconnect). There are **zero STATUS.md files** across 38 projects.

**Root cause.** The memory layer exists and is required, but nothing makes it automatic, so every return to a project pays a full context-rebuild.

**Fix.** Replace the reminder-only SessionStart hook with one that actually recalls (search Memento for the project, inject the top 3, print a one-line state summary), and add a SessionEnd hook that stores a closure record (decisions, pending work, config/login state, blockers). Add a STATUS.md convention generated by that same SessionEnd hook. This is the backbone the next three findings hang off.

**Impact.** 6 to 8 h/mo (est.), grounded mechanism. Effort S. This is the single highest-leverage wiring job because it compounds with everything else.

### Theme B: You design after you build, then pay for the rework

**Evidence (verbatim, the trustworthy signal).**
- subradar: the CLAUDE.md literally says "implement it. We will be creating the design later on." Then later: "this looks offbrand, Redo the design, no emoji's please" and "ok please redesign the layout to look more like this."
- mailplus: "look at the site and make it native looking, it looks bad now. look at how tiktok looks, make it look like that," after the build was already complete.
- Figma MCP (`use_figma`, `generate_figma_design`, Code Connect) is available and was never invoked in either project, despite you owning `design-review`, `design-html`, `design-shotgun`, and `high-end-visual-design` skills.

**Root cause.** Visual design is deferred until after implementation, so the whole QA cycle becomes a rework loop validating generic UI against a reference you had in your head the whole time.

**Fix.** Make design-first the default opening move for any front-end project: generate mockups from reference screenshots (Figma MCP or `design-html`), lock tokens into the Tailwind config, then build. Gate the first build on a `design-review` pass. You already do this on askwell ("currently creating the design in claude design, wait until the design is done") and askwell had far less visual rework. Copy that habit to every app.

**Impact.** 5 to 7 h/mo (est.), grounded by quotes. Effort M. This is the largest real friction in your app work.

### Theme C: You run your review and eval machinery by hand, hundreds of times

**Evidence.** Across subradar, project7, projectultra, and mailplus, the session timelines are saturated with near-identical "Review this change for security vulnerabilities" and "You are an impartial code reviewer, output a SINGLE JSON object" prompts. The prior on-disk audit found the agent-eval GREEN/AMBER/RED reviewer block ran in 30 projects but was pasted by hand 300+ times because the hook is not installed user-wide. The `agent-eval:eval` skill itself was only invoked 19 times. `tsc` appears once in mailplus despite a fully TypeScript monorepo, and a real XSS in `ReaderPanel.tsx` slipped through to manual review.

**Root cause.** The review and eval skills exist and produce good output, but nothing fires them automatically, so the rubric and the security pass are re-invoked manually each session, and basic gates (lint, typecheck) are skipped.

**Fix.** Two coupled moves. (1) Install the agent-eval reviewer as a user-wide hook via `update-config`, so the rubric attaches automatically. (2) Add `husky` + `lint-staged` (tsc, ESLint with eslint-plugin-security, Prettier) on commit and a CI gate on push, optionally firing `/security-review` on PR. The manual security loop disappears and bugs get caught before they reach you.

**Impact.** 4 to 6 h/mo (est.), grounded (300+ pastes). Effort S for the hook, M for CI.

### Theme D: Opus runs everything, including work a faster model should do

**Evidence.** Opus is 77.9 percent of model calls (opus-4-7 at 27,842 plus opus-4-8 at 11,732 = 39,574). Sonnet is 807 (1.6 percent), Haiku 485 (1.0 percent). You have a binary Haiku-or-Opus mental model with Sonnet ignored. Opus runs the browse skill's web-extraction (WebSearch + WebFetch = 2,607 calls), the poll loops, and the review subagents, all latency-sensitive or low-complexity work.

**Root cause.** No routing. Tasks default to Opus regardless of complexity, which costs you latency and attention more than money.

**Fix.** A 3-tier default: Haiku for search, extraction, triage, and polling; Sonnet for mid-size code review and 2-to-5-source synthesis; Opus for real reasoning and synthesis. Start with the highest-rep case: route the browse skill's extraction step to Haiku and pass only the distilled result to Opus.

**Impact.** 3 to 5 h/mo of reclaimed waiting plus a large time-to-first-result win (est.), grounded distribution. Effort M.

### Theme E: You poll long jobs by hand instead of firing and forgetting

**Evidence.** PERSONAL-print: a `ScheduleWakeup(270s)` loop woke the agent hundreds of times to tail a scraper log (39 ScheduleWakeup, 26 Monitor calls). reverse-engineer: "Check again whether the full TCP sweep (PID 15477) on 84.31.80.122 has finished. If still running after this poll, kill it."

**Root cause.** Deterministic wait-check-refresh work runs as synchronous Opus calls rather than a background job that notifies on completion.

**Fix.** Use `run_in_background` (the harness re-invokes you on exit) or the `schedule`/`loop` skill with a Haiku fetch-and-diff that only escalates on a detected change. Wrap recon scans in a background runner that logs to a dated dir, kills on timeout, and reports the partial state automatically.

**Impact.** Verifier-corrected to 4 h/mo (down from a claimed 12), partial. Effort S.

### Theme F: Secrets keep landing in the chat

**Evidence (I verified this directly).** subradar: you pasted a live Google OAuth client id and secret (`GOCSPX-...`) into the conversation. project7/askwell: "Antrhpic key is in .env and merge it," and a real user password seeded inline (redacted here as `ppry...`; it is in your logs verbatim). Three distinct projects, real credentials, in transcripts that persist on disk.

**Root cause.** When wiring auth or seeding users, the fastest path is to paste the secret. There is no convention that keeps it out of the transcript.

**Fix.** A small habit plus a guard: never paste a secret value, always reference `.env` keys by name and have the agent read them itself; add a pre-submit or pre-commit secret-scan (gitleaks or a regex hook) that warns on `GOCSPX-`, `sk-ant-`, `AIza`, and password-like assignments. Rotate the three credentials above, they are in your logs.

**Impact.** Low hours, high risk reduction (est.), grounded (3 instances). Effort S. Listed because it is cheap and the downside is real.

### Theme G: Every new product repeats the same two rituals

**Evidence.**
- **Launch kit.** On askwell: "create a content folder to create media, blurbs, taglines, positioning statement, pitch," then "create a quick and clean pitch deck," "create a demo using hyperframes and remotion," "do some research as to viability, check ycombinator and producthunt, calculate the business model." The same sequence appears around your other launches.
- **Deploy plumbing.** Also askwell: "can you connect github yourself?", "sorry wrong button, create the github skill please," "you can get the token from .env and then RAILWAY," "wire up the postgresql database." GitHub connect plus Railway plus Postgres plus env wiring, by hand, per project.

**Root cause.** No reusable kit for the non-code parts of shipping a product, so naming, positioning, pitch, demo, viability, and deploy plumbing are recreated each time.

**Fix.** Two skills. A `/launch-kit` that composes `office-hours` (premise check) plus your content skills (`gloss-brand`, `substack-editor`, `nanobanana`, `hyperframes`) to produce naming options, positioning, taglines, a pitch deck, a demo, and a viability/business-model note into a `business/` folder. A `/ship-it` (or extend `/new`) that does the GitHub-connect plus Railway plus Postgres plus env plumbing in one go. You explicitly asked for the GitHub skill in the logs.

**Impact.** Launch kit 2 h/mo, deploy skill 2 to 3 h/mo (est.), grounded by quotes. Effort M each.

### Theme H: Sprawl and reinvention drain time and momentum

**Evidence.** 38 logged projects in 7 weeks, 100+ folders in `PERSONAL/`, and the prior on-disk audit found 55 to 60 percent of 352 folders are graveyard and that you rebuilt archer 7x, RAG 5-6x, doc-ingest 4x, session-readers 6x, and the reverse-engineer base 3x (you have `claude-reverse`, `claude-reverse2`, `codex-reveng` as separate folders). `cd` is your single most common shell command at 3,026 calls (605 in project7 alone). `elastofirm-profiler` has roughly 32 GB of datasets committed to git.

**Root cause.** No WIP limit, no sunsetting, no shared-core package, no STATUS.md. So you context-switch reactively, rebuild the same primitives, and carry dead weight in git.

**Fix.** Three controls. (1) A 3-project WIP limit in `.claude/work-in-progress.yaml` with a SessionStart warning when you open something off-list, plus a monthly graveyard prune of projects under 2 active hours and over 30 days idle. (2) A `shared-core` monorepo publishing `rag-core`, `doc-ingest`, `archer-cli`, `session-reader` so the primitives are imported, not rebuilt. (3) A `.dataignore` convention plus a `DATASETS.toml` manifest so datasets live in Blob/HF, not git.

**Impact.** WIP limit 3 to 5 h/mo plus enjoyment (partial); shared-core verifier-corrected to 4.2 h/mo (from 10.5), partial; datasets out of git 1 to 2 h/mo plus repo health, grounded. Effort S, M, M.

### Theme I: Smaller, real, lower-payback items

These are grounded but modest. Build them when they are in your path, not as a project.

- **Content pipeline `/edit-chain`** (Theme: skill). You run Humanizer (37x) and deep-detect (14x) as separate manual steps. Chain them with a fact-check pass and a per-step diff. ~2 to 3 h/mo, grounded.
- **Linear auto-sync.** 104 manual `save_issue` calls, concentrated in NXTPHASE. Parse git for `BLOCKER:`/`DONE:` tags and sync on SessionEnd. ~1 to 2 h/mo, grounded.
- **`proj` jump function.** 3,026 `cd` calls. A `proj <name>` shell function helps interactive shells (note: in-agent cwd resets between bash calls, so the bigger win is the SessionStart hook restoring last project). Verifier-corrected to 1 h/mo from 3.2, partial.
- **Grep-first over whole-file Reads.** Read-to-Grep ratio is 8:1 (4,032 to 499). A `findrep` helper plus a CLAUDE.md nudge. ~1 h/mo, partial.
- **Discovery spec-gate.** Open-ended "find the gap and build me something" prompts loop without converging (projects-project produced "It's empty"). Score top 3 by fit, then `/codex` a one-page spec before code. Verifier-corrected to 3.5 h/mo, partial.
- **Pre-implementation validation with `/codex` and `/redteam`.** You use redteam (213x) only for security, never to enumerate algorithm edge cases. subradar's detection false positives ("this is an obvious subscription, why is this flagged") came from no pre-impl validation. Verifier-corrected to 2 h/mo, partial.
- **Justfile per project.** Verifier deflated this to 0.1 h/mo and was right: the wall-to-active gap is driven by friction and context loss, not by build-command syntax. Skip unless you want consistency for its own sake.

---

## 4. The ranked 10x table

Ranked by realistic monthly payback, with the verifier's corrections applied. "Grounded" means the record directly supports it; "partial" means the pattern is real but the hours are estimated; "est." means I estimated the hours without a verifier pass.

| # | Change | Effort | Realistic h/mo | Confidence | Quick win |
|---|---|:--:|:--:|---|:--:|
| 1 | Wire Memento recall+store hooks (+ STATUS.md) | S | 6-8 (est.) | grounded mechanism | yes |
| 2 | Design-first default for all front-end work | M | 5-7 (est.) | grounded (quotes) | no |
| 3 | Agent-eval hook + pre-commit/CI review gate | S/M | 4-6 (est.) | grounded | yes (hook) |
| 4 | 3-tier model router (Haiku/Sonnet/Opus) | M | 3-5 (est.) | grounded | no |
| 5 | 3-project WIP limit + monthly graveyard prune | S | 3-5 | partial | yes |
| 6 | Fire-and-notify instead of poll loops | S | 4 | partial (corrected) | yes |
| 7 | shared-core monorepo (rag/doc-ingest/session-reader) | M | 4 | partial (corrected) | no |
| 8 | STATUS.md + session-wrap on SessionEnd | S | 3-4 | partial | yes |
| 9 | `/launch-kit` + `/ship-it` deploy skills | M | 4-5 | grounded (quotes) | no |
| 10 | `/edit-chain` content pipeline | M | 2-3 | grounded | no |
| 11 | Discovery spec-gate before building | M | 3.5 | partial (corrected) | no |
| 12 | Datasets out of git (.dataignore + manifest) | M | 1-2 | grounded | no |
| 13 | Pre-impl validation via /codex + /redteam | S | 2 | partial (corrected) | yes |
| 14 | Linear auto-sync from git tags | M | 1-2 | grounded | no |
| 15 | Secrets-out-of-chat habit + scan hook | S | risk, not hours | grounded | yes |
| 16 | `proj` jump + Grep-first helpers | S | 1-2 | partial | yes |

Concentrated total of the top 8: roughly **30 to 45 hours per month** of reclaimable active time, against about 100 hours per month of active Claude work. The honest read on "10x": you do not get 10x on everything. You get a genuine 10x on the specific high-repetition chores (poll loops go from hours to zero, the reviewer block goes from 300 pastes to zero, context rebuild goes from minutes-per-session to one hook), and you get a throughput multiplier on the work that matters from design-first plus a WIP limit. The compounding effect, every fix riding the Memento/SessionEnd backbone, is where the "machine you already built" finally runs itself.

---

## 5. What already works (do not touch)

- Your `/ultracode` dynamic-workflow habit for building apps is strong; the issue is design timing and review wiring around it, not the approach.
- `browse` (450 uses) as a QA and dogfooding loop is excellent and ahead of most developers.
- The content pipeline produces real output (books, Substack, gloss). It needs chaining, not redesign.
- You already use Memento and Linear via MCP; the gap is automation, not adoption.

---

## 6. Where the record is thin (so I am not padding)

- I could not measure per-command timing, so every "hours saved" is an estimate built from frequency times an assumed unit cost. Treat the ranking order as more reliable than the absolute hours.
- The friction percentage is unreliable for build projects, as explained in the caveats. I relied on quotes there.
- Client projects (montferland, Rechtzaak) are substantial but their digests are dominated by domain work and Linear logging; beyond "log issues automatically" and "wire memory," I did not find a generalizable productivity defect there, which is itself a good sign.
- The thin tail (30+ one-session projects) is mostly genuine exploration. The finding there is the WIP limit and the prune, not anything per-project.

See `10x/skills.md`, `10x/automations.md`, and `10x/quick-wins.md` for the actionable breakdowns.
