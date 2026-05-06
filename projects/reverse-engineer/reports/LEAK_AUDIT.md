# Gemini for macOS 1.53.0.262 — Leak Audit

Companion to REPORT.md. This pass focuses on what the production binary unintentionally tells you about Google's internal infrastructure. Everything below was extracted from the local file `/Applications/Gemini.app/Contents/MacOS/Gemini` with `strings(1)`. No external probing.

> Methodology: passive observation of a publicly distributed binary the user owns. No attempts were made to probe Google's internal or external systems with any of this material.

---

## 1. Hardcoded API keys (six of them)

| Flag | Key | Bucket |
|---|---|---|
| `Gemini__gms_api_key_prod` | `AIzaSyCdp-tyumnVxYPL7vPBPmkVJMokbE2NjA4` | Gemini, prod |
| `Gemini__gms_api_key_prerelease` | `AIzaSyAkKv9vgi9lcpTttBjI0mVRwtWq3hwnRqw` | Gemini, prerelease |
| `RobinApolloLocalSearch__place_search_api_key_prod` | `AIzaSyAs-th1kQweF-6b_L9Rv1_IdfuwOmoWyZ4` | Places, prod |
| `RobinApolloLocalSearch__place_search_api_key_pre_release` | `AIzaSyBjFTbqiMdfcnFxhPwumItUiKzU8-aNy98` | Places, prerelease |
| `TagoreAssistant__gms_api_key_iga` | `AIzaSyBwu1q5p-HA745oE-YssxrrKu4UjaHv-7o` | "Tagore" assistant, IGA |
| `TagoreAssistant__gms_api_key_opa` | `AIzaSyC8X_cCSqhqe-dHQ7tDpVvM2SDkSO6Q5Qg` | "Tagore" assistant, OPA |

**What this is.** These are `AIza…` keys, Google's "browser/API key" type. They are *designed* to be embedded in clients — the trust boundary is at the API console, where Google places **Application Restrictions** (bundle-ID + code-signature for iOS/macOS, package-name + cert hash for Android, HTTP referrer for web) and **API Restrictions** (this key may only call these specific APIs). If those restrictions are configured tightly, leaking them is fine.

**What's actually leaked.**

1. The pairing of *flag name → key* tells you which key gates which surface. So an attacker doesn't have to guess which key is for what.
2. The existence of separate **prerelease vs prod** keys tells you Google maintains parallel quotas and the prerelease one likely has weaker rate limits and is allowed to talk to staging endpoints (see §3).
3. A previously unseen codename, **Tagore**, with two sub-flavors `iga` and `opa`. Tagore is most likely an India-region Assistant variant (named for the Bengali Nobel laureate). `iga`/`opa` are two installation modes — based on naming conventions elsewhere, plausibly *In-app Gemini Assistant* and *Out-of-Process Assistant*, though I'm guessing.

**Risk read.** Discovery is low risk on its own *if* the App Restrictions are correctly set to `EQHXZ8M8AV.com.google.GeminiMacOS`. If they aren't (or the keys are permissive on purpose for the public Gemini API surface), they're abusable for quota theft. I did not test this and won't.

## 2. Public-but-undocumented API hostnames

The binary references these `*.googleapis.com` endpoints. The `-pa` suffix is Google's convention for "Private API" (auth + key required, but the hostname is public DNS).

```
robinfrontend-pa.googleapis.com           ← Gemini chat backend (Robin), prod
preprod-robinfrontend-pa.googleapis.com   ← Gemini chat backend (Robin), preprod
accountlinking-pa.googleapis.com
directpath-pa.googleapis.com
feedback-pa.googleapis.com
photospicker.googleapis.com
speechs3proto2-pa.googleapis.com          ← speech to text
mtls.googleapis.com
xds.googleapis.com                        ← gRPC Traffic Director xDS
oauth2.googleapis.com
play.googleapis.com
sandbox.googleapis.com
www.googleapis.com
update.googleapis.com
```

**Notable:** `preprod-robinfrontend-pa.googleapis.com` is the *publicly resolvable hostname* for Gemini's preprod chat backend. Hitting it requires both a key and auth, but the existence of the hostname (and the implicit knowledge that the prerelease API key above is the one that's allowed to talk to it) is a chunk of intel that wasn't otherwise public.

## 3. Internal Google source-tree leak

Asserts/CHECK failures in vendored C++ libraries embed their google3 paths. The macOS substrate Gemini is built on is internally codenamed **Argo**, owned by **gdm** (= Google DeepMind):

```
gdm/apps/janus/argo/
  janus_lib.cc
  base/
    files/file_system.{h,cc}, file_manager.cc, user_dirs_mac.mm
    threading/{message_loop,thread,thread_checker,task_executor.h,...}
    synchronization/{blocking_queue.h, watchable_event_posix.cc}
    posix/posix_util.cc
    power/power_manager_mac.cc
    json/json_parser.h
    janus_log.cc
  net/
    curl_api.cc                          ← libcurl-based transport
    system_certificate_store_mac.mm      ← reads macOS keychain trust roots
    network_status_listener_mac.mm
    unauthenticated_transport.cc         ← class name worth flagging
  auth/
    account_manager.cc
    account_context.cc
    account_linking_http_server.cc       ← localhost HTTP server for account link
    auth_code_provider_localhost.cc      ← OAuth desktop loopback flow
    oauth_http_server.cc                 ← localhost HTTP server for OAuth
    oauth_credential.cc, oauth_config.cc
    credential.cc, credential_store.h
    credential_data.proto, account_config.proto
  crypt/
    crypt_manager_impl.cc
    keystore.proto, keystore_loader_mac.cc      ← macOS keychain-backed keystore
    container.proto                              ← encrypted container format
  proto/
    account_id.proto, account_info.proto, app_info.proto,
    feedback.proto, runtime_info.proto, status.proto
  rpc/feedback_client.cc
  feedback/feedback_manager.cc
```

So we now know:
- The cross-platform desktop substrate is **Argo** (was previously only "Janus" in symbol mangling).
- Owned by GDM (Google DeepMind), not the Search/Assistant org.
- The auth flow uses the standard desktop **OAuth loopback** pattern via a localhost HTTP server (`auth_code_provider_localhost.cc`, `oauth_http_server.cc`). This is fine cryptographically, but worth knowing the binary opens loopback ports during sign-in.
- There is an `unauthenticated_transport.cc` — almost certainly used for pre-login health/version checks, but worth a future look.
- macOS-specific implementations sit alongside (`*_mac.{cc,mm}`) — confirms Argo is a multi-platform tree.

## 4. Engineer attribution

Two TODO comments in `Gemini-expanded.entitlements` carry author LDAPs:

```
TODO(yilongyao): Ideally re-enable unless we absolutely have no solution
                 for a11y and input control.
TODO(jiagu):     Add aps-environment once provisioning profile is updated
                 (b/502756758).
```

So `yilongyao@` and `jiagu@` are on the macOS Gemini team. Not sensitive on its own but it's typical Google infosec hygiene to strip these — the file shipping inside the bundle as XML is the leak.

## 5. Bug IDs (18 unique)

```
b/18148417   b/22036714   b/24559754   b/25010963
b/26741429   b/30511309   b/68064263   b/68823608
b/120484336  b/142477079  b/194822211  b/243388788
b/264528453  b/271584605  b/290224461  b/301106913
b/342232822  b/357090229  b/502756758   ← Gemini-team-specific
```

Most are from vendored Google C++ libraries (Census, Stubby, ThinMint) and have been in many Google binaries for years. The one specific to this app — `b/502756758` — is the open ticket for adding push notifications to Gemini Mac.

## 6. Internal infrastructure inventory

The corp-link rewriter table maps 87 distinct internal services. Loosely categorized:

| Category | Hosts |
|---|---|
| **Bug tracker (Buganizer)** | `b.corp.google.com`, `issuetracker.google.com`, `skyvine` |
| **Code search (Piper)** | `cs.corp.google.com`, `cl.corp.google.com`, `source.corp.google.com` |
| **Gerrit / git review** | `*-review.git.corp.google.com` (privacysandbox, partner-android, prodkernel, devrel, sapbtp, gchips-internal, igi-team, googleplex-polygon-android, moohan-private, infinity-internal, hekate-internal, turquoise-internal, fuchsia, gke-internal, qball, **waymo-launches**) |
| **Org / people** | `moma`, `teams.googleplex.com`, `whostory` |
| **Docs / kb** | `g3doc`, `playbooks`, `yaqs` (Q&A), `dory` (Q&A), `t.corp.google.com` |
| **Releases / launches** | `ariane` (launch), `pcms` (PCR view), `torx` (PCR), `cdpush`, `irm` (incidents) |
| **Build / CI** | `android-build.googleplex`, `fusion2` (test invocations), `dataeng` (Flume) |
| **Monitoring / debug** | `borgcron-dashboard`, `symbolize` (stack traces), `cnsviewer` |
| **Access / vendors** | `accessnow`, `vendorgateway`, `xids`, `grants`, `arsp` |
| **Legal** | `legal-removals`, `legal-removals-staging` |
| **ML training** | `vizier` (hyperparameter tuning), `throughline` (studies) |
| **Privacy review** | `geo-imagery-inspector` |
| **Misc** | `g.corp.google.com`, `o.corp.google.com`, `jumper`, `simhub`, `bamach`, `gnp`, `erkp`, `dm`, `mpmbrowse`, `gamma`, `gamma-staging`, `ganpati`, `ganpati2` |
| **External-Google reach** | `mail.google.com/chat/#chat/space/...`, `uma.googleplex.com/p/chrome/finch/` |

Two specifically interesting entries:

- `https://uma.googleplex.com/p/chrome/finch/\1` — that's the **Chrome Finch experiment ID viewer**, telling you that the same regex library is used by Chrome's experiment tooling. Suggests the Janus binary inherits the corp-link rewriter from Chromium's tree.
- `https://waymo-launches.corp.google.com/issues/\1` — **Waymo's** internal launch tracker. Surprising entry to find inside a chat app; same library reused.

## 7. `go/` shortlinks (100 unique)

A sample of the more interesting ones:

```
go/ThinMintTesting               ← test ThinMint signing keys
go/thinmint-missing-key
go/thinmint-design-key-reloading
go/sp-environments#data-realm    ← DAT signing keys, data realms
go/extension-declarations
go/extension-declarations-oss-plan
go/prod-naming-1-5
go/dg-classification             ← internal data classification taxonomy
go/dg-classification:public_user_data
go/dg-classification:user_class#Data
go/dg-classification:user_class#Data.managed
go/dg-classification:financial_data#financial
go/dg-classification:geo_location#Data.Coarse
go/dg-classification:geo_location#Data.Traces
go/dg-classification:categories#audio_data
go/dg-classification:categories#has_xfood   ← xfood = experimental food = dogfood
go/user-data-accessor-usage-enum
go/pgraph-minor_data_category-proposal
go/thread-note-consistency
go/ppv2#bookmark=id.…
```

The interesting bit isn't any single shortlink — it's the **data-classification taxonomy itself**. We can read the schema:
- A `user_class` sub-attribute `Data.managed` exists (managed = workspace tenant-controlled)
- `geo_location` is split into `Data.Coarse` (IP-level) and `Data.Traces` (continuous trail)
- A `has_xfood` boolean — Google internally tags whether a piece of data is dogfood-only, and *that tag rides the data through the entire pipeline*. The fact that this enum constant is even reachable from a production client binary is the leak.

## 8. Internal frameworks confirmed in-binary

| Framework | What it is | How we know |
|---|---|---|
| **Argo** | macOS/desktop substrate (vendored Chromium-style C++ tree) | source paths `gdm/apps/janus/argo/...` |
| **Janus** | Swift app layer on top of Argo | symbol prefix `googlemac/Janus/MacApplication/...` |
| **Robin** | Gemini chat product (server-side) | `ROBIN…` proto types and `robinfrontend-pa` |
| **Apollo** | iOS layer the macOS UI shares with | flag namespace `RobinApollo*` |
| **Tagore** | India-region (?) Assistant variant | `TagoreAssistant__gms_api_key_*` |
| **ThinMint** | RPC auth-token mint / signing | strings + `go/thinmint-*` |
| **DAT** | parallel signing/verification system | "prod and test versions of the DAT signing keys" |
| **Stubby2** | RPC framework | `DDTContext`, `DMA 52 enforcement` |
| **Census** | telemetry/profiling | `census_cpu_accounting_enabled` flags |
| **Phenotype / Mendel** | experimentation | `PHTExperimentState…`, `JANExperimentFlagsRegistration…` |
| **Borg / borglets** | container orchestrator | "If running under borglets, binary logging is disabled by default" |
| **Vizier** | hyperparameter tuning | corp link present |
| **Streamz** | counters / metrics | `/client_streamz/janus_macos/…` |
| **Clearcut** | event logging | `CCTClearcutLogEvent`, `PRMClearcutTransmitter` |
| **GAIA** | Google account ID | `gaiaIdProvider` in transmitter init |
| **Piper** | source control | `/depot/`, `/depot/google3/` URL templates |

This isn't classified info — most are public to varying degrees. But seeing them all coexisting in a *consumer Mac app* is the surprising part. The app drags Borg/Census/Stubby/ThinMint awareness with it because it shares the same C++ trunk libraries the rest of Google runs on.

## 9. Severity summary

| Finding | Severity | Notes |
|---|---|---|
| 6 `AIza…` keys | Low → Medium (depends on App-Restrictions config) | Not secret by design, but leak gives precise flag→key mapping incl. preprod |
| `preprod-robinfrontend-pa.googleapis.com` exposed | Low | Public DNS, but knowing it exists + which key gates it is intel |
| Internal source-tree paths (`gdm/apps/janus/argo/...`) | Low | Code structure leak, no secrets |
| Engineer LDAPs in entitlements TODOs | Low | Two LDAPs (yilongyao, jiagu) |
| Buganizer ID `b/502756758` for push notifications | Low | Single internal ticket reference |
| Internal corp-link rewriter table | Informational | Same library is in many Google apps; vendoring artifact |
| `go/dg-classification:has_xfood` etc. | Informational | Schema fragments of internal data taxonomy |
| Codename "Tagore" + iga/opa | Informational | New product codename surfaced |
| Codename "Argo" for desktop substrate | Informational | New substrate codename surfaced |
| OAuth loopback HTTP server in-process | Informational | Standard pattern, but flag for any future port-binding analysis |

## 10. What I did NOT do

- Send any traffic to corp.google.com hosts
- Test API keys against Google's APIs
- Probe `preprod-robinfrontend-pa.googleapis.com`
- Attempt to resolve internal DNS names
- Try to use the OAuth loopback flow

If you want any of those done as part of an authorized exercise, that's a separate decision — these would all touch Google's external surface. The findings above are purely from the binary on disk.

## 11. Suggested follow-ups (passive only)

1. Diff this binary against an older Gemini for macOS to see which leaks pre-existed vs. which are new in 1.53.0.262.
2. Diff against the iOS Gemini IPA (if you have one) — Apollo / Robin-side leaks should overlap heavily; differences will isolate macOS-only Argo bits.
3. Pull `Assets.car` with `acextract` / `cartool` — it likely contains rasterized assets that include strings and attribution.
4. Decode `embedded.provisionprofile` (`security cms -D -i`) — should reveal the App ID, capabilities, and possibly the named devices/managers if a Distribution profile.
5. Walk the Phenotype config snapshot in `~/Library/Group Containers/group.com.google.gemini/` (if any) once the app is launched — Phenotype caches flag → value pairs locally.

---
*Audit compiled 2026-05-06 from `/Applications/Gemini.app/Contents/MacOS/Gemini` (arm64, signed Google LLC, Team EQHXZ8M8AV).*
