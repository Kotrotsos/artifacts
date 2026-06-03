# Skill candidates: ad-hoc moves to promote into documented skills

These are rituals you repeat by hand across sessions. Each entry gives a name, the exact trigger, the steps, and the existing skills it should compose (so you wire, not rebuild). Ordered by payback.

Convention: most of these are thin wrappers that chain skills you already own. Build them with `skillify` or `example-skills:skill-creator`.

---

## 1. `/design-first` (highest real-friction fix in your app work)

**Why.** subradar and mailplus both built first and reworked after ("this looks offbrand, redo," "make it native looking, it looks bad now"). Figma MCP and your design skills went unused. askwell, which designed first, had far less rework.

**Trigger.** Before writing any UI component in a new front-end, SaaS, or Electron project. Also when a prompt contains "build the app/UI/screen" and no mockup exists yet.

**Steps.**
1. Collect or generate reference screenshots (the look you are copying) and ask for the brand constraints (font, palette, light or dark default).
2. Call Figma MCP (`generate_figma_design` / `use_figma`) or `design-html` to produce high-fidelity mockups from the references.
3. Extract design tokens (colors, typography, spacing) into the Tailwind config before any component code.
4. Run `design-review` on the mockups and get a go.
5. Only then scaffold components, binding them to the locked tokens.

**Composes.** Figma MCP, `design-html`, `design-review`, `design-shotgun`, `high-end-visual-design`.

---

## 2. `/launch-kit` (the non-code half of shipping a product)

**Why.** Every product repeats the same sequence. On askwell: "create media, blurbs, taglines, positioning statement, pitch," then pitch deck, then demo, then "check ycombinator and producthunt for viability, calculate the business model."

**Trigger.** When a new product gets a name, or you say "create the content/pitch/positioning for X," or right after `office-hours` validates an idea.

**Steps.**
1. Run `office-hours` to pressure-test the premise and the wedge.
2. Generate 5 to 8 name options plus a one-line positioning statement and three taglines.
3. Write blurbs (short, medium, long) and a one-page pitch into a `business/` folder.
4. Build a pitch deck and a demo (hyperframes or remotion) referencing the locked design.
5. Run viability research (YC, Product Hunt, competitors) and a simple business-model calculation.
6. Store the decision record to Memento under the project namespace.

**Composes.** `office-hours`, `gloss-brand`, `substack-editor`, `nanobanana`, `hyperframes`, `company-research`, `make-pdf`.

---

## 3. `/ship-it` (deploy plumbing in one move)

**Why.** You wired GitHub plus Railway plus Postgres plus env by hand on askwell and literally typed "create the github skill please."

**Trigger.** When a project is ready to deploy, or you say "connect github," "deploy to railway," "wire up the database."

**Steps.**
1. Read tokens from `.env` by name (never paste secret values into chat).
2. Create or connect the GitHub repo and push.
3. Create the Railway project and service, set env vars from `.env`.
4. Provision Postgres (or the requested DB) and inject the connection string.
5. Deploy, then return the live URL and a one-line status.

**Composes.** `new`, `deploy`, `database`, `domain`, `environment`, `service`, GitLab/Vercel MCP as needed. Extends your existing Railway skill set rather than replacing it.

---

## 4. `/edit-chain` (content pipeline as one pass)

**Why.** You run Humanizer (37x) and deep-detect (14x) as separate manual steps in a content-heavy practice (PERSONAL-Content is your biggest project at 30.9 active hours).

**Trigger.** `/edit-chain {file} {steps}`, for example `/edit-chain article.md grammar,humanize,fact-check,polish`. Also after any first draft of a blog, Substack, or LinkedIn post.

**Steps.**
1. Read the target file.
2. For each requested step, call the matching skill (Humanizer, deep-detect, a WebSearch fact-check, a shorten/polish pass).
3. Git-commit after each step with message `edit-chain: {step}`.
4. Show the diff and ask "apply?" before the next step.
5. Return the final path and the version count.

**Composes.** `Humanizer`, `deep-detect`, `substack-editor`, `browse` (for fact-check), `gloss-publish`.

---

## 5. `/session-wrap` (capture knowledge on the way out)

**Why.** Sessions end without a captured artifact, so code, decisions, and created issues get re-transcribed by hand. Write is your 4th-highest tool at 2,405 calls. There are zero STATUS.md files.

**Trigger.** SessionEnd hook (preferred, automatic) or an explicit `/wrap` command.

**Steps.**
1. Extract code blocks, any StructuredOutput findings, and the session's git commits.
2. List Memento stores and Linear issues created in-session.
3. Write `{project}/sessions/{timestamp}.md` and refresh `STATUS.md` (current status, last session, blockers, next steps).
4. Store a one-line summary to Memento, tagged by project and date.

**Composes.** `context-save`, Memento MCP, Linear MCP. Rides the same SessionEnd hook as the memory wiring in `automations.md`.

---

## 6. `/pre-impl-check` (validate schema and algorithm before building)

**Why.** subradar shipped detection false positives ("this is an obvious subscription, why is this flagged") and a date-reconciliation bug because nothing validated the schema or the algorithm against edge cases first. You use `/redteam` (213x) only for security, never for algorithm correctness.

**Trigger.** Any task involving a data schema, an accuracy-critical algorithm, or messy input parsing (CSV imports, detection rules, reconciliation).

**Steps.**
1. Call `/codex` with the exact requirements plus real examples to generate a typed schema with field-level comments.
2. Call `/codex` or `/redteam` to enumerate around 20 adversarial edge cases (partial-name matches, same-day recurring, family-plan variants, header rows in CSVs) and a test suite.
3. Store the validation output to Memento as `pre-impl-{feature}`.
4. Implement only after the test suite passes.

**Composes.** `codex`, `redteam`, `office-hours`, Memento MCP.

---

## 7. `/discover` (give open-ended ideation a stopping condition)

**Why.** "Find the gap and fill it" and "check my history and create something, anything" looped without converging and once produced "It's empty." 66 WebSearch calls in projects-project with no spec.

**Trigger.** Any open-ended "find me an opportunity / build me something" request.

**Steps.**
1. Set the stopping condition up front: "score the top 3 opportunities by reach times fit times difficulty, then stop."
2. Research, score each against a fixed rubric, store to Memento as `opportunity-decision`.
3. Force a go / no-go decision.
4. On go, hand the decision record to `/codex` for a one-page MVP spec before any code.

**Composes.** `office-hours`, `company-research`, `codex`, Memento MCP.

---

## Notes on triggers

For the four that should be automatic (session-wrap, and the recall side of memory), put them in hooks via `update-config`, not in your head. The harness runs hooks; preferences and memory cannot enforce "always do X on session end." That distinction is why these stay manual today.
