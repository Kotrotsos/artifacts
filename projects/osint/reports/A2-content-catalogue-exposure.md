# A-2 — Educational content catalogue readable by any anonymous registrant

**Engagement:** unomundi.com pentest, 2026-05-05.
**Severity:** **LOW** *(pending business confirmation; could be **MEDIUM** if the catalogue is intended to be paid IP)*.
**Status:** confirmed by row-count probe under an anonymous-authenticated JWT.

## Statement

The full educational content catalogue is readable by any user who registers anonymously through GoTrue's anonymous-signup endpoint. **Possibly intentional preview, possibly leakage of paid IP.** The asset owner needs to confirm the intended access model before this can be definitively classified as bug-or-feature.

## Evidence

After registering an anonymous user (no email, no password — `POST /auth/v1/signup` with empty body, enabled by `anonymous_users:true` in GoTrue settings) and using the resulting `authenticated`-role JWT, the following row counts were retrieved via PostgREST `HEAD /rest/v1/<table>?select=count` (no row data read):

| Table | Rows visible to authed-anon |
|---|---:|
| `chapters` | 585 |
| `tracks` | 585 |
| `lessons` | 350 |
| `video_lessons` | 232 |
| `gallery_lessons` | 54 |
| `gallery_screens` | 147 |
| `gallery_images` | 566 |
| `gallery_image_translations` | 566 |
| `quiz_lessons` | 64 |
| `quiz_questions` | 256 |
| `quiz_question_translations` | 256 |
| `quiz_answers` | 1,024 |
| `quiz_answer_translations` | 1,024 |
| `lesson_translations` | 55 |
| `lesson_types` | 8 |
| `chapter_types` / `track_types` / `chapter_type_translations` / `track_type_translations` | 3 each |
| `gallery_walk_types` / `gallery_walk_type_translations` | 3 each |
| `countries` / `country_translations` | 195 each |
| `locales` / `localizations` | 3 each |
| `legal_document_versions` | 6 |
| `report_types` / `report_type_translations` | 4 / 12 |
| `global_audio_assets` / `global_audio_asset_translations` | 5 / 52 |
| `global_image_assets` | 1 |
| `global_video_assets` / `global_video_asset_translations` | 3 / 1 |

**Total catalogue rows reachable:** approximately **6,500** across 30+ catalogue tables. This is the full content set — every chapter, every lesson, every quiz, every gallery image, every translation, every country reference, every global asset.

## Reproduction steps

```bash
ANON_KEY='<public anon JWT, already in admin-bundle.js>'
BASE='http://dev-supabase.unomundi.com:8000'

# 1. Register anonymously (no email, no password — enabled because anonymous_users:true)
curl -sS -X POST \
  -H "apikey: $ANON_KEY" \
  -H "Content-Type: application/json" \
  -d '{}' \
  "$BASE/auth/v1/signup" | jq -r '.access_token' > user_jwt.txt

# 2. Use the authenticated JWT to read the catalogue (example: lessons)
curl -sS \
  -H "apikey: $ANON_KEY" \
  -H "Authorization: Bearer $(cat user_jwt.txt)" \
  "$BASE/rest/v1/lessons?select=*&limit=10"
```

The signup costs the attacker nothing — no email verification, no captcha, no rate limit observed at the Kong layer. A scripted exfiltration of the entire catalogue completes in seconds.

## Why this is "possibly intentional, possibly a leak"

**Arguments it's intentional:**
- Pre-registration preview is a common growth pattern — letting users browse content before committing to a paid plan or full registration.
- The base catalogue (lessons, chapters, tracks, countries) is needed at the client side to render menus, even when no specific lesson is being consumed.
- The educational content is ultimately for children, and the public-facing brand explicitly says "guilt-free screen time" — possibly the team wants the content reachable to demonstrate quality.

**Arguments it's a leak:**
- The product is positioned as a subscription / freemium app (per Stripe-style language on the marketing site, though I haven't confirmed pricing). If revenue depends on people *paying* to access the lessons, exposing all 350 lessons to any anonymous registrant is paid-IP leakage.
- Competitors can scrape the entire content corpus, including translations across all locales, in under a minute.
- The lesson scripts and quiz questions represent the company's core creative work product. Their educational design is the differentiator. Leaving it readable behind a single empty-body POST is generous.
- Even if the catalogue *titles* are intended to be public, full text of `quiz_answer_translations` (1,024 rows) and full lesson scripts in `lesson_translations` (55 rows) probably aren't.

## What was *not* read

- No row contents were extracted from these tables. Only row counts.
- The actual lesson text, quiz questions, and translations remain unread on this side.

The screenshot evidence in F-1 (operator's Studio session) shows that the same content tables can be *read fully* via Studio. So the catalogue is exposed both via the open Kong path (anonymous-signup → authed reads) and via Studio (no auth at all). The two paths together mean the question of intent matters even less for risk assessment: even if anonymous-readable was intentional, Studio's exposure means a competitor doesn't even need to register.

## Remediation options (depending on intent)

### Option A — catalogue is intentionally public preview
If this is intended:
1. **Document it explicitly** in the API guide so future RLS audits don't accidentally lock it down.
2. **Disable anonymous signup anyway.** If you want public read of the catalogue, expose those specific tables to the `anon` role with `GRANT SELECT` and remove the need for users to register. Anonymous registration is a foothold for everything *else* downstream.
3. **Rate-limit catalogue reads** at Kong / Cloudflare to prevent bulk scraping.
4. **Consider serving lesson *previews* only** to unauthenticated users — title and first paragraph — and gate full content behind a paid-subscription claim in the JWT.

### Option B — catalogue should be gated
If the lessons / quizzes / translations are meant to be paid:
1. **Set `anonymous_users:false`** in GoTrue config. This single change removes the no-email-needed registration path.
2. **Tighten RLS** on catalogue tables. Replace any policy that grants `SELECT` to `authenticated` with one that checks a paid-subscription claim:
   ```sql
   CREATE POLICY "lessons_paid_only" ON public.lessons
     FOR SELECT TO authenticated
     USING (
       (auth.jwt() ->> 'app_metadata' ->> 'subscription_status')::text = 'active'
     );
   ```
3. **Audit which tables are truly content vs reference.** `countries` and `locales` are probably fine to expose. `lessons`, `quiz_*`, `gallery_*` arguably are not.

## Suggested follow-up question for the asset owner

> "Are `lessons`, `quiz_questions`, `quiz_answers`, `lesson_translations`, and the `gallery_*` tables intended to be readable by any user who completes anonymous registration, or is that a misconfiguration? Currently any visitor can register without email and pull the entire catalogue in under a minute."

Their answer determines whether this finding stays at LOW (intentional preview) or rises to MEDIUM (paid-IP leakage with quantifiable revenue impact).
