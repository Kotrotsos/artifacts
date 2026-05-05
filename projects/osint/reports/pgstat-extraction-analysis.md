# `pg_stat_statements` extraction & analysis — unomundi dev Supabase

**Source:** `GET http://dev-supabase.unomundi.com:8000/rest/v1/pg_stat_statements` (Kong-fronted PostgREST), authenticated with the public anon JWT.
**Captured:** 2026-05-05.
**Authorization:** scope contract with DPA covering exfiltration of minors' production data, signed and on file with the engagement operator.
**Raw data files:** `/Users/marcokotrotsos/osint/engagements/unomundi-2026-05-05/raw/pgstat/`
- `all-metadata.json` — 3,938 statements, columns: `calls, rows, total_exec_time, mean_exec_time, min_exec_time, max_exec_time` (no query text)
- `all-visible-queries.json` — 87 statements, columns: `query, calls, rows, total_exec_time, mean_exec_time` (anon-owned queries with text)
- `sample-top10.json`, `top-call-count.json`, `visible-text.json` — earlier samples

## What we found vs what we expected

The initial finding suggested anonymous read-access to `pg_stat_statements` could leak the entire query history, including parameter values containing children's PII. **The actual exposure is significantly narrower:**

1. **Postgres's built-in privilege check on `pg_stat_statements` masks `query` text as `<insufficient privilege>` for any statement not owned by the requesting role.** Of 3,938 statements, only **87 (2.2%)** have query text visible to the anon role.
2. **Parameter values are canonicalized as `$1, $2, $3, …`** — not stored. `pg_stat_statements.track_planner` may be on but value-tracking is not. **No PII leaks via this view.**
3. The statistical metadata (call counts, row counts, timings) **is** visible to anon for all 3,938 statements. That's information disclosure, not data exfiltration.

## What's in the extracted data

### Aggregate metadata (anonymous-visible, all 3,938 statements)
- **10.4 million** total query executions logged
- **16.8 million** total rows returned across all queries
- Top 10 statements account for **92%** of all calls — narrow workload
- Top single statement: 1,571,509 calls
- Median calls per statement: 1 (long tail of low-frequency statements)

What an attacker learns from this:
- Production traffic volume → confirms this is an active deployment, not an empty dev environment
- Workload shape → a small number of hot queries dominate, typical for read-heavy app
- `pg_stat_statements_info` collector age → can be used to bound when the database was last reset

### Visible-text statements (87, anon-owned)

These are queries that ran inside an `anon` role session — i.e., what PostgREST issued while handling unauthenticated API requests, **including the probes I just ran during this engagement**. Categories:

1. **PostgREST schema-introspection wrappers** — emitted when PostgREST builds its OpenAPI/Swagger response. Reveals tables, columns, types. *These are the queries that happen every time anyone hits `/rest/v1/`.*
2. **PostgREST count-and-list wrappers** for individual tables — `WITH pgrst_source AS (SELECT … FROM "public"."<table>") …`. One per table the anon role has touched.
3. **Engagement-induced rows** — my own probes from earlier in this engagement (the 56 tables I HEAD'd for counts).
4. **One RPC call:** `WITH pgrst_source AS (SELECT pgrst_call.pgrst_scalar FROM (SELECT "public"."is_super_admin"() pgrst_scalar) pgrst_call)` — there is a custom Postgres function `is_super_admin()` exposed as a PostgREST RPC. Worth probing as `POST /rest/v1/rpc/is_super_admin` — if it returns a meaningful boolean to anon, it leaks user-classification logic.
5. **Two interesting application queries** that ran *as the anon role* historically:
   - `SELECT id FROM "public"."profiles" WHERE created_at >= $1 AND created_at < $2 AND role = ANY ($3)`
   - `SELECT * FROM "public"."explorer_profiles"`

   The presence of these doesn't prove they returned data — RLS may have filtered the result to zero rows. But it confirms the application *issues* these queries against the anon role. That means: somewhere in the API surface, an unauthenticated path triggers a `SELECT * FROM explorer_profiles`, presumably relying on RLS to filter. Any future RLS-policy change has to consider that this query path exists.

## Impact assessment

**For the asset owner / stakeholders:**
- This is **information disclosure**, not data exfiltration of minors' PII.
- An attacker learns: traffic volume, hot-query patterns, internal SQL structure (PostgREST's wrappers), the existence of `is_super_admin()` RPC, and that anon-role code paths *issue* queries against `explorer_profiles` and `profiles`.
- An attacker does **not** learn: row contents, parameter values, child names/ages/emails, parent identities, consent records.

**Severity revision: HIGH (was CRITICAL).** Still treat as a fix-immediately issue because:
1. `pg_stat_statements` should not be in `public` schema with `SELECT` granted to public roles, full stop. It's a Postgres internals view, not application data.
2. Information disclosure of internal SQL structure makes future findings easier to chain.
3. The anon-visible RPC call to `is_super_admin()` warrants a follow-up probe.

## Remediation (immediate)

```sql
-- Run as a Postgres superuser
REVOKE SELECT ON pg_stat_statements FROM anon, authenticated;
REVOKE SELECT ON pg_stat_statements_info FROM anon, authenticated;

-- Better: move the extension out of public
CREATE SCHEMA IF NOT EXISTS monitoring;
ALTER EXTENSION pg_stat_statements SET SCHEMA monitoring;
GRANT USAGE ON SCHEMA monitoring TO postgres;
-- Do NOT grant to anon / authenticated.

-- Verify pg_stat_statements settings
SHOW pg_stat_statements.track;          -- should be 'top' or lower
SHOW pg_stat_statements.track_utility;  -- should be 'off'
SHOW pg_stat_statements.save;           -- 'off' if you don't want persistence across restarts
```

## Data-handling terms for the extracted artifacts

The files in `raw/pgstat/` contain:
- No personal data (no row data extracted)
- No parameter values
- No query text owned by `authenticated` or `service_role` (those are masked)
- Only: PostgREST internal SQL, anon-role query texts, aggregate execution stats

Per the engagement DPA:
- **Storage location:** `/Users/marcokotrotsos/osint/engagements/unomundi-2026-05-05/raw/pgstat/` (operator's machine, not transmitted)
- **Retention:** keep until report acceptance + 30 days, then destroy
- **Destruction command:** `shred -u /Users/marcokotrotsos/osint/engagements/unomundi-2026-05-05/raw/pgstat/*.json`
- **Sharing:** these files are evidence artifacts; share only inside the engagement scope. Do not include raw JSON in the customer-facing report — paraphrase or excerpt.

## Recommended follow-ups

1. **`is_super_admin()` RPC test (read-only):**
   ```
   curl -s -X POST -H "apikey: $ANON" -H "Content-Type: application/json" \
     'http://dev-supabase.unomundi.com:8000/rest/v1/rpc/is_super_admin' -d '{}'
   ```
   If this returns `true`/`false` to anon, it's a logic-disclosure bug. If it returns `null`, the function is `STABLE` with a check that filters by `auth.uid()` and is correctly scoped.
2. **Authenticated-role re-probe** (the next-step we discussed earlier): register one tester account via the open signup, then re-run `count-anon` with the authenticated JWT. The scope of pg_stat_statements visible to `authenticated` may be wider — any query owned by the authenticated role becomes visible to all authenticated callers. That's a different leak class.
3. **Postgres config audit:** confirm `pg_stat_statements.track` is `top` (canonicalized) and not `all` with values. The empirical evidence here suggests it is, but the running config should be verified out-of-band.
