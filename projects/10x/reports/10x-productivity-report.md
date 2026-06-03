# The 10x Report

**How to get an order of magnitude more out of the way you already work**

Prepared 2026-06-03. Scope: 879 Claude Code session transcripts (641 MB, distilled to 41 prompt-only digests) and 352 project folders across `PERSONAL`, `projects`, and `NXTPHASE`. Method and limits are at the end.

---

## The one thing to take away

You have already built the machine. You drive it by hand.

The single strongest finding, verified on disk, is that most of the right automation already exists in your setup and simply is not wired to fire. Your agent-eval harness has produced an `eval.html` in 30 projects, yet only 2 projects carry a `.claude/settings.json` and the global settings file contains zero references to the eval runner. So the GREEN/AMBER/RED reviewer block, the single most repeated piece of text in your entire corpus, gets pasted by hand 300-plus times instead of firing automatically. Your Memento hooks run on every session but only print "search Memento" and "did you store?"; they never call recall or store, so your memory is a write-mostly journal with no compounding payoff. There is not a single `STATUS.md` anywhere, which is why "what was the url again" and "where were we" recur in every cluster.

The 10x is therefore not new capability. It is conversion. Wire the hooks you own, commit the scripts you keep retyping, turn your standing pipelines into skills you invoke instead of re-explain, and add four or five load-bearing assets so velocity stops producing seven copies of the same app. Almost everything below is a settings merge, a committed script, or a one-time extraction, not a research project.

---

## Who the data says you are

A high-velocity solo builder and consultant running five tracks at once over roughly 15 months:

- **AI and agent tooling, CLIs, skills, MCP** (~28% of folders): craft-agents-oss, open-cowork, memento, agentboard, symphony, whatsapp-mcp, and dozens more.
- **Claude and LLM reverse-engineering and introspection** (~12%): reverse-engineer (1,500 files), claude-reverse, claude-reverse2, codex-reveng, agentspy, claudespy, portspy. A genuinely distinctive specialty.
- **Client and consulting delivery** (~16%): NXTPHASE plus named companies (KLAFS, Maxeda, Keesing, Triferto, NHA, ANP, Azure POCs). This is the paid work.
- **Content and publishing engine** (~12%): Content (2,800+ files), gloss, Content-Marketing, kyssbook-kdp, hyperframes video.
- **Web and SaaS product attempts** (~16%): remembered, specbridge, openscreen, breakaway, timeweaver, logicgrid-ai, stitchboy, and many more Next.js / React / Tauri builds.
- **Data, scraping, OSINT, document and image profiling** (~10%) and a **creative / generative / retro-computing** hobby track (~6%).

You are not primarily a coder. You are a product owner and orchestrator who delegates execution to Claude and audits hard, with screenshots, an eval gate, and explicit "did you actually check it" verification.

---

## The numbers

- **352** project folders across three roots (PERSONAL 122, projects 184, NXTPHASE 46). Roughly **55 to 60% are graveyard**; only about **120 to 140** are distinct real attempts.
- The eval grader is the **single most repeated literal text** in the corpus: ~140 of 182 prompts in `aside`, 49 of 49 `Rechtzaak-go` sessions, ~45+27 in `project7`, 30+ in `montferland`, 9 in `projectultra`, 5 byte-identical in `elastofirm`. **300+ verbatim pastes.**
- **30 projects** contain `eval.html` (the harness ran) but only **2** have a `.claude/settings.json`, and the global settings has **zero** eval-run references. The trigger is manual in ~28 projects.
- Memento hooks fire every session but are **reminder-only**: they print instructions and never call `memory_recall` or `memory_store`.
- **`STATUS.md` count across all roots: 0.** The project state-card pattern does not exist.
- Only **2 MCP servers** wired (whatsapp, ria). Notion, Linear, GitLab, Gmail, Drive, ms365 are all available and disconnected.
- **~32 GB** of duplicated binary data: `elastofirm-profiler` 20G + `rubbermoulds-profiler-old` 5.8G + `rubbermoulds-profiler-main copy` 5.8G (near-identical 77k-file tree) + an images folder at 489M.
- Four major reinvention clusters: **archer rebuilt 7x**, document-ingest 4x, RAG stack 5-6x, `~/.claude` readers 6x. Plus randy 5x, dot2dot 4x, baps 4x.
- The `print` scraper project: **1,539 prompts** dominated by one hand-typed poll ("check log tail, launch next category, schedule another wakeup in N seconds") retyped all night.
- The `Content` repo: **408 prompts**, with the source-to-trifecta loop re-specified by hand 15+ times.
- You own **151 skills**, a full gstack SDLC, a ~40-agent pentest swarm, and skillify / agent-eval / benchmark. New hooks and skills are realistic for you, not aspirational.

---

## The seven patterns that cost you the most

### 1. Built it, but drive it by hand
You own the automation but never wire the trigger. 30 `eval.html` vs 2 `settings.json` vs 0 eval-run references. Memento hooks print reminders instead of executing recall and store. Remote control and push notifications are switched on, but no cron, monitor, or background loop is configured. **Implication:** the highest-ROI work is wiring, not building. Every loop you already automated by hand is one settings merge from being free.

### 2. Standing pipelines re-specified verbatim instead of invoked
The trifecta is requested 15+ times as "the usual trifecta with all the anonymization and deep-detection and images." The new-SaaS arc is retyped prompt by prompt (TS / Next / Drizzle / Clerk / Railway). The ultracode kickoff, including "make maildmz@gmail.com pro," is reused every time. **Implication:** each pipeline collapsed into one parameterized skill removes a 5 to 20 prompt re-specification per run, and kills the recurring bolt-on afterthoughts (Substack hero, author footer, white-background images).

### 3. Manual orchestration of correctly chosen parallelism
The `print` project's 1,539 prompts are almost entirely the same poll, retyped. `ecoscrape` fans out ~37 subagents hand-batched with "create a team and go," each re-deriving the 9-column rubric. You ask "are you still running, where is the result" and retro-estimate token cost for 16k companies after the fact. **Implication:** the strategy is right, the packaging is missing. A resumable job harness plus a fixed-schema enrich-and-score skill turns an all-night babysitting session into one launch and one ping.

### 4. No durable per-project state, so every session cold-starts
Zero `STATUS.md` files. "What was the url again for montferland," "how did I get to admin again," "what is the url" (Rechtzaak, six times), "where were we" across 47 `aside` sessions and 39 `subradar` sessions, two `project7` context overflows. Tuning thresholds live only in chat history. **Implication:** a deploy-written `STATUS.md` surfaced at session start plus active Memento recall kills 5 to 15 minutes of re-grounding on every cold or compacted session across 80+ multi-session projects.

### 5. House rules enforced by memory, not by the harness
"Looks offbrand, no emojis" after a reskin. Mid-stream global rules bolted on (em-dash, light-mode, white background). Rechtzaak Dutch-locale defects re-reported ("August instead of augustus," "date can never be in the future"). Your own CLAUDE.md even documents the heredoc-git-commit ANSI footgun. **Implication:** a PostToolUse brand and locale linter plus golden-file fixtures convert recurring correction rounds into deterministic gates. The model stops being asked to remember, and the harness guarantees it.

### 6. Distrust and recheck, audited by hand
You do not trust unverified "done" claims. "Ouch, do better please, do check your work." "What do you mean verified live, did you check it?" The hallucinated red-team system prompt ending in your fix: "double check the finding using a subagent before writing to file." 28 numbered screenshots in `project7`, ~24 in `aside`. **Implication:** auto visual-QA chained after deploy (browse + emil-design-eng gating any "verified live" claim) and a verification subagent gate in /redteam convert your manual audit into a harness-run gate.

### 7. Folder sprawl with one structural root cause: a fresh repo per attempt
~32 GB duplicate datasets. archer rebuilt 7x with byte-identical READMEs. Exact filesystem copies (`ai-insights` vs `ai insights`, `galaga` vs `galaga 2`, the rubbermoulds copy). ~25 empty stubs. **Implication:** a small set of load-bearing assets (one SaaS scaffold, four internal packages, a pre-create guard) absorbs whole future classes of restarts. This is the only durable fix, and it compounds: each asset prevents the next five reinventions.

---

## The 10x backlog, ranked by leverage

Leverage = impact times how often it bites, divided by effort. Impact "10x" is reserved for genuine step-changes.

### 1. Install the agent-eval hooks user-wide, impact 10x, effort S
Run the agent-eval plugin's installer at user scope so the PreToolUse capture and Stop run hooks land in the global `~/.claude/settings.json` and every project auto-grades the last turn with zero pasted prompt. Add self-healing guards (`mkdir -p .claude/eval-state`, `touch eval/eval.html`) to the top of `eval-run.sh` so it can never exit 127 on a fresh repo.
- **Kills:** the most repeated text in the corpus (300+ pastes) plus the cross-session breakage you reported.
- **Saves:** thousands of round-trips, daily.
- **First step:** run `bash /Users/marcokotrotsos/.claude/plugins/cache/nxtphaseai/agent-eval/0.3.0/install.sh --user`, confirm a Stop hook referencing `eval-run.sh` appears, then finish one task in `project7` and watch a card auto-append.

### 2. Turn the Memento hooks into a real recall and store loop, backed by STATUS.md, impact high, effort M
Edit `session-start-memento.sh` to actually call `memory_recall` for the cwd project namespace and inject the top hits plus the project `STATUS.md`. Edit `stop-memento.sh` to store a structured card (what shipped, key decisions, file paths, live url, tuning thresholds). Have the deploy step write a `STATUS.md` per project.
- **Kills:** status anxiety and re-orientation in every cluster.
- **Saves:** 2 to 4 hours a week.
- **First step:** make `session-start-memento.sh` run a recall and cat `STATUS.md` instead of only printing the instruction.

### 3. Commit a per-project verify-deploy script chained into /ship, impact high, effort M
Extract the hand-pasted Railway smoke test (cookie-jar login, GET /documenten, version grep, migration check) into a committed `scripts/verify-deploy.sh` that reads the expected version from `git describe`, parses the full semver with a correct regex so the "grep stops at the first hyphen" bug disappears, polls until live, and prints one-line PASS/FAIL plus the url. Chain it onto /ship and /land-and-deploy.
- **Kills:** Rechtzaak's six near-identical verify pastes and a recurring bug you annotated by hand instead of fixing.
- **Saves:** 5 to 8 minutes per deploy, 20+ deploys per active build week.
- **First step:** create the script from the prompt-157 incantation, fix the regex to `v[0-9]+\.[0-9]+\.[0-9]+(-[a-z0-9.-]+)?`, add a "Deploy & verify" section to a new Rechtzaak CLAUDE.md.

### 4. Build a resumable enrich-and-score harness plus a self-driving stage runner, impact 10x, effort L
An enrich-and-score skill that takes a list, auto-chunks into parallel subagents, runs website-find then multi-page scrape then deterministic email harvest then the fixed 9-column rubric as a schema in the skill, backed by a SQLite job table (pending / scraped / scored / done) with a status command. Plus a generic state-machine stage-runner launched once in the background that PushNotifies only on completion or error.
- **Kills:** the 1,539-prompt poll loop and the hand-batched ecoscrape swarm.
- **Saves:** an entire all-night session collapses to one launch and one ping; unlocks the 16k-company scale.
- **First step:** write the SQLite job-table schema and a `status` command first, then wrap `build_xlsx.py` and the men/women/kids advance logic into one stage runner.

### 5. Build the /trifecta content-pipeline skill, impact high, effort M
One orchestrator: input a source, output Medium + Substack + LinkedIn drafts into `/drafts/<slug>/`, auto-run Humanizer and deep-detect, generate hero plus a default Substack hero plus diagrams via nanobanana with your image defaults baked in as a fixed preset (pure white background, infographic style not watercolour, per-channel sizing, no duplicate heroes), and append the author footer. You never restate "the usual trifecta" or separately ask for a Substack hero again.
- **Kills:** the 15+ trifecta re-specifications and the image-redo loop in your 408-prompt Content repo.
- **Saves:** 20 to 40 minutes per article trio, multiple trios a week.
- **First step:** run /skillify on a recent complete trifecta transcript to capture the exact stage sequence, then author the SKILL.md with the image preset stored in assets.

### 6. Wire Notion, Linear, and GitLab MCP plus two reconciliation subagents, impact high, effort M
Add the connectors to `mcpServers` (only whatsapp and ria are wired today). Build a Notion-to-spec reconciliation subagent that diffs board items against implemented features and deploy tags, and a /mr GitLab subagent wrapping `glab` for the recurring Montferland flow (list open MRs, open by number, summarize and reply to comments, resolve conflicts, push, return merge url).
- **Kills:** the hand-driven "go through the notion board" and 10+ manual glab actions, including from-scratch glab onboarding.
- **Saves:** 15 to 30 minutes per client review session, several a week.
- **First step:** add the Notion, Linear, and gitlab connectors to settings, then write `notion-spec-reconcile.md`.

### 7. One starter-kit SaaS scaffold (a /new-saas skill), impact high, effort M
A cookiecutter encoding your standard stack: E2E TypeScript, Next.js, Drizzle, Clerk (auth + billing), Railway config, GitHub repo on `kotrotsos` with main + feature-branch convention, marketing and content folders, light-mode-default UI, a pre-filled project CLAUDE.md, the Memento namespace, and an auto-seed of `maildmz@gmail.com` as the pro account. It scaffolds inside a chosen workspace and registers in a projects index so it cannot quietly become folder 353.
- **Kills:** the greenfield bootstrap loop retyped every few weeks.
- **Saves:** 2 to 4 hours of setup per new product, and it feeds STATUS.md, the verify script, and the eval hook into every new project for free.
- **First step:** extract project7 / AskWell's scaffolding into a template, save as `PERSONAL/starter-saas`, wire a /new-saas skill.

### 8. Auto visual-QA gate after deploy plus a verification gate in /redteam, impact high, effort M
Extend /ship so after deploy it drives the changed page with browse, screenshots at your breakpoints, self-checks against emil-design-eng heuristics, reports defects before you eyeball, and refuses "verified live" without an attached capture. Add a rule to /redteam so any claimed secret is re-probed by an independent verification subagent and only written to the report with the confirming curl attached.
- **Kills:** the screenshot ping-pong (28 images in project7) and false-positive findings.
- **Saves:** 5 to 15 round-trips per UI change, 10 to 20 minutes per red-team engagement.
- **First step:** add a "visual gate" step to /ship; edit the redteam skill so report-writing must call a verification subagent first.

### 9. Move ~32 GB of duplicate datasets out of git, impact high, effort S
Hash-compare the two 5.8 GB rubbermoulds trees, delete the confirmed copy, `git rm --cached` the DWG/DXF/PNG dirs from the canonical profiler repo, push the datasets to one Azure Blob container (you already use Azure), add a tiny `fetch-data.sh`. Keep only the matcher code versioned.
- **Kills:** the largest single disk and git weight in the corpus.
- **Saves:** ~26 GB immediately, plus minutes per future git op.
- **First step:** hash-compare `rubbermoulds-profiler-old` and `rubbermoulds-profiler-main copy`, delete the copy, then extract datasets.

### 10. Extract four internal packages from your reinvention clusters, impact high, effort L
Publish four small versioned packages via your existing pip_release skill so the next instance is `pip install`, not a new repo: archer-core (Rich TUI + Anthropic tool loop), doc-ingest (pluggable OCR/vision to markdown-with-frontmatter), rag-core (one ingest-embed-query API with a config per corpus), claude-session-reader (parse `~/.claude` sessions to an API). Each existing folder becomes a thin consumer or branch.
- **Kills:** the four largest reinvention clusters (archer 7x, doc-ingest 4x, RAG 5-6x, session readers 6x).
- **Saves:** each new variant drops from 1 to 2 days to about an hour; ends graveyard growth at the source.
- **First step:** start with archer-core: diff archer-code vs archer-research, extract the shared core to `src/archer_core` with a pyproject, pip_release it, make the others import it.

### 11. Enforce house style and domain invariants with a linter hook plus golden fixtures, impact med, effort M
A Write/Edit linter that deterministically rejects em-dashes in prose, emojis, serif fonts, dark-default CSS, and guards the heredoc-git ANSI footgun, and auto-runs `archive-artifact.sh` after any artifact write. Pair with golden-file tests: Rechtzaak Dutch-legal invariants (month names always Dutch, datum perslijst earliest and never future, deadlines) and a subradar subscription-detection regression suite locking the BUNQ/KNAB/PayPal CSVs.
- **Kills:** the recurring brand and locale correction rounds.
- **Saves:** 2 to 5 minutes per affected diff plus avoided rework.
- **First step:** write `~/.claude/hooks/brand-lint.sh` and register it as a PreToolUse Write|Edit hook.

### 12. A one-time triage sweep plus a pre-create guard, impact med, effort S
Hash folder trees and fold `<name> copy` / `<name> 2` / `name with space` into the canonical repo as a branch, delete the ~25 empty stubs, archive the ~40 one-shots into a single `archive/` folder. Bake a pre-create guard into `mk new` that checks a projects index and warns "archer already exists 7x, branch it?" Result: from 352 folders to ~120 distinct attempts, and it stays there.
- **Kills:** the navigation tax and the structural cause of sprawl.
- **Saves:** a one-time half day, then near-zero; this is what makes the package consolidation durable.
- **First step:** run a content-hash sweep, delete the verified-empty stubs and the rubbermoulds copy first (zero risk), add "never create copy/2/with-space, branch instead" to global CLAUDE.md.

---

## Quick wins (each under an hour, do them today)

1. Install agent-eval at user scope. One command kills the most repeated text in the corpus.
2. Add the self-healing guard to `eval-run.sh` so the eval never dies on a fresh project.
3. Wire Notion, Linear, and gitlab into `mcpServers` so the next board or MR action does not start from scratch.
4. Make `session-start-memento.sh` actually call `memory_recall` and cat `STATUS.md` instead of printing a reminder.
5. Write `brand-lint.sh` (em-dash, emoji, serif, dark-default) and register it as a PreToolUse hook.
6. Hash-compare and delete `rubbermoulds-profiler-main copy` for an immediate 5.8 GB reclaim at zero risk.
7. Delete the ~25 empty stub folders to cut navigation noise.
8. Add a "Deploy & verify" section to a new `Rechtzaak/CLAUDE.md` pointing at the verify script.

---

## The 30 / 60 / 90 day roadmap

**Day 30, wire what you own.** Install agent-eval user-wide, upgrade Memento to recall and store, add the brand and locale linter. Commit `verify-deploy.sh` into Rechtzaak chained to /ship and emit STATUS.md on success; backfill STATUS.md into montferland and project7 by hand once. Wire Notion + Linear + GitLab MCP and the two reconciliation subagents. Delete the rubbermoulds copy and the empty stubs. Stand up the first /trifecta from a /skillify capture.

**Day 60, package the standing pipelines.** Ship the resumable enrich-and-score harness and stage runner, proven by re-running the next ecoscrape batch as config not rebuild. Finalize /trifecta and add the visual-QA gate to /ship plus the verification gate to /redteam. Build the starter-saas scaffold and /new-saas so every new project inherits the eval hook, STATUS.md, verify script, and your conventions for free. Add golden fixtures for Rechtzaak and subradar.

**Day 90, fix the structure.** Extract and publish archer-core, then doc-ingest, rag-core, claude-session-reader, converting the duplicate folders into thin consumers. Complete the dataset-to-object-store migration. Run the triage sweep and add the pre-create guard, taking 352 folders to ~120 and keeping it there. Set up scheduled remote agents (you have remote control and push on, but zero crons): a weekly red-team delta for recurring targets, a Polymarket-to-article snapshot-and-diff, and uptime monitors on your live apps that page you via push.

---

## Stop doing these

- Stop pasting the GREEN/AMBER/RED reviewer block by hand. It runs in 30 projects but triggers in 2.
- Stop hand-typing the Railway smoke test per deploy, and stop annotating the grep bug instead of fixing it.
- Stop re-specifying the trifecta, the new-SaaS stack, and the ultracode mandate every time. Templatize them.
- Stop driving long scrapes by retyping the same poll all night. Use a state-machine runner with completion-only notifications.
- Stop hand-batching subagent swarms and re-deriving the rubric per batch. Put the rubric in a skill schema.
- Stop relying on the model to remember house rules. Enforce them with a linter.
- Stop pasting live secrets into the chat stream (Clerk `pk_live_`/`sk_live_`, Google OAuth secret, X `ct0`/`auth_token`, passwords). Use a secrets-sync skill.
- Stop creating `<name> copy` / `<name> 2` / `name with space` folders for the same idea. Branch or install the package.
- Stop committing multi-gigabyte datasets into git. Code in git, datasets in object storage.
- Stop treating Memento as write-only. The recall loop your own CLAUDE.md mandates is not running.

---

## Worth resurrecting or harvesting

Substantial builds that went cold but have real plumbing. Flagged on build-completeness and product framing, not on whether the market still matters.

- **stitchboy**: 61 TS files, full app + server, Dockerfile, railway.toml, requirements, mockups, a security_todo, even a video. Mid-build.
- **linkforge**: full PRD, spec, designs, Drizzle, Next.js, railway.json, file-upload work in the last commit.
- **remembered**: 160-file Next.js app with a real schema, 63 commits. A memory/recall product that aligns with your Memento focus.
- **breakaway / breakloop**: habit-breaking app with a pitch deck and an Expo/RN build, 22 commits ending mid-redesign.
- **inkwell**: 108-file Next.js writing/editor app, on-brand for your content focus.
- **timeweaver**: 152-file scheduling visualization app with heavy real data backing it.
- **voiced**: a complete Outlook text-to-speech add-in with deploy plumbing.
- **gemcut**: Tauri desktop app (Rust + TSX), a real cross-platform scaffold.
- **Claude_Prophet**: a Go trading engine with real config data, self-contained.
- **complia / qual / PE**: substantial B2B product UIs (compliance, qualitative analysis) under the nxtphaseai org.

---

## Method and honest limits

This is a prompt-and-filesystem analysis run as a 16-agent workflow: 879 transcripts distilled to 41 prompt-only digests, 352 folders inventoried, and your existing tooling catalogued, then analyzed across session clusters, folder lifecycle and reinvention, and tooling gaps, recommended through three independent lenses, and curated adversarially.

What the analysis can and cannot see:

- It is not an execution trace. Prompt counts conflate text you typed with content the harness auto-fired. The eval block in particular inflates apparent counts, because some of the "300+ pastes" are the agent-eval hook surfacing as prompts rather than literal manual copy-paste. The fix (wire the trigger) is identical either way, but read the per-paste time-saved figures as upper bounds.
- Time-saved estimates are order-of-magnitude judgments from loop frequency, meant for ranking, not forecasting.
- The load-bearing claims were verified directly on disk: only whatsapp and ria MCP wired, zero eval-run references, reminder-only Memento hooks, 30 eval.html vs 2 settings.json, zero STATUS.md, ~32 GB profiler datasets with the 5.8 GB copy, Rechtzaak with no CLAUDE.md or verify script, 352 folders. Individual per-prompt citations (specific Rechtzaak prompt numbers, the 1,539 print prompts, the 408 Content prompts) come from the lens analyses and are taken as reported, not re-checked against raw logs.
- The graveyard classification cannot see commercial reality. A cold folder may be a finished client deliverable, a parked bet, or live elsewhere. Resurrection candidates are flagged on completeness, not on whether the market still matters.
- Reinvention clusters assume the folders share intent. archer and the profiler were spot-confirmed on disk; doc-ingest, RAG, and session readers rely on byte-identical-README evidence without re-hashing every tree. Confirmed false positives (project / project7 / projectultra are distinct products) are excluded.
- Code quality, test coverage, security posture, and actual API spend were not assessed. The tooling gaps are inferred from missing skills, not observed failures.
- Some recommendations assume your stack stays Railway + Next/Drizzle/Clerk and your content stack stays Humanizer/deep-detect/nanobanana. A migration (the Rechtzaak Bitbucket + Kubernetes move) would shorten the shelf life of the verify-deploy and new-saas scaffolds, though the patterns transfer.
