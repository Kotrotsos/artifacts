# Penetration Test Findings — unomundi.com

**Engagement date:** 2026-05-05
**Target:** unomundi.com and all discovered subdomains
**Tester:** maildmz@gmail.com (self-attested authorization)
**Methodology:** passive OSINT → asset discovery → fingerprint → targeted endpoint probing → port scan → vuln scan. Active SQL execution and data exfiltration were deliberately *not* performed despite available access; structural proof was sufficient.

---

## Executive summary

The marketing site (`unomundi.com`) is on Wix and is largely fine. The product backend, however, is a self-hosted Supabase + NestJS deployment on AWS us-east-1 with **multiple critical exposures**:

1. **Self-hosted Supabase Studio is publicly accessible without authentication** on both production and dev. Studio internally uses a service-role Postgres key — anyone on the internet can issue arbitrary SQL.
2. **Direct Kong API gateway** on `dev-supabase:8000` is wide open to the internet, leaking the **entire database schema as a 245 KB Swagger document** (59 tables, 99 endpoints) including `explorer_profiles` (children), `guardian_explorer_relations` (parents), `consent_records`, `explorer_screen_time_*`, and `pg_stat_statements`.
3. The **production admin panel** was built pointing at the **dev** Supabase backend — either the build pipeline leaks dev env vars into prod or prod and dev are not actually segregated.
4. Anonymous signup is enabled at the auth layer (`anonymous_users:true`, `disable_signup:false`), so the auth path itself is open registration.

Because the product is for children ages 6-12 and operated from the Netherlands, this combination triggers **GDPR-K / AVG / COPPA** obligations. Under GDPR Art. 33 a 72-hour breach-notification clock may already apply once exposure to any personal data is established, even if no exfiltration is confirmed. Recommend treating P0 items as incident-response work, not just engineering tickets.

| ID | Severity | Title |
|----|----------|-------|
| F-1 | **CRITICAL** | Self-hosted Supabase Studio publicly accessible without authentication |
| F-2 | **CRITICAL** | Kong API gateway exposed on `dev-supabase:8000`; full DB schema disclosed; anonymous signup enabled |
| F-3 | **HIGH** | Production admin panel built with dev backend URLs (env-var leak or environment merge) |
| F-4 | **HIGH** | Public OpenAPI spec at `api.unomundi.com/docs` reveals full endpoint inventory |
| F-5 | **MEDIUM** | Dev environments (`dev-admin`, `dev-api`, `dev-supabase`) fully internet-reachable |
| F-6 | **MEDIUM** | Multiple non-standard ports (3000, 5000, 5060, 8080, 8443) open on backend hosts |
| F-7 | **LOW** | `/health` discloses uptime and DB status |
| F-8 | **LOW** | nginx version drift across hosts (1.27.5 vs 1.28.2) |
| F-9 | **INFO** | CAA records absent |
| F-10 | **INFO** | DMARC `p=quarantine` (not `reject`) |
| F-12 | **MEDIUM** | Admin panel served without security headers; `robots.txt` `Allow: /` |
| F-13 | **LOW** | API 404 reflects request path in JSON body |
| F-14 | **LOW** | `Access-Control-Allow-Origin: *` on api.unomundi.com |
| F-15 | **INFO** | SSH (22/tcp) reachable on all backend hosts; AWS EC2 rDNS hostnames disclosed |

---

## Asset inventory

| Host | IP | Cloud | Stack | Role |
|---|---|---|---|---|
| unomundi.com / www.unomundi.com | 185.230.63.171/186/107, 34.160.37.117 | Wix + GCP CDN | Wix CMS, React, HTTP/3 | marketing |
| api.unomundi.com | 98.82.112.82 | AWS us-east-1 | nginx 1.28.2 → NestJS | prod API |
| admin.unomundi.com | 100.52.249.43 | AWS us-east-1 | nginx 1.27.5 → React SPA (Lovable.dev build) | prod admin |
| supabase.unomundi.com | 3.238.254.151 | AWS us-east-1 | nginx 1.28.2 → Supabase Studio (Next.js) | prod DB admin |
| dev-api.unomundi.com | 54.90.109.3 | AWS us-east-1 | nginx 1.28.2 → NestJS | dev API |
| dev-admin.unomundi.com | 98.80.252.176 | AWS us-east-1 | nginx 1.27.5 → React SPA | dev admin |
| dev-supabase.unomundi.com | 100.52.182.111 | AWS us-east-1 | nginx + Kong 2.8.1 + GoTrue v2.186.0 + Supabase Studio | dev backend |

---

## Findings

### F-1 — CRITICAL — Self-hosted Supabase Studio publicly accessible without authentication
**Hosts:** `supabase.unomundi.com`, `dev-supabase.unomundi.com`

**Evidence:**
```
GET https://supabase.unomundi.com/                     → 307 → /project/default
GET https://supabase.unomundi.com/project/_/sql        → 200  (SQL editor UI)
GET https://supabase.unomundi.com/api/platform/profile → 200
{"id":1,"primary_email":"johndoe@supabase.io","username":"johndoe",
 "first_name":"John","last_name":"Doe",
 "organizations":[{"id":1,"name":"Default Organization","slug":"default-org-slug",
   "billing_email":"billing@supabase.co",
   "projects":[{"id":1,"ref":"default","name":"Default Project",...}]}]}
```
The `johndoe@supabase.io / Default Organization` body is the canonical stub Studio returns when no `DASHBOARD_USERNAME` / `DASHBOARD_PASSWORD` is set in the docker-compose `.env`. Studio uses an internal service-role key to call pg-meta, which means anyone reaching the UI can issue arbitrary SQL via `/project/default/sql/new`.

**Impact:** read/modify/drop any table in production Postgres, including children's PII (see F-2 schema dump for table list).

**Remediation (priority order):**
1. **Immediate:** add `DASHBOARD_USERNAME` / `DASHBOARD_PASSWORD` env vars to the Studio container, OR put nginx `auth_basic` / Cloudflare Access in front of `*.supabase.unomundi.com` *now*.
2. **Short term:** move Studio off the public internet entirely. Bind it to localhost or to a VPC-internal subnet, access via SSH tunnel / Tailscale / Cloudflare WARP.
3. **Audit:** review Postgres logs for the past 30 days for queries originating outside expected app sources. Rotate any secrets that may have been retrievable from the DB.
4. **Compliance:** consult counsel on GDPR Art. 33. Even without confirmed exfiltration, the access vector existed during the exposure window.

**References:** https://supabase.com/docs/guides/self-hosting/docker (Studio "MUST be protected" warning)

---

### F-2 — CRITICAL — Kong API gateway exposed; full DB schema disclosed; anonymous signup enabled
**Host:** `dev-supabase.unomundi.com:8000` (HTTP), `:8443` (HTTPS), Kong 2.8.1

The internal Supabase API gateway is reachable from the internet on these ports. Production's port 8000 is correctly firewalled (timeout) — only dev is exposed. But because the prod admin panel is built against dev (see F-3), this gap matters operationally.

**What's exposed (read-only confirmation):**
```
GET  http://dev-supabase.unomundi.com:8000/rest/v1/   → 200, 245 KB Swagger
     Title: "standard public schema", Postgres version 14.5
     59 tables/views, 99 paths

GET  /auth/v1/health                  → 200  GoTrue v2.186.0
GET  /auth/v1/settings                → 200  email signup ON, anonymous_users ON,
                                              disable_signup OFF, mailer_autoconfirm OFF
GET  /functions/v1/                   → 400  (Edge Functions reachable, missing fn name)
GET  /storage/v1/health               → 400  (Storage requires JWT — correct)
```

**Tables exposed in public schema (truncated to high-signal ones):**
```
explorer_profiles                       ← child profiles
guardian_explorer_relations             ← parent ↔ child mapping
guardian_settings, guardian_action_logs ← parent activity
explorer_settings
explorer_screen_time_daily_usage        ← per-child behavior
explorer_screen_time_state
consent_records                         ← consent (likely parental)
profiles
chapter_progresses, lesson_progresses, lesson_ratings
quiz_answers, quiz_questions
analytics_events                        ← telemetry per session
una_session_daily_metrics
report_types                            ← moderation/abuse reports
legal_document_versions
pg_stat_statements                      ← Postgres internals (should never be in public schema)
schema_migrations                       ← migration history
locales, localizations, country_translations, …
```

99 REST paths (one GET / POST / PATCH / DELETE per table) all accept the `apikey` header. Whether any of these tables expose data depends entirely on the Postgres RLS policies. We did **not** query rows; the structural disclosure alone is enough to flag.

**Adjacent risk: anonymous_users + disable_signup=false.** Anyone can register. With registration the user gets an authenticated `eyJ…role:authenticated` JWT, which often loosens RLS. An attacker can self-register and re-test every endpoint with an `authenticated` claim.

**`pg_stat_statements` in the public schema is itself a finding** — it's a `postgres` superuser-owned view of every query that has run on the DB, including parameter values. If it is selectable via the REST API at all, it leaks query history.

**Remediation:**
1. **Firewall** Kong ports (8000, 8443) on the dev EC2 host. They should only be reachable from the app's internal subnet or via a bastion.
2. Drop `pg_stat_statements` from the public schema. It belongs in a `monitoring` schema with no anon grant.
3. Audit RLS policies on every table — every public-schema table needs an explicit policy or `disable RLS` decision. Default to `enable RLS` and write deny-by-default.
4. Set `disable_signup=true` for the dev project unless dev is intentionally an open-signup playground (and then it must not share data with prod).
5. Consider whether `anonymous_users` is required; if not, disable.
6. Re-issue API keys (anon and service-role) after the firewall is in place.

---

### F-3 — HIGH — Production admin panel built with dev backend URLs
**Host:** `admin.unomundi.com` (production)

**Evidence:** the prod JS bundle `/assets/index-LVG8KGiM.js` contains:
```
VITE_BACKEND_URL: "https://dev-supabase.unomundi.com"
VITE_SUPABASE_URL: "https://dev-supabase.unomundi.com"
VITE_SUPABASE_ANON_KEY: "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJyb2xlIjoiYW5vbiIsImlzcyI6InN1cGFiYXNlIiwiaWF0IjoxNzcyNTY1NzczLCJleHAiOjE5MzAyNDU3NzN9..."
```
Anon JWT decoded: `role=anon, iss=supabase, iat=2026-03-04, exp=2031-03-08`.

**Implication:** either CI is failing to inject prod env vars (and dev values are baked in), or there is no real prod backend and "dev-supabase" is the single live environment. Combined with F-1 + F-2, the prod admin is operating against an unauthenticated database with a public Kong gateway.

**Remediation:**
1. Verify Vite build pipeline injects environment-specific `.env` files (`vite build --mode production` and a `.env.production` file with prod URLs).
2. Confirm prod and dev databases are physically separate. If not, separate them and migrate.
3. Re-issue anon JWTs after segregation.
4. Add a build-time guard: fail the build if `VITE_SUPABASE_URL` matches `dev-*` when targeting prod.

---

### F-4 — HIGH — Public OpenAPI spec discloses full API endpoint inventory
**Hosts:** `api.unomundi.com/docs`, `dev-api.unomundi.com/docs`

`/docs` returns a Scalar API Reference page revealing the NestJS app's full route map:
- `POST /ai/session` — ElevenLabs conversation session creation (signed URL)
- `GET  /ai/conversation-token` — voice conversation auth token
- `POST /content-import/lessons` — admin lesson import (x-api-key)
- `GET  /video/playback-token/{lessonId}` — signed video URL
- `GET  /health`

The auth layer is correctly enforced (returns 401 on protected paths without bearer tokens), but the inventory disclosure accelerates targeted attacks. The `/ai/session` path is especially sensitive given the children's voice-AI use case.

**Remediation:** restrict `/docs` to internal IPs in production. Set `app.use('/docs', ipFilter([INTERNAL_CIDR]))` or remove the route entirely on prod and host the spec elsewhere.

---

### F-5 — MEDIUM — Dev environments fully internet-reachable
**Hosts:** `dev-admin.unomundi.com`, `dev-api.unomundi.com`, `dev-supabase.unomundi.com`

All three resolve publicly with valid Let's Encrypt certs and were discovered via certificate transparency. They run identical software stacks to prod, so any latent bug in dev is also exploitable in prod (mod F-3). Dev is *also* the host where Kong was left open (F-2).

**Remediation:** put dev behind VPN / Cloudflare Access / IP allowlist. Consider a private CA for dev TLS so dev hosts don't appear in CT logs.

---

### F-6 — MEDIUM — Non-standard ports open on backend EC2 hosts
**Hosts:** all backend hosts

```
admin.unomundi.com         22, 53, 80, 443, 5060, 8080
api.unomundi.com           22, 53, 80, 443, 5060, 8080
supabase.unomundi.com      22, 53, 80, 443, 5060, 8080
dev-admin.unomundi.com     22, 53, 80, 443, 5060, 8080  (8080 = nginx 1.27.5)
dev-api.unomundi.com       22, 53, 80, 443, 3000 (NestJS), 5060, 8080
dev-supabase.unomundi.com  22, 53, 80, 443, 3000 (Studio), 5000 (NestJS-style), 5060,
                           8000/tcp (Kong/2.8.1), 8080, 8443 (Kong TLS)
```

- Port **3000** on dev-api/dev-supabase is the underlying app process exposed directly, bypassing nginx; clients can hit the app without any reverse-proxy hardening.
- Port **5000** on dev-supabase returns a NestJS-style 404 — likely Supabase Auth or Storage internal port directly exposed.
- Port **5060** on every host (sip?) — likely AWS-internal or a misconfig. Worth investigating; SIP exposure is unusual on a web stack.
- Port **8080** is open everywhere; on dev-admin it's another nginx/1.27.5 (same admin panel?).

**Remediation:** EC2 security groups should allow only 80/443 (and 22 from a bastion / company VPN). Lock down everything else. Consider using AWS Session Manager instead of public 22.

---

### F-7 — LOW — `/health` endpoint discloses uptime and DB status
**Hosts:** `api.unomundi.com/health`, `dev-api.unomundi.com/health`

```
{"status":"ok","uptime":1548606.978547315,"responseTimeMs":87,"db":"ok"}
```
Uptime ~17.9 days indicates last deploy time, useful for timing attacks. `db:"ok"` confirms the API has DB connectivity.

**Remediation:** make public `/health` a thin liveness ping (`{"ok":true}`); keep verbose status on `/health/internal` or behind auth.

---

### F-8 — LOW — nginx version drift across hosts (1.27.5 vs 1.28.2)
Admin panels run nginx 1.27.5 (Apr 2025); other hosts run 1.28.2. Both are mainline branches. Align versions and consider stable-branch (1.26.x at time of writing) for production.

---

### F-9 — INFO — CAA records absent
Any public CA can issue certs for `unomundi.com`. Add:
```
unomundi.com. IN CAA 0 issue "letsencrypt.org"
unomundi.com. IN CAA 0 iodef "mailto:security@unomundi.com"
```

---

### F-10 — INFO — DMARC `p=quarantine`
`_dmarc.unomundi.com` → `v=DMARC1; p=quarantine; rua=mailto:dmarc@unomundi.com`

For a children's brand, recommend tightening to `p=reject` after a 2-week monitoring window confirms no legitimate mail is being misclassified. Add `ruf=` for forensic reports.

---

### F-12 — MEDIUM — Admin panel served without security headers
**Hosts:** `admin.unomundi.com`, `dev-admin.unomundi.com`

nikto + manual `curl -I` confirm the admin SPA is served with **none** of: HSTS, CSP, Referrer-Policy, Permissions-Policy, X-Content-Type-Options. It also returns `Content-Encoding: deflate`, which keeps the BREACH attack surface on the table for any pages that reflect attacker-controlled input alongside secrets. The marketing site (`www.unomundi.com`) and the API (`api.unomundi.com`) both have HSTS and X-Content-Type-Options at minimum; the admin panel does not.

The admin `robots.txt` further has `Allow: /` for every user-agent — admin panels should be `Disallow: /` to keep them out of search indexes.

**Remediation:**
- Add nginx-level headers on admin: `Strict-Transport-Security: max-age=31536000; includeSubDomains; preload`, `Content-Security-Policy: default-src 'self'; script-src 'self'; …`, `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Referrer-Policy: strict-origin-when-cross-origin`.
- Replace admin `robots.txt` body with `User-agent: *\nDisallow: /`.

### F-13 — LOW — API returns reflected path in JSON 404 body
**Host:** `api.unomundi.com`

`GET /<anything>` returns `{"message":"Cannot GET /<anything>","error":"Not Found","statusCode":404}`. Path reflection isn't exploitable as XSS because the response is `application/json` and CSP restricts script-src; nikto false-positives flagged it as XSS but those are not real. Still — the reflection is unnecessary noise that helps phishing-style typosquat URLs look more "official" and reveals the framework's default error format.

**Remediation:** strip path from the public 404 message; return `{"error":"Not Found","statusCode":404}` only.

### F-14 — LOW — `Access-Control-Allow-Origin: *` on api.unomundi.com
The wildcard CORS policy is benign because the API uses `Authorization: Bearer …` (which is not auto-attached cross-origin) and not credentials/cookies. However, restricting `Access-Control-Allow-Origin` to a known list of frontend origins (admin, mobile webview) is best practice and would block any future cookie-auth path from accidentally being exploitable.

### F-15 — INFO — SSH on all backend EC2 hosts; AWS EC2 rDNS hostnames disclosed
All backend hosts have port 22 open and rDNS records of the form `ec2-X-Y-Z-W.compute-1.amazonaws.com` — confirms AWS us-east-1 deployment and gives attackers the EC2 instance public IPs.

**Remediation:** restrict 22/tcp via security group to the VPN egress IPs or move to AWS Systems Manager Session Manager. Public-facing SSH is unnecessary if you have SSM.

---

## What was checked and is OK

- DMARC published (just below ideal policy; F-10)
- SPF: `v=spf1 include:_spf.google.com include:spf.mailjet.com ~all` — proper mail provider declaration
- TLS: valid Let's Encrypt certs with proper SANs across all subdomains
- API authentication on `/ai/session`, `/ai/conversation-token`, `/video/playback-token/{id}` correctly enforces "Missing or invalid Authorization header"
- Storage API at `/storage/v1/health` correctly demands a JWT
- Production Kong port 8000 is firewalled (timed out from internet) — dev is the leak
- Email/Mailjet provider configured

---

## Out of scope / not tested

- **Database queries** through Studio or pg-meta. Structural proof was sufficient; running SQL would risk altering production state and exceed read-only PoC requirements.
- **Authenticated fuzzing** of `/ai/session`, `/content-import/lessons`. Requires credentials.
- **Social engineering / phishing** of staff (out of scope per ROE).
- **Bucket enumeration** of any Supabase Storage buckets (would require auth).
- **Mobile / native app** binary analysis (assumes a kids' app frontend exists; not in scope here).
- **Source code review**, dependency CVE scan, container image audit.

---

## Recommended sequence of remediation work

1. **TODAY:** put basic auth in front of `*.supabase.unomundi.com` and firewall Kong ports 8000/8443 on dev-supabase. Drop `pg_stat_statements` from public schema. Disable anonymous signup + disable open signup on the dev project.
2. **Within 72h (GDPR clock):** review Postgres audit logs for unexpected queries; consult counsel on Art. 33 notification.
3. **Week 1:** fix the prod-admin-points-at-dev-supabase build issue. Separate prod/dev DBs if not already separate. Rotate anon and service-role JWTs.
4. **Week 1:** restrict EC2 security groups to 80/443 + bastion-only SSH. Restrict `/docs` to internal IPs. Add `IP-allowlist` middleware to dev frontends.
5. **Week 2:** RLS audit across all 59 tables; deny-by-default; explicit policies with tests.
6. **Week 2-3:** add build-time guard preventing `VITE_SUPABASE_URL=dev-*` in prod artifacts. Move Studio off public internet permanently.
7. **Ongoing:** version-align nginx; add CAA records; tighten DMARC to `p=reject`; reduce `/health` verbosity.
