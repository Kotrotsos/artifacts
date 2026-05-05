# Admin host gap-fill — results

**Hosts:** `admin.unomundi.com`, `dev-admin.unomundi.com`
**Captured:** 2026-05-05.
**Authorization:** scope contract + DPA on file.

## Summary

| Check | Result |
|---|---|
| Source-map leaks (`/assets/*.js.map`) | **None** — all return SPA catch-all (1193 bytes) |
| Build-artifact exposure (`.env`, `.git/HEAD`, `.DS_Store`, `package.json`, `vite.config.*`, etc.) | **None** — all return SPA catch-all |
| Admin `nginx_status` / `server-status` / OIDC well-known | **None** — all return SPA catch-all |
| TLS audit | **Clean** — TLS 1.3 + AES_256_GCM_SHA384, modern ECDHE-ECDSA cipher on TLS 1.2, TLS 1.0 disabled |
| Storage bucket listing without auth (`/storage/v1/bucket`, `/storage/v1/object/list/<bucket>`) | **Clean** — 400 "headers must have required property 'authorization'" |
| Admin Edge Function `admin-dashboard-cards` invocation | **Finding A-4** — no top-level admin-role check |

The static-asset hardening is **good**: nginx `try_files` falls through to `index.html` for every unknown path, hiding all build-artifact patterns. That's the right setup for a Vite SPA.

The dynamic surface (Edge Functions) is where the new finding sits.

## A-4 — Admin Edge Function `admin-dashboard-cards` lacks role check

**Severity:** **MEDIUM** (no admin data confirmed leaked, but the auth model is broken; sibling mutating function untested)
**Endpoint:** `POST/GET https://dev-supabase.unomundi.com:8000/functions/v1/admin-dashboard-cards`

### Evidence

```
# Without auth header
$ curl -i -H "apikey: <anon>" .../admin-dashboard-cards
HTTP/1.1 401 Unauthorized
{"error_code":"auth_error","error":"Error: Missing authorization header"}

# With our anonymous-registered guardian's JWT (role: 'guardian', is_anonymous: true)
$ curl -i -H "apikey: <anon>" -H "Authorization: Bearer <user-jwt>" .../admin-dashboard-cards
HTTP/1.1 500 Internal Server Error  (after 4842ms upstream)
{"error_code":"internal_server_error","error":"Internal server error"}
```

A correctly-authored admin function should return **403 Forbidden** when called by a non-admin authenticated user, not crash with a 500. The 4.8-second upstream latency before the error indicates the function ran real business logic against the request before failing — meaning the role-check happens (if at all) inside the data layer rather than at the function's entry. Carelessly authored variants of this pattern routinely leak admin data via error messages, partial responses, or side-channel timing.

### Why this matters

1. **Sibling function risk.** The bundle references `admin-create-legal-document-version` alongside `admin-dashboard-cards`. If they share the same auth template, any anonymously-registered user can probably publish fake legal-document versions to the production database — directly affecting the consent records governing minors' data. We deliberately did not invoke it (it's a mutating endpoint).
2. **Authorization model leakage.** A 401-vs-500 difference confirms to an attacker that the function is reachable by authenticated callers; they can then probe other admin functions to find one that 200s without a clean role check.
3. **No defense-in-depth.** If the database layer's RLS were ever to regress (a fix-then-break cycle is the most common vector for privilege creep), the absence of a function-layer gate means admin functions become directly exploitable.

### Reproduction

```bash
# 1. Create anonymous user (anonymous_users:true is enabled — see F-2)
USER_JWT=$(curl -sS -X POST -H "apikey: $ANON" -H "Content-Type: application/json" \
  -d '{}' http://dev-supabase.unomundi.com:8000/auth/v1/signup | jq -r .access_token)

# 2. Invoke admin function with guardian-role JWT
curl -i -H "apikey: $ANON" -H "Authorization: Bearer $USER_JWT" \
  http://dev-supabase.unomundi.com:8000/functions/v1/admin-dashboard-cards
```

### Remediation

In every admin Edge Function, add an explicit role check at the top, before any business logic:

```ts
// Deno / Supabase Edge Function pattern
const { data: { user }, error } = await supabaseClient.auth.getUser(jwt);
if (error || !user || user.is_anonymous) {
  return new Response(JSON.stringify({ error: "forbidden" }), { status: 403 });
}
const { data: profile } = await supabaseAdmin
  .from("profiles")
  .select("role")
  .eq("id", user.id)
  .single();
if (!profile || profile.role !== "admin") {
  return new Response(JSON.stringify({ error: "forbidden" }), { status: 403 });
}
// … rest of function …
```

For mutating admin functions especially (`admin-create-legal-document-version`, anything referenced in `admin-bundle.js`):
- Add the role check above.
- Add a check that the caller is not an anonymous user (`!user.is_anonymous`).
- Add a check that the caller's email is verified (`user.email_confirmed_at !== null`).
- Audit the function code for any pattern that returns data on the error path.

### Audit checklist for the asset owner

```
[ ] List all Edge Functions deployed:
    supabase functions list  (against the production project)

[ ] For each function, verify the entry point does:
    1. JWT validation
    2. is_anonymous = false check (for admin functions)
    3. role IN ('admin', 'super_admin') check (for admin functions)
    4. email_confirmed_at IS NOT NULL check (for mutating functions)

[ ] Smoke-test each admin function with:
    - no JWT                → expect 401
    - anonymous-user JWT    → expect 403
    - real authenticated user with role='guardian' → expect 403
    - real authenticated user with role='admin'    → expect 200
```

## Other findings from the gap-fill

### A-5 — Admin route inventory disclosed via JS bundle

25 admin routes were extracted from the (heavily minified) admin SPA bundle, including `/users/admin`, `/users/app`, `/PASSWORD_RECOVERY`, `/settings/legal`, `/una/context`, `/una/logs`, `/una/settings`, `/chapters/new`, `/countries/new`, `/lessons`, `/analytics`, plus the two Edge Functions above and Supabase Storage signed-URL paths (`/object/sign/`, `/render/image/sign/`).

**Severity:** **INFO** — disclosure of route inventory is normal for an SPA (the bundle has to know its own routes), and the fact that our extraction yielded only 25 routes from a 1.4 MB bundle indicates the minifier is doing real work. But the route names are descriptive enough to seed targeted attack work. Not a finding to fix; just a note that admin URLs are not secrets.

### Clean-bill items

- **No source-map leak.** Vite production build correctly omits `.map` files (or they're not served by nginx). Distinguishing those two is academic — either way, attackers don't get original source.
- **No build-artifact exposure.** The static-host story is right.
- **No nginx_status / server-status leak.** stub_status module not enabled or not bound to a public location.
- **TLS posture good.** Modern protocol/cipher floor; old TLS disabled.
- **Storage bucket listing requires auth.** `/storage/v1/bucket` and `/storage/v1/object/list/<bucket>` both reject without an `Authorization` header.

## Updated severity rollup

| ID | Sev | Title |
|----|---|---|
| **F-1** | **CRITICAL** | Production Supabase Studio publicly accessible without auth |
| F-2a | HIGH | `pg_stat_statements` anonymously readable |
| F-2 | HIGH | Kong gateway exposed on dev-supabase |
| F-3 | HIGH | Production admin built with dev backend URL |
| F-4 | HIGH | Public OpenAPI spec at `api.unomundi.com/docs` |
| **A-4** | **MEDIUM** | Admin Edge Function `admin-dashboard-cards` lacks role check (sibling mutating function untested) |
| A-1 | MEDIUM | Anonymous signup auto-classifies users as `guardian` |
| F-5 | MEDIUM | Dev environments fully internet-reachable |
| F-6 | MEDIUM | Non-standard ports open on backend EC2 hosts |
| F-12 | MEDIUM | Admin panel served without security headers |
| (other findings unchanged) | | |
| **A-5** | **INFO** | Admin route inventory extractable from JS bundle |
