# Automations to build once, ranked by effort vs payback

Each item is something you do by hand repeatedly that should be a hook, script, or config. Ranked by payback per unit of effort (best ratio first). Hours are realistic monthly estimates with the adversarial verifier's corrections applied where it ran. Almost all of these wire together tools you already own.

Reminder from your environment: in agent threads the shell cwd resets between Bash calls, and `cat` is aliased to pygmentize, so commit messages must be written with `printf` to a temp file. Build accordingly.

## Ranking

| Rank | Build | Effort | h/mo | Ratio | Type |
|---|---|:--:|:--:|:--:|---|
| 1 | Memento SessionStart recall + SessionEnd store hooks | S | 6-8 | very high | hook |
| 2 | agent-eval reviewer hook (stop hand-pasting the rubric) | S | 4-6 | very high | hook |
| 3 | Fire-and-notify runner for long jobs (kill poll loops) | S | 4 | high | script |
| 4 | STATUS.md generator on SessionEnd | S | 3-4 | high | hook |
| 5 | 3-project WIP limit + monthly graveyard prune | S | 3-5 | high | config + cron |
| 6 | 3-tier model router (Haiku/Sonnet/Opus) | M | 3-5 | medium | config |
| 7 | pre-commit + CI gate (lint, tsc, security, tests) | M | 4-6 | medium | template |
| 8 | shared-core monorepo (rag, doc-ingest, session-reader) | M | 4 | medium | package |
| 9 | Secrets-out-of-chat scan hook | S | risk | medium | hook |
| 10 | Datasets out of git (.dataignore + manifest + sync) | M | 1-2 | medium | convention |
| 11 | Linear auto-sync from git BLOCKER/DONE tags | M | 1-2 | low-med | script |
| 12 | `proj` jump + `findrep` shell helpers | S | 1-2 | low-med | shell |

---

## 1. Memento SessionStart recall + SessionEnd store hooks  (S, 6-8 h/mo, the backbone)

**Problem.** Your SessionStart hook only prints a reminder. Memento search is used in under 1 percent of sessions. Every project return pays a full context rebuild (4,032 Reads, repeated `/init`, re-pasted logins).

**Build.**
- Use `update-config` to add a SessionStart hook that calls `mcp__memento__memory_search(project + " context architecture decisions blockers", limit 3)`, injects the results, and echoes one line: branch, last decisions, pending work.
- Add a SessionEnd hook that calls `mcp__memento__memory_store` with a closure record (project, date, decisions, pending work, config/login state, friction points), tagged by project.
- Test on PERSONAL-Content, project7, Rechtzaak and watch the Read count and the "what was the login" questions drop.

**Wires.** Memento MCP (already used 78x), `update-config`, `context-save`/`context-restore` patterns. Build nothing new.

## 2. agent-eval reviewer hook  (S, 4-6 h/mo)

**Problem.** The GREEN/AMBER/RED reviewer block ran in 30 projects but is pasted by hand 300+ times. The eval skill itself was used only 19 times.

**Build.**
- `update-config` hook firing on `agent-eval:eval` completion (or on a Linear issue tagged `eval`/`review`).
- On fire, append the standard rubric (confidence grounded/partial/speculative, effort S/M/L, hours saved) to `eval/eval.html`, and if a Linear issue is in context, add it as a comment.

**Wires.** `agent-eval:eval`, Linear MCP (104x), `update-config`.

## 3. Fire-and-notify runner for long jobs  (S, 4 h/mo, verifier-corrected from 12)

**Problem.** You babysit scrapers and scans with `ScheduleWakeup(270s)` poll loops on Opus (PERSONAL-print) and manual "check if PID still running" prompts (reverse-engineer).

**Build.**
- Prefer the harness `run_in_background` so you are re-invoked on exit, no polling.
- For external state, a Haiku fetch-and-diff on the `schedule`/`loop` skill: store last state in Memento, escalate to Opus only on a detected change or completion.
- A `bgrun` helper that launches a job to a dated log dir, kills on timeout, and reports partial state.

**Wires.** `run_in_background`, `schedule`, `loop`, Memento. Route the checks to Haiku, not Opus.

## 4. STATUS.md generator on SessionEnd  (S, 3-4 h/mo)

**Problem.** Zero STATUS.md files across 38 projects, so resuming after an idle gap means a full re-read. Worst idle ratios: aside, Rechtzaak, montferland, Content.

**Build.**
- A `~/.claude/templates/STATUS.md.template` (Current Status, Last Session, Blockers, Next Steps).
- The SessionEnd hook (same one as item 1) populates it from Memento plus the last few messages.
- On SessionStart, if STATUS.md is over 3 days old, show it and offer "continue from here?".
- A `status-gen backfill <project>` CLI to seed the top idle projects.

**Wires.** Rides item 1's SessionEnd hook. This is `/session-wrap` from skills.md in hook form.

## 5. 3-project WIP limit + monthly graveyard prune  (S, 3-5 h/mo plus enjoyment)

**Problem.** 38 projects in 7 weeks, top 7 hold 71 percent of time, 55-60 percent of folders are graveyard. Reactive context-switching, dead weight in the registry.

**Build.**
- `.claude/work-in-progress.yaml` with `active: [A,B,C]` plus a queue. SessionStart warns when you open something off-list.
- A monthly cron (via `schedule`) tagging projects under 2 active hours and over 30 days idle as sunset, batch-archiving to `.archive/` with a final snapshot and a logged rationale.

**Wires.** `schedule`, `TaskUpdate`, `agentboard`/`taskboard`.

## 6. 3-tier model router  (M, 3-5 h/mo plus latency)

**Problem.** Opus is 77.9 percent of calls; Sonnet 1.6, Haiku 1.0. Search, extraction, triage, polling, and review subagents all run on Opus.

**Build.**
- `~/.claude/config/model-router.json`: Haiku for under-500-token extraction and search, Sonnet for under-500-LOC review and 2-to-5-source synthesis, Opus for the rest.
- Start with the highest-rep case: route the browse skill's WebSearch/WebFetch extraction to Haiku, pass the distilled result to Opus.
- Teach `security-review`, `redteam`, `codex` the Sonnet tier. A/B test quality before locking thresholds.

**Wires.** `model` skill (advisory today), `browse` (the integration point at 450 uses).

## 7. pre-commit + CI gate  (M, 4-6 h/mo)

**Problem.** Manual security-review loops; an XSS reached review in mailplus; `tsc` ran once in a TS monorepo.

**Build.**
- `husky` + `lint-staged`: `tsc --noEmit`, ESLint with `eslint-plugin-security`, Prettier on commit.
- `.github/workflows/ci.yml`: lint, test, build, `npm audit`/Snyk; gate merge with branch protection.
- Optionally fire `/security-review` on PR via the agent-eval hook from item 2.
- Ship as a template so every scaffold gets it for free.

**Wires.** `security-review`, `review`, `code-review`, `codex`. Pairs with item 2.

## 8. shared-core monorepo  (M, 4 h/mo, verifier-corrected from 10.5)

**Problem.** archer rebuilt 7x, RAG 5-6x, doc-ingest 4x, session-readers 6x, reverse-engineer base 3x.

**Build.**
- `~/projects/shared-core/` with `rag-core`, `doc-ingest`, `archer-cli`, `reverse-engineer-base`, `session-reader`.
- Publish versioned packages (npm and PyPI).
- Starter templates that import the latest shared-core. Update `/new` to suggest clone-from-template for these categories.

**Wires.** `/new` (424x) as the entry point. Pairs with the scaffold library.

## 9. Secrets-out-of-chat scan hook  (S, risk reduction)

**Problem.** A live Google OAuth secret (subradar), the Anthropic key, and a user password `ppryme123` (askwell) sit in your transcripts.

**Build.**
- A pre-submit or pre-commit hook running gitleaks or a regex for `GOCSPX-`, `sk-ant-`, `AIza`, and `pass(word)?[:=]`, warning before the secret persists.
- A habit baked into `/ship-it`: reference `.env` keys by name, have the agent read them.
- Rotate the three credentials already in the logs.

**Wires.** `update-config`, gitleaks. Cheap insurance.

## 10. Datasets out of git  (M, 1-2 h/mo plus repo health)

**Problem.** ~32 GB committed in elastofirm-profiler; no `.dataignore`.

**Build.**
- `~/.claude/.dataignore-template` (`*.parquet`, `*.pkl`, `*.bin`, `*.onnx`, `data/raw/*`, `cache/*`).
- `git rm --cached` the offenders, move data to Azure Blob or HF Hub, write a `DATASETS.toml` manifest, and a `data-sync pull/push` CLI wired into SessionStart.

**Wires.** Azure Blob / HuggingFace MCP, the `archive-artifact.sh` pattern.

## 11. Linear auto-sync from git tags  (M, 1-2 h/mo)

**Problem.** 104 manual `save_issue` calls, concentrated in NXTPHASE.

**Build.**
- A `linear-log` step (SessionEnd or `/linear-log`) that parses the last commit and diff for `BLOCKER:`/`DONE:` tags, creates or updates Linear issues, and returns the URLs.

**Wires.** Linear MCP (confirmed wired), `update-config` for credentials, the SessionEnd hook.

## 12. `proj` jump + `findrep` shell helpers  (S, 1-2 h/mo)

**Problem.** 3,026 `cd` calls (largest single shell command); Read-to-Grep ratio 8:1.

**Build.**
- `proj <name>`: cd to a project root, source per-project env, with completion. For agent sessions the real win is the SessionStart hook restoring last project (cwd resets in-thread).
- `findrep <pattern> <path>`: find plus grep plus head, to preview before Read. Add a CLAUDE.md nudge to prefer Grep-first.

**Wires.** Shell config plus the SessionStart hook from item 1.

---

## Suggested build order

Do items 1, 2, and 4 together in one sitting: they share the SessionStart/SessionEnd hook plumbing and unlock the backbone everything else rides on. Then 3 and 5 (both small, both high payback). Then the medium items as they land in your path. See `quick-wins.md` for the exact first week.
