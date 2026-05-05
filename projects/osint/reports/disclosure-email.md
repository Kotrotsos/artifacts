**To:** security@unomundi.com (cc: legal@unomundi.com, dpo@unomundi.com if applicable)
**From:** [operator email — maildmz@gmail.com if no engagement-specific address]
**Subject:** [URGENT — Security] Unauthenticated production database admin access — please read today
**Sensitivity:** Confidential
**Date:** 2026-05-05

---

Hello Unomundi team,

This is a time-sensitive notification from the security engagement currently in progress against your infrastructure (engagement reference 2026-05-05, scope contract and DPA on file). I'm sending this directly to your security alias because two of the findings warrant action **today** — within hours, not within a sprint.

## The two findings that need action today

**1. Your self-hosted Supabase Studio at `https://supabase.unomundi.com` is reachable on the public internet without authentication.**

A visitor reaching that URL is dropped into the Studio Table Editor with full read and write access to every table in your production database — including `auth.users`, `auth.sessions`, `explorer_profiles`, `explorer_login_otps`, `explorer_pairing_codes`, `consent_records`, and the rest of your application schema. Studio uses an internal service-role key to talk to Postgres, which means it bypasses every RLS policy you have configured. Anyone on the internet can issue arbitrary SQL against the database — read children's profile rows, modify consent records, drop tables, or harvest active session tokens.

We confirmed this empirically today. We did not exfiltrate row data from sensitive tables; we used the structural responses (the unauthenticated stub-profile endpoint) to prove the access vector existed, and the operator independently confirmed via browser that the Studio Table Editor renders production data including children's ages and country codes.

The same exposure exists at `https://dev-supabase.unomundi.com`. Production is the higher-priority of the two.

**Suggested immediate mitigation (one of these, in order of preference):**
- Put nginx `auth_basic` or Cloudflare Access in front of `*.supabase.unomundi.com` right now. This is a 5-minute change.
- Configure `DASHBOARD_USERNAME` and `DASHBOARD_PASSWORD` env vars in the Studio container and restart.
- Block the host at the firewall and bring it back up only inside a VPN.

The Supabase self-hosting documentation explicitly warns that Studio "MUST be protected" when self-hosted: https://supabase.com/docs/guides/self-hosting/docker

**2. The dev environment's Kong API gateway is also internet-reachable** (`http://dev-supabase.unomundi.com:8000`), and `pg_stat_statements` is selectable from the `public` schema by the anon role. This leaks query metadata (call counts, timings) for the entire database, plus Postgres internals. Less critical than #1 but should be remediated this week.

## Why I'm flagging the legal clock

Because the product is for children ages 6-12, **GDPR Art. 33 likely applies**. Article 33 obligates the controller (Unomundi) to notify the Dutch Autoriteit Persoonsgegevens within 72 hours of becoming aware of a personal-data breach, unless the breach is unlikely to result in risk to the rights and freedoms of the data subjects. In a children's context the threshold for "unlikely" is high.

This email is the awareness moment. Your 72-hour clock starts now (whichever business interpretation of "awareness" you take, the safe reading is "today"). I'd recommend looping in your DPO and legal counsel today, even before remediation completes, so the documentation trail is clean.

This is not an "if and when" — Studio has been open for at least the duration of our engagement (6+ hours of confirmed exposure today) and likely much longer based on the Postgres uptime our health probes returned (~17 days since last app restart, longer for the database itself). Treat the exposure window as "since deployment" until you can confirm otherwise from Postgres audit logs.

## Other findings — important, not same-day

These are detailed in the full report (attached). Brief summary:

- **F-3:** the production admin SPA at `admin.unomundi.com` was built with `VITE_SUPABASE_URL=https://dev-supabase.unomundi.com` — your prod admin is pointing at dev. Either env-var injection failed in CI, or prod and dev are not actually segregated. Either way, this needs investigation.
- **F-4:** `https://api.unomundi.com/docs` exposes your full OpenAPI spec publicly. Restrict to internal IPs.
- **A-1:** GoTrue's anonymous-signup is enabled and auto-creates a `role: "guardian"` profile for any anonymous registrant. Anyone on the internet can become a "parent" in your data model with no email verification. For a children's product, this dilutes the consent model under GDPR Art. 8.
- **A-2:** roughly 6,500 rows of educational content (350 lessons, 256 quiz questions, 1,024 quiz answers, 566 gallery images, etc.) are readable by any anonymously-registered user. Possibly intentional preview, possibly leakage of paid IP — would appreciate your confirmation of intent.
- **F-5..F-15:** dev-environment internet exposure, security headers on the admin panel, DMARC tightening, CAA records, and similar hygiene. Itemized in the report.

## What I have, what I will and won't do

I have:
- Schema dump (no row data) of all 59 tables visible via PostgREST.
- `pg_stat_statements` metadata-only (call counts, timings; query texts only for queries my own session ran; parameters canonicalized to `$1, $2, …` — no PII).
- A row-count map of which tables are visible to the anonymous and authenticated-anon roles.
- The 6,500-row content catalogue dump from the A-2 finding (catalogue content only — no children's data, no parents' data, no consent records).
- One auto-provisioned anonymous user account I created for the authenticated-role test, UUID `9e8efcb3-a753-41dc-9bae-614b70a7dd51`. Cleanup SQL is in the report.

I have not:
- Read row data from `explorer_profiles`, `consent_records`, `guardian_explorer_relations`, `analytics_events`, `lesson_progresses`, screen-time tables, or any other table containing children's or parents' personal data.
- Run SQL through the open Studio.
- Modified any data.

I will not:
- Take further actions against your infrastructure until you've acknowledged this email and confirmed next-step expectations. Engagement is paused.

## What I'd like from you

1. **Acknowledge receipt** today.
2. **Take Studio off the public internet** today. Reply confirming when done so I can verify externally.
3. **Loop in your DPO / legal** today. Even if the conclusion is no notification is required, the deliberation needs to be documented.
4. **Tell me if you'd like the engagement to continue** after Studio is closed (there are remaining read-only follow-ups that would benefit from a stable target), or to wind down with a final report.

Per the engagement DPA, the artifacts I hold will be retained until report acceptance + 30 days, then destroyed via `shred -u`. Storage location is the engagement operator's machine only; nothing has been transmitted to any third party. The engagement directory is `/Users/marcokotrotsos/osint/engagements/unomundi-2026-05-05/` if your DPO wants a custody record.

I'm available the rest of today and tomorrow for a call if it would help; reply to this thread with a time and channel.

Best,
[Operator]

---

**Attachments referenced:**
- `findings/findings.md` — full findings report
- `findings/F1-studio-evidence.png` — operator-supplied screenshot of open production Studio
- `findings/unomundi-database-schema.md` — schema reference
- `findings/authenticated-role-probe.md` — authenticated-role analysis
- `findings/A2-content-catalogue-exposure.md` — catalogue exposure analysis
- `findings/pgstat-extraction-analysis.md` — pg_stat_statements analysis
- `findings/count-anon-summary.md` — empirical RLS-coverage map
