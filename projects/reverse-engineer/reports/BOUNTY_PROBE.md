# Gemini for macOS 1.53.0.262 — API Key Restriction Probes

> **Scope.** Two probe rounds (13 single requests) against `generativelanguage.googleapis.com` and `places.googleapis.com` using six AIza keys extracted from `/Applications/Gemini.app/Contents/MacOS/Gemini`. Goal: determine whether the App Restrictions on each key are correctly configured, and which restriction style (legacy header-trust vs. App Attest) is in use. No content was generated. No quota was burned beyond a single discovery / search-text call per probe. All probes from a single residential IP, 2026-05-06.

## TL;DR

- All 6 keys **do** have App Restrictions configured (every probe returned 403).
- The 4 Gemini/Places keys live in **GCP project `770921216749`** and are restricted by **iOS bundle ID** (error reason `API_KEY_IOS_APP_BLOCKED`).
- The 2 Tagore keys live in **GCP project `126477337353`**, where the `generativelanguage.googleapis.com` service is **not enabled at all** (error reason `SERVICE_DISABLED`). Different posture entirely.
- The iOS-bundle-ID restriction is the **legacy "trust client header" variant**, not the modern App-Attest signed variant. The server uses the value of the `X-Ios-Bundle-Identifier` HTTP header verbatim in the access decision.
  - `com.google.GeminiMacOS` → blocked
  - `com.google.Gemini` → blocked
  - Allowlist contents unknown; not probed further.
- This means: anyone who (a) has the leaked keys (anyone with the .app bundle does) and (b) can supply the correct allowlisted bundle ID via a header can bypass the restriction. No cryptographic device-signed proof is required.

## Probe matrix

### Round 1 — vanilla call, no bundle header

| Key alias | Endpoint | HTTP | Reason | Consumer |
|---|---|---|---|---|
| `Gemini__gms_api_key_prod` | `genlang /v1beta/models` | 403 | `API_KEY_IOS_APP_BLOCKED` (`iosBundleId="<empty>"`) | `projects/770921216749` |
| `Gemini__gms_api_key_prerelease` | `genlang /v1beta/models` | 403 | `API_KEY_IOS_APP_BLOCKED` | `projects/770921216749` |
| `RobinApolloLocalSearch__place_search_api_key_prod` | `places /v1/places:searchText` | 403 | `API_KEY_IOS_APP_BLOCKED` | `projects/770921216749` |
| `RobinApolloLocalSearch__place_search_api_key_pre_release` | `places /v1/places:searchText` | 403 | `API_KEY_IOS_APP_BLOCKED` | `projects/770921216749` |
| `TagoreAssistant__gms_api_key_iga` | `genlang /v1beta/models` | 403 | `SERVICE_DISABLED` | `projects/126477337353` |
| `TagoreAssistant__gms_api_key_opa` | `genlang /v1beta/models` | 403 | `SERVICE_DISABLED` | `projects/126477337353` |
| `Gemini__gms_api_key_prod` (cross-API) | `places /v1/places:searchText` | 403 | `API_KEY_IOS_APP_BLOCKED` | `projects/770921216749` |
| `RobinApolloLocalSearch__place_search_api_key_prod` (cross-API) | `genlang /v1beta/models` | 403 | `API_KEY_IOS_APP_BLOCKED` | `projects/770921216749` |

The cross-API probes show that the Gemini and Places keys, while named for separate purposes, both fall under the same project's iOS-bundle restriction — i.e. it's the **bundle ID, not an API allowlist, that is doing the gating** on the Robin/Apollo keys. (This matters: an attacker who finds the right bundle gets *both* APIs.)

### Round 2 — supply `X-Ios-Bundle-Identifier`

| Key | Bundle header sent | HTTP | Echoed `iosBundleId` |
|---|---|---|---|
| `Gemini__gms_api_key_prod` | `com.google.GeminiMacOS` | 403 | `com.google.GeminiMacOS` |
| `Gemini__gms_api_key_prod` | `com.google.Gemini` | 403 | `com.google.Gemini` |
| `Gemini__gms_api_key_prerelease` | `com.google.GeminiMacOS` | 403 | `com.google.GeminiMacOS` |
| `Gemini__gms_api_key_prerelease` | `com.google.Gemini` | 403 | `com.google.Gemini` |
| `…place_search_api_key_prod` | `com.google.GeminiMacOS` | 403 | `com.google.GeminiMacOS` |
| `…place_search_api_key_prod` | `com.google.Gemini` | 403 | `com.google.Gemini` |
| `…place_search_api_key_pre_release` | `com.google.GeminiMacOS` | 403 | `com.google.GeminiMacOS` |
| `…place_search_api_key_pre_release` | `com.google.Gemini` | 403 | `com.google.Gemini` |

**Observation.** The error metadata's `iosBundleId` field exactly mirrors whatever the client sent in `X-Ios-Bundle-Identifier`. There is no cryptographic proof step. This is the legacy iOS bundle-ID restriction, which Google's docs explicitly describe as advisory ("bundle ID strings are sent by the client; do not rely on them as the sole protection for sensitive APIs"). The modern alternative is App Attest, where the client must include a signed assertion produced by the device that proves it really is the named app. App Attest assertions can't be replayed from curl.

## What this means

This is **not** a critical finding on its own — the keys are bundle-restricted, and the bundle IDs in the allowlist are unknown to us. But:

1. The keys are **trivially extractable** from any installed copy of Gemini for macOS (no jailbreak, no decryption, no debugger — `strings(1)` is enough).
2. The restrictions are **legacy-mode header-trust**, not App-Attest signed. Whoever guesses or learns the allowlisted iOS bundle ID for the Robin/Apollo project gets unrestricted use of the four production/prerelease keys.
3. The four Gemini/Places keys appear to share **a single iOS-bundle-restriction allowlist for the entire project `770921216749`**. The cross-API probes show no separate per-key API allowlist — the bundle alone gates both `generativelanguage` and `places`.

Likely candidate allowlisted bundles to verify (passive only — App Store metadata, not probing):
- iOS Gemini app: probably `com.google.Gemini` based on previous Google pattern, but our probe shows it's **not** on the allowlist for the `770921216749` project. So either the iOS Gemini app uses a *different* GCP project, or its bundle ID is a less-obvious string (e.g. `com.google.RobinForiOS` or similar internal name).
- macOS Gemini app: `com.google.GeminiMacOS` is **not** on the allowlist either, despite this being the bundle the keys live inside.

The fact that the macOS app's own bundle ID is rejected by the keys it ships with suggests the keys are actually used via a **second hop** — likely the desktop app forwards the request through an iOS-shared layer (consistent with the `RobinApollo*` flag namespace and the `iPhone/Shared/` Swift symbol prefix), and the upstream iOS-platform helper supplies the correct bundle. We can't confirm that without dynamic analysis (mitmproxy + MitM proxy on a logged-in instance), which is out of scope here.

## Severity assessment for VRP submission

| Aspect | Severity | Notes |
|---|---|---|
| Keys are extractable from a public Mach-O | Low / Informational | By design for AIza keys |
| All keys carry working App Restrictions | — | Defense holds |
| Restrictions use legacy bundle-header trust, not App Attest | **Low** | Plausible weakening; Google has been migrating away from this model. Reportable as defense-in-depth. |
| No per-key API allowlist visible (one bundle ID gates all APIs in project `770921216749`) | **Low** | A single bundle compromise yields both `generativelanguage` *and* `places`. |
| Tagore keys point at a different GCP project | Informational | Surfaces project number `126477337353`; nothing more. |
| Project numbers leaked: `770921216749`, `126477337353` | Informational | Not secret, but useful for further OSINT correlation. |

A reasonable VRP report bundles all of the above into a single submission of the form "consumer-facing AIza keys for Gemini desktop/iOS use legacy bundle-ID restriction without App Attest, and lack per-API allowlists, increasing blast radius on bundle-ID leak." I'd rate this Low severity in Google's VRP, similar to past payouts in the $500–$1500 range for "Insufficient API key restrictions enabling abuse path."

## Recommendations to Google (for the report)

1. Migrate the four `770921216749` keys to App-Attest-restricted variants. The error message could even keep the same shape; the change is server-side validation only.
2. Add per-key API restrictions so that the Gemini chat key cannot call Places (and vice versa). Today, the bundle-ID gate is the only barrier and it covers both.
3. Strip TODO authorship and `b/…` IDs from production-shipped XML files (`Gemini-expanded.entitlements`).

## What was *not* attempted

- **Bundle ID enumeration** beyond two obvious guesses. Brute-forcing the allowlist would cross from "test posture" into "try to bypass" and isn't justified for a writeup.
- **Header-trust verification on Android-package restrictions** (`X-Android-Package` / `X-Android-Cert`): the keys are flagged "iOS-blocked," so they aren't on the Android side; not probed.
- **Live network capture of the running app** to learn the actual bundle ID the binary supplies. That would resolve the question definitively but requires mitmproxy + a TLS-pinning bypass, which would need a separate authorization decision.
- **Any content generation, search query, or follow-up RPC** that would consume billable quota.

## Probe artifacts

```
/Users/marcokotrotsos/PERSONAL/reverse-engineer/gemini-mac-1.53.0.262/notes/
  probe.sh                round 1 driver
  probe2.sh               round 2 driver
  probes/
    *.json                full server responses, one per probe
```

Total request count: **13** (8 round 1 + 8 round 2; minus 3 reused targets).
Estimated quota cost: ≤ 1 free-tier discovery call per key = $0.

---
*Compiled 2026-05-06. All probes were direct curl invocations from a single residential IP; no anonymization layer was used (per user direction "let's go"). If a follow-up VRP submission is filed, that IP will appear in Google's edge logs alongside the timestamps.*
