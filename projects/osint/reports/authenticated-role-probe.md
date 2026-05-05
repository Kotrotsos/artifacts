# Authenticated-role probe — unomundi dev Supabase

**Source:** dev-supabase.unomundi.com:8000 (Kong gateway).
**Captured:** 2026-05-05.
**Method:** registered an anonymous user via GoTrue's anonymous signup flow (`POST /auth/v1/signup` with empty body, enabled by `anonymous_users:true`), then re-ran `count-anon` with the resulting authenticated JWT.
**Authorization:** scope contract + DPA on file with operator.

## Test account (for cleanup after engagement)

| Field | Value |
|---|---|
| `auth.users.id` | `9e8efcb3-a753-41dc-9bae-614b70a7dd51` |
| Session ID | `31551840-51ee-49e6-8b6e-f77961fd3141` |
| Email / phone | (none — anonymous) |
| Role | `authenticated` |
| `is_anonymous` | `true` |
| Created | 2026-05-05T11:25:09Z |
| JWT expiry | 30 minutes (refreshable) |

**Cleanup SQL the asset owner should run after engagement closes:**
```sql
DELETE FROM auth.users WHERE id = '9e8efcb3-a753-41dc-9bae-614b70a7dd51';
-- triggers cascade to public.profiles, public.guardian_settings, sessions, refresh_tokens
```

## `is_super_admin()` RPC probe

```
POST /rest/v1/rpc/is_super_admin    →  200  body: false
GET  /rest/v1/rpc/is_super_admin    →  200  body: false
```

Function exists, callable by anon and authenticated, returns the safe answer (`false`) in both contexts. Information disclosure (function name + signature inferred), no privilege escalation. **No action needed.**

## Schema visibility

- Anon role: 59 tables/views advertised in the OpenAPI spec.
- Authenticated role: **65 tables/views.** Six additional tables become visible:
  - `gallery_walk_image_selections`
  - `global_audio_assets`, `global_audio_asset_translations`
  - `global_image_assets`
  - `global_video_assets`, `global_video_asset_translations`

These are global media assets (likely a reusable image/audio/video library used across lessons). Visibility to authenticated only is correct.

## Row-count results (authenticated anon)

| Table | Rows visible | Notes |
|---|---:|---|
| `chapters` | 585 | Catalogue. Visible to all authenticated. |
| `tracks` | 585 | Catalogue. |
| `lessons` | 350 | Catalogue. |
| `quiz_questions` | 256 | Catalogue. |
| `quiz_answers` | 1024 | Catalogue. |
| `quiz_question_translations` | 256 | Catalogue. |
| `quiz_answer_translations` | 1024 | Catalogue. |
| `gallery_images` | 566 | Catalogue. |
| `gallery_image_translations` | 566 | Catalogue. |
| `gallery_screens` | 147 | Catalogue. |
| `gallery_lessons` | 54 | Catalogue. |
| `video_lessons` | 232 | Catalogue. |
| `chapter_types` / `track_types` / `lesson_types` / `report_types` | 3-12 | Lookup tables. |
| `countries` / `country_translations` | 195 | Lookup tables. |
| `locales` / `localizations` | 3 | Lookup tables. |
| `legal_document_versions` | 6 | Active legal docs (privacy policy, ToS versions). |
| `report_type_translations` | 12 | Lookup. |
| `global_*_assets` (audio/image/video + translations) | 1-52 | Global media library. |
| `lesson_translations` / `lesson_types` | 55 / 8 | Lookup + content. |
| **`profiles`** | **1** | Our own auto-provisioned row. |
| **`guardian_settings`** | **1** | Our own auto-provisioned row. |
| `v_user_chapter_progress` | 585 | View of chapter availability per the **current** user (`subject_user_id = auth.uid()`). |
| `v_user_lesson_progress` | 350 | Same pattern. |
| `v_user_track_progress` | 585 | Same pattern. |
| `pg_stat_statements` | 3,983 | Same view as anon, but query text now visible for authenticated-owned queries (still parameterized, no PII). |

**Tables that remained `Content-Range: */0` (RLS blocking authenticated-anon):**

```
analytics_events             chapter_progresses
calluna_lesson_translations  calluna_lessons
chapter_translations         consent_records
explorer_profiles            explorer_screen_time_daily_usage
explorer_screen_time_state   explorer_settings
gallery_import_jobs          gallery_walk_image_selections
guardian_action_logs         guardian_explorer_relations
lesson_progresses            lesson_ratings
lesson_type_translations     quiz_import_jobs
schema_migrations            track_translations
una_session_daily_metrics    user_content_cursor
video_import_jobs            video_lesson_audio_tracks
video_lesson_reports
```

## Reading the three "1-row" results

To confirm whether the visible 1 row was our own data or someone else's, all three were read once. All three returned **our** UUID as the owning identifier. Excerpts:

```json
profiles[0]              { "id": "9e8efcb3-…dd51", "role": "guardian", "is_adult": null, … }
guardian_settings[0]     { "guardian_id": "9e8efcb3-…dd51", "controls": { "ai_guide_enabled": false, "ai_terms_accepted": false }, … }
v_user_chapter_progress[0] { "subject_user_id": "9e8efcb3-…dd51", "chapter_id": "c5f75caf-…", "lessons_completed": 0, … }
```

**Verdict:** RLS is correctly scoping per-user data. Authenticated-anon reads only its own rows. **No leak of other users' children, parents, consent, progress, screen-time, or analytics data.**

## Findings revealed by the authenticated probe

### A-1 — MEDIUM — Anonymous signup auto-creates a `guardian` (parent) profile

**Evidence:** the auto-provisioning trigger creates `public.profiles` with `role: "guardian"` and a corresponding `public.guardian_settings` row whenever a user is inserted into `auth.users` — regardless of whether they signed up via email, OAuth, or anonymous. An anonymous user with no verified identity becomes a "guardian" in the application's data model.

**Why this matters for a children's product:**
- The whole point of a guardian role in a kids'-tech product is to assert verified adult oversight of a minor. Letting anonymous users self-classify as guardians dilutes that assurance.
- The Dutch AVG and GDPR-K assume parental consent is given by an identifiable adult. An "anonymous guardian" cannot give meaningful Art. 8 GDPR consent.
- If any of the consent or guardian-action paths trust `role = 'guardian'` to permit operations on a child account (e.g. linking an explorer to a guardian, granting screen-time, accepting AI terms on the child's behalf), an anonymous-but-authenticated session could perform those actions.

**Remediation options:**
- Default new profiles to a `pending` role until email-verified or explicitly upgraded.
- Block the anonymous signup path entirely (`anonymous_users: false` in GoTrue) unless there's a documented product flow that needs it.
- If anonymous signup is needed for a try-before-you-buy flow, gate guardian-classified actions on `is_adult = true AND email_confirmed_at IS NOT NULL`.

### A-2 — LOW — Educational content reachable by any anonymous registrant

The full catalogue (lessons, chapters, tracks, quizzes, gallery content) is readable by any authenticated user, including anonymous registrations. This may be intentional (browse-before-subscribe), but if the business model assumes content is gated behind a paid or verified guardian, this is leakage of paid IP.

**To verify:** ask the asset owner whether the content tables are intended to be readable by anonymous-authenticated users.

### A-3 — INFO — `is_super_admin()` RPC publicly callable

Callable by anon and authenticated. Returns `false` to both, which is correct. No action needed beyond noting the RPC's signature is inferable from `pg_stat_statements`.

## Engagement state revision

- **F-1 (Studio open) — CRITICAL — unchanged.** Studio bypasses RLS via service-role; everything below this line becomes irrelevant if Studio remains exposed.
- **F-2a (`pg_stat_statements` anonymous-readable) — HIGH.** Information disclosure only; no PII leaks (Postgres role-based privilege check + canonicalized parameters).
- **F-2 (Kong gateway exposed) — HIGH.** Schema + signup + content reachable by anonymous registrants. RLS holds for the user/PII tables.
- **A-1 (anonymous→guardian auto-classification) — MEDIUM.** New finding. Impacts consent integrity for a kids'-tech product.
- **A-2 (content reachable by anon registrants) — LOW.** May be intentional.
- **A-3 (`is_super_admin()` callable) — INFO.** No action needed.

## Files saved

- `raw/anon-signup-response.txt` — GoTrue signup response (contains test JWT)
- `raw/rest-swagger-authed.json` — OpenAPI spec as visible to authenticated role (262 KB)
- `raw/count-authed/results.json` — full count-anon results under authenticated JWT
- `raw/profile-self.json`, `raw/guardian-settings-self.json`, `raw/v_user_chapter_progress-row0.json` — three 1-row reads (our own session data)
