# Unomundi Supabase database schema (extracted)

**Source:** `GET http://dev-supabase.unomundi.com:8000/rest/v1/` (PostgREST OpenAPI/Swagger 2.0).
**Captured:** 2026-05-05 during the authorized engagement.
**Database:** `standard public schema` on Postgres `14.5`
**Surface:** `0.0.0.0:3000` (Kong-routed; nginx-fronted at `dev-supabase.unomundi.com:443`)
**Tables/views:** 59  •  **REST paths:** 99

This document is a metadata-only extract. **No row data was queried.** The schema itself was published by the application's PostgREST instance because the Kong gateway is reachable from the public internet (see engagement finding F-2). PostgREST emits this Swagger from the live Postgres catalog, so column names, types, defaults, primary keys, and foreign keys are authoritative.

## How to read this

- Each table section lists columns, their PostgreSQL types, default values, primary keys, and foreign keys.
- Foreign keys are annotated `→ table.col`. Inbound references show which other tables point at this one.
- The 99 REST paths follow PostgREST's standard pattern: `GET /<table>` (list/filter), `POST /<table>` (insert), `PATCH /<table>` (update), `DELETE /<table>` (delete) — actually executable depends on Postgres RLS policies.
- Tables prefixed with `v_` are views.

## Schema at a glance

### Children (Explorer) data

- **`explorer_profiles`** (8 cols) — Explorer-specific profile fields. 1:1 with profiles (role=explorer).
- **`explorer_settings`** (15 cols) — Explorer settings managed mostly by guardian; explorer can edit only allowed self subset via edge functions.
- **`explorer_screen_time_daily_usage`** (4 cols) — *no description*
- **`explorer_screen_time_state`** (4 cols) — *no description*

### Guardian (parent) data

- **`guardian_explorer_relations`** (4 cols) — Many-to-many relation between guardians and explorers.
- **`guardian_settings`** (7 cols) — Guardian self settings: faith/cultural context and guardian-side AI guide controls.
- **`guardian_action_logs`** (6 cols) — Append-only action logs for guardian-driven explorer settings/profile changes.

### Identity & consent

- **`profiles`** (11 cols) — Unified profile table for all roles. id = auth.users.id
- **`consent_records`** (7 cols) — Guardian acceptance of a legal document version. One acceptance applies to all linked children. Type/version come from legal_document_versions.
- **`legal_document_versions`** (6 cols) — Versions of legal documents. Exactly one row per document_type has active = true. Append-only: only INSERT allowed; trigger deactivates previous when inserting new active.

### Learning content (catalogue)

- **`tracks`** (8 cols) — *no description*
- **`track_translations`** (3 cols) — *no description*
- **`track_types`** (6 cols) — *no description*
- **`track_type_translations`** (3 cols) — *no description*
- **`chapters`** (9 cols) — *no description*
- **`chapter_translations`** (3 cols) — *no description*
- **`chapter_types`** (5 cols) — *no description*
- **`chapter_type_translations`** (3 cols) — *no description*
- **`lessons`** (7 cols) — *no description*
- **`lesson_translations`** (3 cols) — *no description*
- **`lesson_types`** (4 cols) — *no description*
- **`lesson_type_translations`** (3 cols) — *no description*
- **`video_lessons`** (4 cols) — Subtype table for lessons of kind video/campfire.
- **`video_lesson_audio_tracks`** (3 cols) — *no description*
- **`calluna_lessons`** (2 cols) — Subtype table for lessons of kind call_una.
- **`calluna_lesson_translations`** (4 cols) — *no description*
- **`gallery_lessons`** (1 cols) — Subtype table for lessons of kind gallery_walk.
- **`gallery_screens`** (4 cols) — *no description*
- **`gallery_walk_types`** (2 cols) — *no description*
- **`gallery_walk_type_translations`** (3 cols) — *no description*
- **`gallery_images`** (4 cols) — *no description*
- **`gallery_image_translations`** (4 cols) — *no description*
- **`quiz_lessons`** (1 cols) — Subtype table for lessons of kind quiz.
- **`quiz_questions`** (3 cols) — *no description*
- **`quiz_question_translations`** (7 cols) — *no description*
- **`quiz_answers`** (4 cols) — *no description*
- **`quiz_answer_translations`** (4 cols) — *no description*
- **`countries`** (6 cols) — *no description*
- **`country_translations`** (3 cols) — *no description*
- **`locales`** (2 cols) — *no description*
- **`localizations`** (5 cols) — *no description*

### Progress & engagement

- **`chapter_progresses`** (8 cols) — ERD-aligned chapter progress table linked to profiles (subject user).
- **`lesson_progresses`** (8 cols) — ERD-aligned lesson progress table linked to profiles (subject user).
- **`lesson_ratings`** (6 cols) — Immutable user ratings for lessons. Business logic currently allows only video lessons.
- **`user_content_cursor`** (6 cols) — Per-user continue-explore cursor storing last visited country/track/chapter context.
- **`v_user_chapter_progress`** (14 cols) — *no description*
- **`v_user_lesson_progress`** (21 cols) — *no description*
- **`v_user_track_progress`** (8 cols) — *no description*

### Telemetry & analytics

- **`analytics_events`** (6 cols) — Append-only analytics/audit events. Clients may insert only self signup/login. Readable only by admin role.
- **`una_session_daily_metrics`** (9 cols) — *no description*

### Moderation / reports

- **`report_types`** (4 cols) — *no description*
- **`report_type_translations`** (6 cols) — *no description*
- **`video_lesson_reports`** (7 cols) — *no description*

### Content-import jobs

- **`video_import_jobs`** (13 cols) — *no description*
- **`gallery_import_jobs`** (9 cols) — *no description*
- **`quiz_import_jobs`** (8 cols) — *no description*

### Database internals (should NOT be in public schema)

- **`pg_stat_statements`** (43 cols) — *no description*
- **`pg_stat_statements_info`** (2 cols) — *no description*
- **`schema_migrations`** (1 cols) — *no description*

## Privacy-sensitive data summary

This is a children's-app database (target audience ages 6-12). The following tables hold or imply personal data subject to **GDPR Art. 8 / GDPR-K / Dutch AVG / COPPA**:

- **`profiles`** — Identity rows for every user (children + parents). Likely contains email, name, role, locale, auth identifiers.
- **`explorer_profiles`** — Per-child profile: age, country, AI-features toggle, nickname. Linked 1:1 to `profiles` by `explorer_id`. Direct `age` storage of minors is itself a sensitive design choice.
- **`explorer_settings`** — Child-level preferences/settings.
- **`explorer_screen_time_daily_usage`** — Per-child screen-time usage records, daily granularity.
- **`explorer_screen_time_state`** — Current screen-time state per child.
- **`guardian_explorer_relations`** — Maps each parent (guardian) to their child(ren). Disclosure of this single table reveals the family graph.
- **`guardian_settings`** — Parent preferences.
- **`guardian_action_logs`** — Log of guardian actions (parental decisions, consent changes, screen-time overrides). Useful for audit, dangerous if leaked.
- **`consent_records`** — Records of consent given (almost certainly parental consent for minors). Critical legal artefact.
- **`analytics_events`** — Per-event behavioral telemetry. Per Art. 22 GDPR considerations may apply if the events drive personalization to minors.
- **`una_session_daily_metrics`** — Per-session daily aggregations (likely AI-tutor `Una` interactions).
- **`video_lesson_reports`** — Abuse / complaint reports about lesson content. Reporter identity may be a child or parent.
- **`lesson_ratings`** — Child-submitted ratings.
- **`lesson_progresses`** — Child progress per lesson.
- **`chapter_progresses`** — Child progress per chapter.
- **`user_content_cursor`** — Last-position-watched style data per user.
- **`pg_stat_statements`** — Postgres statement-level execution stats. **Should NEVER be in the `public` schema** because it contains query text and parameter samples — effectively a query log. If selectable by the anon role this leaks every query the app has run, including parameters that may include user IDs and emails.

## Foreign-key topology (most-pointed-at tables)

Tables ranked by inbound foreign-key references (a proxy for centrality of the data model):

| Table | Inbound FKs | Comes from |
|---|---:|---|
| `profiles` | 15 | `consent_records.actor_id`, `guardian_action_logs.explorer_id`, `guardian_action_logs.guardian_id`, `explorer_profiles.explorer_id`, `chapter_progresses.profile_id`, … +10 |
| `locales` | 13 | `gallery_walk_type_translations.locale_code`, `gallery_image_translations.locale_code`, `chapter_type_translations.locale_code`, `lesson_translations.locale_code`, `chapter_translations.locale_code`, … +8 |
| `lessons` | 10 | `quiz_import_jobs.lesson_id`, `lesson_translations.lesson_id`, `gallery_import_jobs.lesson_id`, `lesson_progresses.lesson_id`, `video_lessons.lesson_id`, … +5 |
| `countries` | 6 | `v_user_chapter_progress.country_cc`, `v_user_lesson_progress.country_cc`, `v_user_track_progress.country_cc`, `tracks.country_cc`, `user_content_cursor.country_cc`, … +1 |
| `track_types` | 6 | `v_user_chapter_progress.track_type_id`, `v_user_lesson_progress.track_type_id`, `chapter_types.track_type_id`, `v_user_track_progress.track_type_id`, `tracks.track_type_id`, … +1 |
| `chapter_types` | 5 | `v_user_chapter_progress.chapter_type_id`, `v_user_lesson_progress.chapter_type_id`, `chapter_type_translations.chapter_type_id`, `chapters.chapter_type_id`, `lesson_types.chapter_type_id` |
| `chapters` | 4 | `lessons.chapter_id`, `chapter_translations.chapter_id`, `chapter_progresses.chapter_id`, `user_content_cursor.chapter_id` |
| `tracks` | 3 | `track_translations.track_id`, `chapters.track_id`, `user_content_cursor.track_id` |
| `lesson_types` | 2 | `lessons.lesson_type_id`, `lesson_type_translations.lesson_type_id` |
| `gallery_walk_types` | 2 | `gallery_walk_type_translations.gallery_walk_type_id`, `gallery_screens.gallery_walk_type_id` |
| `video_lessons` | 2 | `video_lesson_audio_tracks.lesson_id`, `video_lesson_reports.video_lesson_id` |
| `report_types` | 2 | `report_type_translations.report_type_id`, `video_lesson_reports.report_type_id` |
| `quiz_questions` | 2 | `quiz_answers.quiz_question_id`, `quiz_question_translations.quiz_question_id` |
| `gallery_images` | 1 | `gallery_image_translations.gallery_image_id` |
| `legal_document_versions` | 1 | `consent_records.legal_document_version_id` |

---

## Tables (alphabetical)

### `analytics_events`

_Append-only analytics/audit events. Clients may insert only self signup/login. Readable only by admin role._

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `id` | `uuid` | yes | `gen_random_uuid()` | PK |
| `event_type` | `text` | yes | `` |  |
| `actor_role` | `text` | yes | `` |  |
| `actor_id` | `uuid` |  | `` |  |
| `created_at` | `timestamp with time zone` | yes | `now()` |  |
| `metadata` | `jsonb` |  | `` |  |

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `calluna_lesson_translations`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `lesson_id` | `uuid` | yes | `` | PK • FK → `calluna_lessons.lesson_id` |
| `locale_code` | `text` | yes | `` | PK • FK → `locales.code` |
| `starting_questions` | `text[]` | yes | `` |  |
| `backup_audio` | `text` |  | `` |  |

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `calluna_lessons`

_Subtype table for lessons of kind call_una._

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `lesson_id` | `uuid` | yes | `` | PK • FK → `lessons.id` |
| `max_duration_seconds` | `integer` | yes | `120` |  |

**Referenced by:** `calluna_lesson_translations.lesson_id`

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `chapter_progresses`

_ERD-aligned chapter progress table linked to profiles (subject user)._

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `id` | `uuid` | yes | `gen_random_uuid()` | PK |
| `profile_id` | `uuid` | yes | `` | FK → `profiles.id` |
| `chapter_id` | `uuid` | yes | `` | FK → `chapters.id` |
| `completed` | `boolean` | yes | `False` |  |
| `badge_awarded` | `boolean` | yes | `False` |  |
| `completed_at` | `timestamp with time zone` |  | `` |  |
| `created_at` | `timestamp with time zone` | yes | `now()` |  |
| `updated_at` | `timestamp with time zone` | yes | `now()` |  |

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `chapter_translations`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `chapter_id` | `uuid` | yes | `` | PK • FK → `chapters.id` |
| `locale_code` | `text` | yes | `` | PK • FK → `locales.code` |
| `title` | `text` | yes | `` |  |

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `chapter_type_translations`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `chapter_type_id` | `integer` | yes | `` | PK • FK → `chapter_types.id` |
| `locale_code` | `text` | yes | `` | PK • FK → `locales.code` |
| `label` | `text` | yes | `` |  |

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `chapter_types`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `id` | `integer` | yes | `` | PK |
| `track_type_id` | `integer` | yes | `` | FK → `track_types.id` |
| `chapter_number` | `integer` | yes | `` |  |
| `icon` | `text` |  | `` |  |
| `background` | `text` |  | `` |  |

**Referenced by:** `v_user_chapter_progress.chapter_type_id`, `v_user_lesson_progress.chapter_type_id`, `chapter_type_translations.chapter_type_id`, `chapters.chapter_type_id`, `lesson_types.chapter_type_id`

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `chapters`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `id` | `uuid` | yes | `gen_random_uuid()` | PK |
| `public_id` | `text` | yes | `` |  |
| `track_id` | `uuid` | yes | `` | FK → `tracks.id` |
| `chapter_type_id` | `integer` | yes | `` | FK → `chapter_types.id` |
| `reward_badge` | `text` |  | `` |  |
| `status` | `public.content_status` | yes | `draft` | enum: `draft`, `published`, `hidden` |
| `created_at` | `timestamp with time zone` | yes | `now()` |  |
| `updated_at` | `timestamp with time zone` | yes | `now()` |  |
| `reward_badge_off` | `text` |  | `` |  |

**Referenced by:** `lessons.chapter_id`, `chapter_translations.chapter_id`, `chapter_progresses.chapter_id`, `user_content_cursor.chapter_id`

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `consent_records`

_Guardian acceptance of a legal document version. One acceptance applies to all linked children. Type/version come from legal_document_versions._

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `id` | `uuid` | yes | `gen_random_uuid()` | PK |
| `actor_id` | `uuid` | yes | `` | FK → `profiles.id` |
| `legal_document_version_id` | `uuid` | yes | `` | FK → `legal_document_versions.id` |
| `accepted_at` | `timestamp with time zone` | yes | `now()` |  |
| `ip` | `text` |  | `` |  |
| `os` | `text` |  | `` |  |
| `browser` | `text` |  | `` |  |

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `countries`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `cc_code` | `text` | yes | `` | PK |
| `background_image` | `text` |  | `` |  |
| `paid_tier_only` | `boolean` | yes | `False` |  |
| `status` | `public.content_status` | yes | `draft` | enum: `draft`, `published`, `hidden` |
| `created_at` | `timestamp with time zone` | yes | `now()` |  |
| `updated_at` | `timestamp with time zone` | yes | `now()` |  |

**Referenced by:** `v_user_chapter_progress.country_cc`, `v_user_lesson_progress.country_cc`, `v_user_track_progress.country_cc`, `tracks.country_cc`, `user_content_cursor.country_cc`, `country_translations.country_cc`

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `country_translations`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `country_cc` | `text` | yes | `` | PK • FK → `countries.cc_code` |
| `locale_code` | `text` | yes | `` | PK • FK → `locales.code` |
| `name` | `text` | yes | `` |  |

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `explorer_profiles`

_Explorer-specific profile fields. 1:1 with profiles (role=explorer)._

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `explorer_id` | `uuid` | yes | `` | PK • FK → `profiles.id` |
| `age` | `integer` |  | `` |  |
| `country_iso2` | `character` |  | `` | max 2 |
| `ai_features_enabled` | `boolean` | yes | `False` |  |
| `created_at` | `timestamp with time zone` | yes | `now()` |  |
| `updated_at` | `timestamp with time zone` | yes | `now()` |  |
| `deleted_at` | `timestamp with time zone` |  | `` |  |
| `nickname` | `text` |  | `` |  |

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `explorer_screen_time_daily_usage`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `explorer_id` | `uuid` | yes | `` | PK • FK → `profiles.id` |
| `day` | `date` | yes | `` | PK |
| `minutes_used_today` | `integer` | yes | `0` |  |
| `updated_at` | `timestamp with time zone` | yes | `now()` |  |

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `explorer_screen_time_state`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `explorer_id` | `uuid` | yes | `` | PK • FK → `profiles.id` |
| `current_day` | `date` | yes | `` |  |
| `last_client_now` | `timestamp with time zone` |  | `` |  |
| `updated_at` | `timestamp with time zone` | yes | `now()` |  |

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `explorer_settings`

_Explorer settings managed mostly by guardian; explorer can edit only allowed self subset via edge functions._

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `explorer_id` | `uuid` | yes | `` | PK • FK → `profiles.id` |
| `notifications` | `jsonb` | yes | `` |  |
| `controls` | `jsonb` | yes | `` |  |
| `learning_level_mode` | `text` | yes | `auto` |  |
| `learning_level_detail` | `jsonb` | yes | `` |  |
| `guidance_style` | `jsonb` | yes | `` |  |
| `faith_text` | `text` |  | `` |  |
| `cultural_context_countries` | `text[]` | yes | `` |  |
| `app_preferences` | `jsonb` | yes | `` |  |
| `activity_visibility` | `jsonb` | yes | `` |  |
| `privacy_data` | `jsonb` | yes | `` |  |
| `created_at` | `timestamp with time zone` | yes | `now()` |  |
| `updated_at` | `timestamp with time zone` | yes | `now()` |  |
| `timezone_offset_minutes` | `integer` |  | `` |  |
| `app_preferences_guardian_locks` | `jsonb` | yes | `` |  |

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `gallery_image_translations`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `gallery_image_id` | `integer` | yes | `` | PK • FK → `gallery_images.id` |
| `locale_code` | `text` | yes | `` | PK • FK → `locales.code` |
| `affirmation_text` | `text` | yes | `` |  |
| `affirmation_audio` | `text` |  | `` |  |

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `gallery_images`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `id` | `integer` | yes | `` | PK |
| `gallery_screen_id` | `integer` | yes | `` | FK → `gallery_screens.id` |
| `image` | `text` | yes | `` |  |
| `sort_order` | `integer` | yes | `0` |  |

**Referenced by:** `gallery_image_translations.gallery_image_id`

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `gallery_import_jobs`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `id` | `uuid` | yes | `gen_random_uuid()` | PK |
| `lesson_id` | `uuid` | yes | `` | FK → `lessons.id` |
| `status` | `text` | yes | `processing` |  |
| `error_message` | `text` |  | `` |  |
| `storyboard_url` | `text` | yes | `` |  |
| `voice_over_url` | `text` | yes | `` |  |
| `images_url` | `text` | yes | `` |  |
| `created_at` | `timestamp with time zone` | yes | `now()` |  |
| `updated_at` | `timestamp with time zone` | yes | `now()` |  |

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `gallery_lessons`

_Subtype table for lessons of kind gallery_walk._

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `lesson_id` | `uuid` | yes | `` | PK • FK → `lessons.id` |

**Referenced by:** `gallery_screens.lesson_id`

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `gallery_screens`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `id` | `integer` | yes | `` | PK |
| `lesson_id` | `uuid` | yes | `` | FK → `gallery_lessons.lesson_id` |
| `gallery_walk_type_id` | `integer` | yes | `` | FK → `gallery_walk_types.id` |
| `sort_order` | `integer` | yes | `0` |  |

**Referenced by:** `gallery_images.gallery_screen_id`

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `gallery_walk_type_translations`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `gallery_walk_type_id` | `integer` | yes | `` | PK • FK → `gallery_walk_types.id` |
| `locale_code` | `text` | yes | `` | PK • FK → `locales.code` |
| `question` | `text` | yes | `` |  |

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `gallery_walk_types`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `id` | `integer` | yes | `` | PK |
| `type` | `text` | yes | `` |  |

**Referenced by:** `gallery_walk_type_translations.gallery_walk_type_id`, `gallery_screens.gallery_walk_type_id`

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `guardian_action_logs`

_Append-only action logs for guardian-driven explorer settings/profile changes._

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `id` | `uuid` | yes | `gen_random_uuid()` | PK |
| `explorer_id` | `uuid` | yes | `` | FK → `profiles.id` |
| `guardian_id` | `uuid` | yes | `` | FK → `profiles.id` |
| `action_type` | `text` | yes | `` |  |
| `action_payload` | `jsonb` | yes | `` |  |
| `created_at` | `timestamp with time zone` | yes | `now()` |  |

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `guardian_explorer_relations`

_Many-to-many relation between guardians and explorers._

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `guardian_id` | `uuid` | yes | `` | PK • FK → `profiles.id` |
| `explorer_id` | `uuid` | yes | `` | PK • FK → `profiles.id` |
| `created_at` | `timestamp with time zone` | yes | `now()` |  |
| `relationship` | `text` |  | `` | Guardian-defined free-text relation label for this specific explorer link (e.g., Daddy, Mommy). |

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `guardian_settings`

_Guardian self settings: faith/cultural context and guardian-side AI guide controls._

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `guardian_id` | `uuid` | yes | `` | PK • FK → `profiles.id` |
| `faith_text` | `text` |  | `` |  |
| `country_iso2` | `character` |  | `` | max 2 |
| `cultural_context_countries` | `text[]` | yes | `` |  |
| `controls` | `jsonb` | yes | `` |  |
| `created_at` | `timestamp with time zone` | yes | `now()` |  |
| `updated_at` | `timestamp with time zone` | yes | `now()` |  |

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `legal_document_versions`

_Versions of legal documents. Exactly one row per document_type has active = true. Append-only: only INSERT allowed; trigger deactivates previous when inserting new active._

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `id` | `uuid` | yes | `gen_random_uuid()` | PK |
| `document_type` | `public.legal_document_type` | yes | `` | enum: `guardian_consent`, `terms_of_service`, `privacy_policy`, `parental_consent`, `ai_terms` |
| `active` | `boolean` | yes | `False` |  |
| `created_at` | `timestamp with time zone` | yes | `now()` |  |
| `content` | `text` | yes | `` | Rich text/HTML content snapshot for this exact legal document version. |
| `version_number` | `integer` | yes | `` | Auto-incremented numeric version per document_type. |

**Referenced by:** `consent_records.legal_document_version_id`

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `lesson_progresses`

_ERD-aligned lesson progress table linked to profiles (subject user)._

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `id` | `uuid` | yes | `gen_random_uuid()` | PK |
| `profile_id` | `uuid` | yes | `` | FK → `profiles.id` |
| `lesson_id` | `uuid` | yes | `` | FK → `lessons.id` |
| `status` | `text` | yes | `started` |  |
| `quiz_score` | `integer` |  | `` |  |
| `completed_at` | `timestamp with time zone` |  | `` |  |
| `created_at` | `timestamp with time zone` | yes | `now()` |  |
| `updated_at` | `timestamp with time zone` | yes | `now()` |  |

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `lesson_ratings`

_Immutable user ratings for lessons. Business logic currently allows only video lessons._

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `id` | `uuid` | yes | `gen_random_uuid()` | PK |
| `profile_id` | `uuid` | yes | `` | FK → `profiles.id` |
| `lesson_id` | `uuid` | yes | `` | FK → `lessons.id` |
| `rating` | `smallint` | yes | `` |  |
| `created_at` | `timestamp with time zone` | yes | `now()` |  |
| `updated_at` | `timestamp with time zone` | yes | `now()` |  |

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `lesson_translations`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `lesson_id` | `uuid` | yes | `` | PK • FK → `lessons.id` |
| `locale_code` | `text` | yes | `` | PK • FK → `locales.code` |
| `title_override` | `text` |  | `` |  |

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `lesson_type_translations`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `lesson_type_id` | `integer` | yes | `` | PK • FK → `lesson_types.id` |
| `locale_code` | `text` | yes | `` | PK • FK → `locales.code` |
| `default_title` | `text` | yes | `` |  |

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `lesson_types`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `id` | `integer` | yes | `` | PK |
| `chapter_type_id` | `integer` | yes | `` | FK → `chapter_types.id` |
| `sort_order` | `integer` | yes | `0` |  |
| `kind` | `public.lesson_kind` | yes | `` | enum: `video`, `campfire`, `quiz`, `gallery_walk`, `call_una` |

**Referenced by:** `lessons.lesson_type_id`, `lesson_type_translations.lesson_type_id`

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `lessons`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `id` | `uuid` | yes | `gen_random_uuid()` | PK |
| `public_id` | `text` | yes | `` |  |
| `chapter_id` | `uuid` | yes | `` | FK → `chapters.id` |
| `lesson_type_id` | `integer` | yes | `` | FK → `lesson_types.id` |
| `status` | `public.content_status` | yes | `draft` | enum: `draft`, `published`, `hidden` |
| `created_at` | `timestamp with time zone` | yes | `now()` |  |
| `updated_at` | `timestamp with time zone` | yes | `now()` |  |

**Referenced by:** `quiz_import_jobs.lesson_id`, `lesson_translations.lesson_id`, `gallery_import_jobs.lesson_id`, `lesson_progresses.lesson_id`, `video_lessons.lesson_id`, `lesson_ratings.lesson_id`, `quiz_lessons.lesson_id`, `video_import_jobs.lesson_id`, `gallery_lessons.lesson_id`, `calluna_lessons.lesson_id`

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `locales`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `code` | `text` | yes | `` | PK |
| `label` | `text` | yes | `` |  |

**Referenced by:** `gallery_walk_type_translations.locale_code`, `gallery_image_translations.locale_code`, `chapter_type_translations.locale_code`, `lesson_translations.locale_code`, `chapter_translations.locale_code`, `video_lesson_audio_tracks.locale_code`, `calluna_lesson_translations.locale_code`, `quiz_answer_translations.locale_code`, `track_translations.locale_code`, `country_translations.locale_code`, `lesson_type_translations.locale_code`, `track_type_translations.locale_code`, `quiz_question_translations.locale_code`

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `localizations`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `id` | `uuid` | yes | `gen_random_uuid()` | PK |
| `locale` | `text` | yes | `` |  |
| `translations` | `jsonb` | yes | `` |  |
| `created_at` | `timestamp with time zone` | yes | `now()` |  |
| `updated_at` | `timestamp with time zone` | yes | `now()` |  |

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `pg_stat_statements`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `userid` | `oid` |  | `` |  |
| `dbid` | `oid` |  | `` |  |
| `toplevel` | `boolean` |  | `` |  |
| `queryid` | `bigint` |  | `` |  |
| `query` | `text` |  | `` |  |
| `plans` | `bigint` |  | `` |  |
| `total_plan_time` | `double precision` |  | `` |  |
| `min_plan_time` | `double precision` |  | `` |  |
| `max_plan_time` | `double precision` |  | `` |  |
| `mean_plan_time` | `double precision` |  | `` |  |
| `stddev_plan_time` | `double precision` |  | `` |  |
| `calls` | `bigint` |  | `` |  |
| `total_exec_time` | `double precision` |  | `` |  |
| `min_exec_time` | `double precision` |  | `` |  |
| `max_exec_time` | `double precision` |  | `` |  |
| `mean_exec_time` | `double precision` |  | `` |  |
| `stddev_exec_time` | `double precision` |  | `` |  |
| `rows` | `bigint` |  | `` |  |
| `shared_blks_hit` | `bigint` |  | `` |  |
| `shared_blks_read` | `bigint` |  | `` |  |
| `shared_blks_dirtied` | `bigint` |  | `` |  |
| `shared_blks_written` | `bigint` |  | `` |  |
| `local_blks_hit` | `bigint` |  | `` |  |
| `local_blks_read` | `bigint` |  | `` |  |
| `local_blks_dirtied` | `bigint` |  | `` |  |
| `local_blks_written` | `bigint` |  | `` |  |
| `temp_blks_read` | `bigint` |  | `` |  |
| `temp_blks_written` | `bigint` |  | `` |  |
| `blk_read_time` | `double precision` |  | `` |  |
| `blk_write_time` | `double precision` |  | `` |  |
| `temp_blk_read_time` | `double precision` |  | `` |  |
| `temp_blk_write_time` | `double precision` |  | `` |  |
| `wal_records` | `bigint` |  | `` |  |
| `wal_fpi` | `bigint` |  | `` |  |
| `wal_bytes` | `numeric` |  | `` |  |
| `jit_functions` | `bigint` |  | `` |  |
| `jit_generation_time` | `double precision` |  | `` |  |
| `jit_inlining_count` | `bigint` |  | `` |  |
| `jit_inlining_time` | `double precision` |  | `` |  |
| `jit_optimization_count` | `bigint` |  | `` |  |
| `jit_optimization_time` | `double precision` |  | `` |  |
| `jit_emission_count` | `bigint` |  | `` |  |
| `jit_emission_time` | `double precision` |  | `` |  |

**REST verbs available:** `GET`

### `pg_stat_statements_info`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `dealloc` | `bigint` |  | `` |  |
| `stats_reset` | `timestamp with time zone` |  | `` |  |

**REST verbs available:** `GET`

### `profiles`

_Unified profile table for all roles. id = auth.users.id_

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `id` | `uuid` | yes | `` | PK |
| `role` | `public.user_role` | yes | `` | enum: `guardian`, `explorer`, `admin`, `super_admin` |
| `status` | `public.account_status` | yes | `active` | enum: `active`, `suspended`, `deleted` |
| `display_name` | `text` |  | `` |  |
| `avatar_key` | `text` |  | `` |  |
| `created_at` | `timestamp with time zone` | yes | `now()` |  |
| `updated_at` | `timestamp with time zone` | yes | `now()` |  |
| `preferred_language` | `character` | yes | `'en'::bpchar` | max 2 • ISO 639-1 language code: en, nl, de. Default en. Used for UI localization. |
| `email` | `text` |  | `` |  |
| `has_active_subscription` | `boolean` | yes | `False` | Temporary subscription flag used for paid content access checks. |
| `is_adult` | `boolean` |  | `` | Optional guardian adult confirmation flag captured during legal consent acceptance. |

**Referenced by:** `consent_records.actor_id`, `guardian_action_logs.explorer_id`, `guardian_action_logs.guardian_id`, `explorer_profiles.explorer_id`, `chapter_progresses.profile_id`, `lesson_progresses.profile_id`, `explorer_settings.explorer_id`, `guardian_explorer_relations.guardian_id`, `guardian_explorer_relations.explorer_id`, `lesson_ratings.profile_id`, `explorer_screen_time_state.explorer_id`, `user_content_cursor.user_id`, `explorer_screen_time_daily_usage.explorer_id`, `guardian_settings.guardian_id`, `video_lesson_reports.profile_id`

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `quiz_answer_translations`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `quiz_answer_id` | `integer` | yes | `` | PK • FK → `quiz_answers.id` |
| `locale_code` | `text` | yes | `` | PK • FK → `locales.code` |
| `answer_text` | `text` | yes | `` |  |
| `answer_audio` | `text` |  | `` |  |

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `quiz_answers`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `id` | `integer` | yes | `` | PK |
| `quiz_question_id` | `integer` | yes | `` | FK → `quiz_questions.id` |
| `is_correct` | `boolean` | yes | `False` |  |
| `sort_order` | `integer` |  | `` |  |

**Referenced by:** `quiz_answer_translations.quiz_answer_id`

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `quiz_import_jobs`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `id` | `uuid` | yes | `gen_random_uuid()` | PK |
| `lesson_id` | `uuid` | yes | `` | FK → `lessons.id` |
| `status` | `text` | yes | `processing` |  |
| `error_message` | `text` |  | `` |  |
| `storyboard_url` | `text` | yes | `` |  |
| `voice_over_url` | `text` | yes | `` |  |
| `created_at` | `timestamp with time zone` | yes | `now()` |  |
| `updated_at` | `timestamp with time zone` | yes | `now()` |  |

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `quiz_lessons`

_Subtype table for lessons of kind quiz._

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `lesson_id` | `uuid` | yes | `` | PK • FK → `lessons.id` |

**Referenced by:** `quiz_questions.lesson_id`

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `quiz_question_translations`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `quiz_question_id` | `integer` | yes | `` | PK • FK → `quiz_questions.id` |
| `locale_code` | `text` | yes | `` | PK • FK → `locales.code` |
| `question_text` | `text` | yes | `` |  |
| `question_audio` | `text` |  | `` |  |
| `correct_affirmation_audio` | `text` |  | `` |  |
| `correct_affirmation_text` | `text` |  | `` |  |
| `wrong_answer_text` | `text` |  | `` |  |

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `quiz_questions`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `id` | `integer` | yes | `` | PK |
| `lesson_id` | `uuid` | yes | `` | FK → `quiz_lessons.lesson_id` |
| `sort_order` | `integer` | yes | `0` |  |

**Referenced by:** `quiz_answers.quiz_question_id`, `quiz_question_translations.quiz_question_id`

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `report_type_translations`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `id` | `uuid` | yes | `gen_random_uuid()` | PK |
| `report_type_id` | `uuid` | yes | `` | FK → `report_types.id` |
| `locale_code` | `text` | yes | `` |  |
| `label` | `text` | yes | `` |  |
| `created_at` | `timestamp with time zone` | yes | `now()` |  |
| `updated_at` | `timestamp with time zone` | yes | `now()` |  |

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `report_types`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `id` | `uuid` | yes | `gen_random_uuid()` | PK |
| `code` | `text` | yes | `` |  |
| `created_at` | `timestamp with time zone` | yes | `now()` |  |
| `updated_at` | `timestamp with time zone` | yes | `now()` |  |

**Referenced by:** `report_type_translations.report_type_id`, `video_lesson_reports.report_type_id`

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `schema_migrations`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `version` | `character varying` | yes | `` | PK • max 14 |

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `track_translations`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `track_id` | `uuid` | yes | `` | PK • FK → `tracks.id` |
| `locale_code` | `text` | yes | `` | PK • FK → `locales.code` |
| `title` | `text` |  | `` |  |

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `track_type_translations`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `track_type_id` | `integer` | yes | `` | PK • FK → `track_types.id` |
| `locale_code` | `text` | yes | `` | PK • FK → `locales.code` |
| `label` | `text` | yes | `` |  |

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `track_types`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `id` | `integer` | yes | `` | PK |
| `sort_order` | `integer` | yes | `0` |  |
| `icon` | `text` |  | `` |  |
| `color` | `text` |  | `` |  |
| `background` | `text` |  | `` |  |
| `path` | `public.track_path` | yes | `explorer` | enum: `explorer`, `ambassador` |

**Referenced by:** `v_user_chapter_progress.track_type_id`, `v_user_lesson_progress.track_type_id`, `chapter_types.track_type_id`, `v_user_track_progress.track_type_id`, `tracks.track_type_id`, `track_type_translations.track_type_id`

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `tracks`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `id` | `uuid` | yes | `gen_random_uuid()` | PK |
| `public_id` | `text` | yes | `` |  |
| `country_cc` | `text` | yes | `` | FK → `countries.cc_code` |
| `track_type_id` | `integer` | yes | `` | FK → `track_types.id` |
| `reward_badge` | `text` |  | `` |  |
| `status` | `public.content_status` | yes | `draft` | enum: `draft`, `published`, `hidden` |
| `created_at` | `timestamp with time zone` | yes | `now()` |  |
| `updated_at` | `timestamp with time zone` | yes | `now()` |  |

**Referenced by:** `track_translations.track_id`, `chapters.track_id`, `user_content_cursor.track_id`

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `una_session_daily_metrics`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `day` | `timestamp with time zone` |  | `` |  |
| `country_cc` | `character` |  | `` | max 2 |
| `age_tier` | `text` |  | `` |  |
| `mode` | `public.una_session_mode` |  | `` | enum: `free`, `lesson` |
| `sessions_started` | `bigint` |  | `` |  |
| `sessions_terminal` | `bigint` |  | `` |  |
| `sessions_forced` | `bigint` |  | `` |  |
| `sessions_safety_escalated` | `bigint` |  | `` |  |
| `sessions_failed` | `bigint` |  | `` |  |

**REST verbs available:** `GET`

### `user_content_cursor`

_Per-user continue-explore cursor storing last visited country/track/chapter context._

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `user_id` | `uuid` | yes | `` | PK • FK → `profiles.id` |
| `country_cc` | `text` | yes | `` | FK → `countries.cc_code` |
| `track_id` | `uuid` |  | `` | FK → `tracks.id` |
| `chapter_id` | `uuid` |  | `` | FK → `chapters.id` |
| `created_at` | `timestamp with time zone` | yes | `now()` |  |
| `updated_at` | `timestamp with time zone` | yes | `now()` |  |

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `v_user_chapter_progress`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `subject_user_id` | `uuid` |  | `` |  |
| `country_cc` | `text` |  | `` | FK → `countries.cc_code` |
| `track_id` | `uuid` |  | `` | PK |
| `track_type_id` | `integer` |  | `` | FK → `track_types.id` |
| `chapter_id` | `uuid` |  | `` | PK |
| `chapter_type_id` | `integer` |  | `` | FK → `chapter_types.id` |
| `chapter_number` | `integer` |  | `` |  |
| `lessons_total` | `integer` |  | `` |  |
| `lessons_completed` | `integer` |  | `` |  |
| `is_completed` | `boolean` |  | `` |  |
| `badge_awarded` | `boolean` |  | `` |  |
| `completed_at` | `timestamp with time zone` |  | `` |  |
| `is_available` | `boolean` |  | `` |  |
| `next_available_lesson_id` | `uuid` |  | `` |  |

**REST verbs available:** `GET`

### `v_user_lesson_progress`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `subject_user_id` | `uuid` |  | `` |  |
| `country_cc` | `text` |  | `` | FK → `countries.cc_code` |
| `track_id` | `uuid` |  | `` | PK |
| `track_type_id` | `integer` |  | `` | FK → `track_types.id` |
| `chapter_id` | `uuid` |  | `` | PK |
| `chapter_type_id` | `integer` |  | `` | FK → `chapter_types.id` |
| `chapter_number` | `integer` |  | `` |  |
| `lesson_id` | `uuid` |  | `` | PK |
| `lesson_public_id` | `text` |  | `` |  |
| `lesson_type_id` | `integer` |  | `` | PK |
| `sort_order` | `integer` |  | `` |  |
| `kind` | `public.lesson_kind` |  | `` | enum: `video`, `campfire`, `quiz`, `gallery_walk`, `call_una` |
| `started_at` | `timestamp with time zone` |  | `` |  |
| `completed_at` | `timestamp with time zone` |  | `` |  |
| `quiz_score` | `integer` |  | `` |  |
| `status` | `text` |  | `` |  |
| `is_started` | `boolean` |  | `` |  |
| `is_completed` | `boolean` |  | `` |  |
| `is_available` | `boolean` |  | `` |  |
| `is_current` | `boolean` |  | `` |  |
| `rating` | `smallint` |  | `` |  |

**REST verbs available:** `GET`

### `v_user_track_progress`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `subject_user_id` | `uuid` |  | `` |  |
| `country_cc` | `text` |  | `` | FK → `countries.cc_code` |
| `track_id` | `uuid` |  | `` | PK |
| `track_type_id` | `integer` |  | `` | FK → `track_types.id` |
| `chapters_total` | `integer` |  | `` |  |
| `chapters_completed` | `integer` |  | `` |  |
| `is_completed` | `boolean` |  | `` |  |
| `is_available` | `boolean` |  | `` |  |

**REST verbs available:** `GET`

### `video_import_jobs`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `id` | `uuid` | yes | `gen_random_uuid()` | PK |
| `lesson_id` | `uuid` | yes | `` | FK → `lessons.id` |
| `provider` | `text` | yes | `mux` |  |
| `provider_asset_id` | `text` | yes | `` |  |
| `provider_playback_id` | `text` |  | `` |  |
| `status` | `text` | yes | `processing` |  |
| `error_message` | `text` |  | `` |  |
| `source_url` | `text` |  | `` |  |
| `created_at` | `timestamp with time zone` | yes | `now()` |  |
| `updated_at` | `timestamp with time zone` | yes | `now()` |  |
| `temp_storage_bucket` | `text` |  | `` |  |
| `temp_storage_object_key` | `text` |  | `` |  |
| `temp_storage_deleted_at` | `timestamp with time zone` |  | `` |  |

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `video_lesson_audio_tracks`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `lesson_id` | `uuid` | yes | `` | PK • FK → `video_lessons.lesson_id` |
| `locale_code` | `text` | yes | `` | PK • FK → `locales.code` |
| `mux_audio_track_id` | `text` | yes | `` |  |

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `video_lesson_reports`

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `id` | `uuid` | yes | `gen_random_uuid()` | PK |
| `profile_id` | `uuid` | yes | `` | FK → `profiles.id` |
| `video_lesson_id` | `uuid` | yes | `` | FK → `video_lessons.lesson_id` |
| `report_type_id` | `uuid` | yes | `` | FK → `report_types.id` |
| `report_code_snapshot` | `text` | yes | `` |  |
| `reported_at_seconds` | `integer` |  | `` |  |
| `created_at` | `timestamp with time zone` | yes | `now()` |  |

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

### `video_lessons`

_Subtype table for lessons of kind video/campfire._

| Column | Type | Required | Default | Notes |
|---|---|---|---|---|
| `lesson_id` | `uuid` | yes | `` | PK • FK → `lessons.id` |
| `video_file` | `text` | yes | `` |  |
| `created_at` | `timestamp with time zone` | yes | `now()` |  |
| `updated_at` | `timestamp with time zone` | yes | `now()` |  |

**Referenced by:** `video_lesson_audio_tracks.lesson_id`, `video_lesson_reports.video_lesson_id`

**REST verbs available:** `DELETE`, `GET`, `PATCH`, `POST`

---

## REST path inventory

PostgREST emits one path per table for the collection (`/<table>`) and one per RPC. Here are the 99 routes the gateway exposed:

```
  GET                          /
  DELETE,GET,PATCH,POST        /analytics_events
  DELETE,GET,PATCH,POST        /calluna_lesson_translations
  DELETE,GET,PATCH,POST        /calluna_lessons
  DELETE,GET,PATCH,POST        /chapter_progresses
  DELETE,GET,PATCH,POST        /chapter_translations
  DELETE,GET,PATCH,POST        /chapter_type_translations
  DELETE,GET,PATCH,POST        /chapter_types
  DELETE,GET,PATCH,POST        /chapters
  DELETE,GET,PATCH,POST        /consent_records
  DELETE,GET,PATCH,POST        /countries
  DELETE,GET,PATCH,POST        /country_translations
  DELETE,GET,PATCH,POST        /explorer_profiles
  DELETE,GET,PATCH,POST        /explorer_screen_time_daily_usage
  DELETE,GET,PATCH,POST        /explorer_screen_time_state
  DELETE,GET,PATCH,POST        /explorer_settings
  DELETE,GET,PATCH,POST        /gallery_image_translations
  DELETE,GET,PATCH,POST        /gallery_images
  DELETE,GET,PATCH,POST        /gallery_import_jobs
  DELETE,GET,PATCH,POST        /gallery_lessons
  DELETE,GET,PATCH,POST        /gallery_screens
  DELETE,GET,PATCH,POST        /gallery_walk_type_translations
  DELETE,GET,PATCH,POST        /gallery_walk_types
  DELETE,GET,PATCH,POST        /guardian_action_logs
  DELETE,GET,PATCH,POST        /guardian_explorer_relations
  DELETE,GET,PATCH,POST        /guardian_settings
  DELETE,GET,PATCH,POST        /legal_document_versions
  DELETE,GET,PATCH,POST        /lesson_progresses
  DELETE,GET,PATCH,POST        /lesson_ratings
  DELETE,GET,PATCH,POST        /lesson_translations
  DELETE,GET,PATCH,POST        /lesson_type_translations
  DELETE,GET,PATCH,POST        /lesson_types
  DELETE,GET,PATCH,POST        /lessons
  DELETE,GET,PATCH,POST        /locales
  DELETE,GET,PATCH,POST        /localizations
  GET                          /pg_stat_statements
  GET                          /pg_stat_statements_info
  DELETE,GET,PATCH,POST        /profiles
  DELETE,GET,PATCH,POST        /quiz_answer_translations
  DELETE,GET,PATCH,POST        /quiz_answers
  DELETE,GET,PATCH,POST        /quiz_import_jobs
  DELETE,GET,PATCH,POST        /quiz_lessons
  DELETE,GET,PATCH,POST        /quiz_question_translations
  DELETE,GET,PATCH,POST        /quiz_questions
  DELETE,GET,PATCH,POST        /report_type_translations
  DELETE,GET,PATCH,POST        /report_types
  POST                         /rpc/admin_dashboard_cards_json
  POST                         /rpc/admin_list_app_users_page_json
  GET,POST                     /rpc/can_read_chapter_for_user
  GET,POST                     /rpc/can_read_chapter_type_for_user
  GET,POST                     /rpc/can_read_country_for_user
  GET,POST                     /rpc/can_read_gallery_walk_type_for_user
  GET,POST                     /rpc/can_read_lesson_for_user
  GET,POST                     /rpc/can_read_lesson_type_for_user
  GET,POST                     /rpc/can_read_progress_for_user
  GET,POST                     /rpc/can_read_track_for_user
  GET,POST                     /rpc/can_read_track_type_for_user
  GET,POST                     /rpc/can_write_progress_for_user
  POST                         /rpc/complete_lesson
  POST                         /rpc/content_create_lesson_bundle
  POST                         /rpc/content_update_lesson_bundle
  POST                         /rpc/custom_access_token_hook
  GET,POST                     /rpc/enforce_lesson_kind_for_subtable
  POST                         /rpc/explorer_register_finalize
  POST                         /rpc/guardian_apply_explorer_settings_patch
  POST                         /rpc/guardian_link_explorer_by_pairing_code
  POST                         /rpc/has_lesson_rating
  GET,POST                     /rpc/is_content_admin
  GET,POST                     /rpc/is_content_admin_for_user
  GET,POST                     /rpc/is_lesson_available_for_user
  GET,POST                     /rpc/is_super_admin
  GET,POST                     /rpc/is_valid_user_content_cursor
  POST                         /rpc/issue_explorer_pairing_code_tx
  GET,POST                     /rpc/next_lesson_id_for_lesson
  POST                         /rpc/pg_stat_statements
  POST                         /rpc/pg_stat_statements_info
  POST                         /rpc/pg_stat_statements_reset
  GET,POST                     /rpc/previous_lesson_id_for_lesson
  POST                         /rpc/purge_old_auth_and_guardian_logs
  POST                         /rpc/rate_lesson
  POST                         /rpc/resolve_import_lesson_context
  POST                         /rpc/rls_auto_enable
  GET,POST                     /rpc/screen_time_allows_lesson_completion
  POST                         /rpc/screen_time_apply_day_deltas
  POST                         /rpc/start_lesson
  DELETE,GET,PATCH,POST        /schema_migrations
  DELETE,GET,PATCH,POST        /track_translations
  DELETE,GET,PATCH,POST        /track_type_translations
  DELETE,GET,PATCH,POST        /track_types
  DELETE,GET,PATCH,POST        /tracks
  GET                          /una_session_daily_metrics
  DELETE,GET,PATCH,POST        /user_content_cursor
  GET                          /v_user_chapter_progress
  GET                          /v_user_lesson_progress
  GET                          /v_user_track_progress
  DELETE,GET,PATCH,POST        /video_import_jobs
  DELETE,GET,PATCH,POST        /video_lesson_audio_tracks
  DELETE,GET,PATCH,POST        /video_lesson_reports
  DELETE,GET,PATCH,POST        /video_lessons
```

---

## Caveats

- Whether each `GET/POST/PATCH/DELETE` is *executable* by an anonymous attacker depends on the Postgres Row-Level-Security (RLS) policies, which are not visible from outside the database. The schema disclosure alone tells an attacker exactly which tables to probe.
- This document does not include row data. The engagement explicitly avoided querying any rows.
- Column comments and table descriptions are taken verbatim from the live PostgREST output (which sources them from `COMMENT ON …` in Postgres).