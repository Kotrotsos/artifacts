# Top 5 quick wins for this week

Chosen for the best payback-to-effort ratio and genuinely doable in one week. The first two share the same hook plumbing, so do them together. Everything here wires tools you already own.

---

## 1. Wire the memory backbone (SessionStart recall + SessionEnd store + STATUS.md)

**Why it is first.** Your `CLAUDE.md` mandates Memento but the hook only prints a reminder. So memory search runs in under 1 percent of sessions and you rebuild context by hand every return (4,032 Reads, repeated `/init`, re-pasted logins). This is the backbone the next four wins ride on.

**Do this.**
1. `update-config`: add a SessionStart hook calling `mcp__memento__memory_search(project + " context decisions blockers", limit 3)`, inject the top 3, print one summary line.
2. Add a SessionEnd hook calling `mcp__memento__memory_store` with a closure record (decisions, pending work, config state), tagged by project, that also writes `STATUS.md`.
3. Test on PERSONAL-Content, project7, and Rechtzaak.

**Payback.** 6 to 8 h/mo and far less re-explaining yourself. Effort S.

---

## 2. Install the agent-eval reviewer hook

**Why.** The GREEN/AMBER/RED reviewer block ran in 30 projects but is pasted by hand 300+ times; the eval skill itself was used only 19. Your build sessions are full of repeated "Review this change for security vulnerabilities" and "impartial code reviewer" prompts.

**Do this.**
1. `update-config`: hook that fires on `agent-eval:eval` completion (or a Linear issue tagged `eval`).
2. On fire, append the standard rubric to `eval/eval.html` and, if a Linear issue is in context, as a comment.

**Payback.** 4 to 6 h/mo, and the rubric stops living in your clipboard. Effort S.

---

## 3. Make design-first the default opening move

**Why.** This is your single biggest real friction. subradar and mailplus built first then reworked ("this looks offbrand, redo," "make it native looking, it looks bad now"). Figma MCP and your design skills sat unused. askwell designed first and barely reworked.

**Do this (no build, just a default).**
1. Add one line to your global `CLAUDE.md`: "For any new UI, generate mockups and lock Tailwind tokens before writing components."
2. Next app you start: open with Figma MCP or `design-html` from reference screenshots, run `design-review`, then build.
3. Optionally save it as `/design-first` (see `skills.md`).

**Payback.** 5 to 7 h/mo of rework avoided. Effort S, mostly behavioral.

---

## 4. Stop burning Opus on waiting

Two small moves, both about not paying premium reasoning for low-value work.

**4a. Fire-and-notify instead of poll loops.** PERSONAL-print woke the agent hundreds of times on a `ScheduleWakeup(270s)` loop to tail a scraper. Use `run_in_background` (the harness re-invokes you on exit) or a Haiku fetch-and-diff that only escalates on change. Wrap recon scans in a background runner that reports on completion.

**4b. Route browse extraction to Haiku.** The browse skill's WebSearch/WebFetch extraction (2,607 calls) runs on Opus. Route the extract step to Haiku, pass only the distilled result to Opus. This is the first slice of the model router and the highest-rep one.

**Payback.** ~4 h/mo plus a noticeable latency drop on every research and QA loop. Effort S.

---

## 5. Get secrets out of your transcripts (and rotate the leaked ones)

**Why.** A live Google OAuth client secret (subradar), your Anthropic key, and a real user password (redacted here as `ppry...`, verbatim in your logs) for askwell are sitting in saved transcripts on disk.

**Do this.**
1. Rotate those three credentials now.
2. Add a pre-submit or pre-commit secret-scan (gitleaks or a regex for `GOCSPX-`, `sk-ant-`, `AIza`, `pass[:=]`) that warns before a secret persists.
3. New habit: reference `.env` keys by name and let the agent read them, never paste the value.

**Payback.** Low hours, real risk removed. Effort S.

---

## After this week

The next tier is the 3-project WIP limit plus a monthly graveyard prune (focus and enjoyment), the `/launch-kit` and `/ship-it` skills (the rituals you repeat per product), and the shared-core monorepo. See `automations.md` for the build order and `skills.md` for the skill specs.
