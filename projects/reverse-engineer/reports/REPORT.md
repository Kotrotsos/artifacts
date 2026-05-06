# Gemini for macOS 1.53.0.262 — Reverse Engineering Report

**Date:** 2026-05-06
**Bundle:** `/Applications/Gemini.app`
**Bundle ID:** `com.google.GeminiMacOS`
**Version / Build:** 1.53.0.262 (CFBundleShortVersionString == CFBundleVersion)
**Min macOS:** 15.0
**Architectures:** arm64 only (no Intel slice)
**Compiled with:** Xcode 17C52, SDK macosx26.2, clang
**Code signature:** Developer ID Application, Google LLC, Team `EQHXZ8M8AV`, Apr 29 2026
**Update channel:** `KSChannelID = m1-prod` (Google Keystone updater, M1 production channel)

---

## 1. Bundle layout

```
Gemini.app/Contents/
  Info.plist
  PkgInfo
  embedded.provisionprofile          (12.7 KB — Apple provisioning profile)
  MacOS/Gemini                        (main binary, single arm64 slice)
  Frameworks/libswiftCompatibilitySpan.dylib
  Helpers/
    GeminiAppLauncher.app             (login-item helper, com.google.GeminiMacOS.launcher)
    crashpad_handler                  (Google Crashpad reporter)
  Resources/                          (~150 entries)
    AppIconRelease.icns
    Assets.car
    Gemini-expanded.entitlements      (build-time entitlements with TODO comments)
    com.google.gemini.client.json     (Chrome NativeMessaging host manifest)
    GIPPseudonymousID.bundle          (Google Identity Pseudonymous ID, ~78 lprojs)
    GoogleSans{Bold,Flex,Medium,Regular}.bundle (incl. variable Flex font)
    GoogleSansText-*.ttf              (UI text family)
    XITSMath-Regular.otf, mathFonts.bundle (latinmodern-math, texgyretermes-math) — LaTeX
    highlightjs_bundle.bundle/highlight.pack.js (~1.1 MB) — syntax highlighting
    gRPCCertificates.bundle/roots.pem (~258 KB) — pinned gRPC CA bundle
    GPI_Aurora_Spark.json, GPI_Aurora_Spinner.json — Lottie animations
    GelIdle.mp4 (~7 MB) — idle "gel" visual
    *.lproj                          — ~100 locales
```

The app is **not sandboxed** but uses the hardened runtime.

## 2. Identity, code-signing, entitlements

```
Authority=Developer ID Application: Google LLC (EQHXZ8M8AV)
Identifier=com.google.GeminiMacOS
Runtime Version=26.2.0
Sealed Resources rules=13 files=651
```

**Runtime entitlements:**
- `app-sandbox = false`
- `application-groups = [group.com.google.common, group.com.google.gemini]` (shared container with other Google apps)
- `device.audio-input = true`
- `device.camera = true`
- `files.user-selected.read-only = true`
- `network.client = true`
- `network.server = true`
- `keychain-access-groups = [EQHXZ8M8AV.com.google.GeminiMacOS]`

**Build-time entitlements file (`Gemini-expanded.entitlements`) — leaked TODOs:**
```xml
<!-- TODO(yilongyao): Ideally re-enable unless we absolutely have no
     solution for a11y and input control. -->
<!-- <true/> -->
<false/>   <!-- com.apple.security.app-sandbox -->
...
<!-- TODO(jiagu): Add aps-environment once provisioning profile is updated
     (b/502756758). -->
```

Translation: Google ships unsandboxed because Gemini needs Accessibility-API and synthetic-input access to drive the OS (Computer Use / Agent Mode). Push notifications are planned but blocked on a provisioning-profile update; the internal bug ID `b/502756758` made it into the binary.

`LSUIElement = true` → it is a menubar/agent-style app, not a Dock app by default.

## 3. Linked frameworks (selected)

Networking + IPC: `Network`, `CFNetwork`, `Kerberos`, `ServiceManagement`
Media / capture: `AVKit`, `AVFoundation`, `AVFAudio`, `AudioToolbox`, `CoreAudio`, `CoreMedia`, `CoreVideo`, `ScreenCaptureKit`
UI: `Cocoa`, `AppKit`, `SwiftUI`, `Combine`, `WebKit`, `JavaScriptCore`, `PDFKit`, `QuickLookThumbnailing/UI`
Persistence + crypto: `SwiftData`, `CryptoKit`, `Security`
ML / language: `NaturalLanguage`
Other: `Accessibility`, `CoreLocation`, `MetricKit`, `Carbon`

The combination — `ScreenCaptureKit` + `Accessibility` + `JavaScriptCore` + `WebKit` + camera + mic + the unsandboxed runtime — is what enables Gemini Live + Agent / Computer-Use modes locally.

## 4. Internal architecture (Janus)

Swift demangled symbols expose the full module tree. The macOS app is built on Google's internal **Janus** framework:

```
googlemac/
  Janus/
    LibWrapper/API/ ...
    MacApplication/
      Components/
        Chat/MessageTree/Container, PasteboardSupport, AttributedStringTypes
        Chat/Canvas/SelectableMessageContentView           ← in-chat canvas
        Markdown/WebView, Coordinator, NonScrollingWebView ← markdown via WebKit
        MyStuff/   ← "My Stuff" sidebar
        Sidebar/
        RichInput/AutocompleteProvider
        MediaPanel/
      Services/
        StartupTask/  (DidFinishLaunching / WillFinishLaunching API)
        RPC/
          S3Service          + Interceptor / Streaming / Impl
          RobinService       + Interceptor / Streaming / Impl   ← chat backend
        FilePicker
        DrivePicker
        PhotosPicker
        ChatService
        GMacChecker
        GeminiLive/Impl/    (CameraCaptureManagerDelegate, ScreenCaptureManagerDelegate)
        NotebookLM/         (NotebookLMService)                  ← NotebookLM
        ShareLogger
        TaskService                                              ← scheduled tasks
        UserService
        XPC_Service                                              ← XPC for helpers
  iPhone/Shared/             ← code shared with iOS Gemini app
    RPC/API/...              (Authorizer, Transport, Interceptor)
    GoogleSearch/AppFlows/   (Host, Clock)
    GoogleSearch/SDS         (Search Design System — Typography, ExtendedCommonMark)
    People/ShareKit
  Shared/Logging/Loggers/ClientLogger/Decorator
  Shared/Metrics/Clearcut/LogLoss
```

**Codenames worth noting:**
- **Janus** — internal name for Google's shared macOS app foundation.
- **Robin** — Gemini's chat product proto namespace. Almost every server-side message type is `ROBIN…` (e.g. `ROBINAction`, `ROBINActivityBlock`, `ROBINActionCard…`, `ROBINGeneratedVideo`, `ROBINSafeguardCondition`, `ROBINVeoDiscovery`).
- **Apollo** — the iOS layer; many flags are namespaced `RobinApollo*` because the macOS UI shares plumbing with iOS Apollo.

## 5. Generative Language API surface (client-embedded protos)

Hundreds of `Google_Ai_Generativelanguage_V1main_*` Swift proto types are baked into the binary, consistent with `generativelanguage.googleapis.com` v1main internal API. Notable types found:

`AllowedTools`, `AssetMetadata`, `AttributionReference`, `AudioContentOptions`, `AudioOptions`, `AudioTrack`, `AudioTranscription`, `Bash`, `BeyondProcessorConfig`, `BilledTool`, `Blob`, …

The presence of a `Bash` tool type and `AllowedTools` confirms tool-use / shell-tool wiring on the API side, and `BilledTool` is suggestive of per-tool billing metadata.

`ROBINGeneratedVideo_GeneratedVideoMetadata_VeoMode` enum carries:
`Veo3T2V`, `Veo3I2V`, `Veo3FastT2V`, `Veo3FastI2V`, `Veo3Fast12T2V/I2V`, `Veo3Fast30T2V/I2V`, `Veo3Fast30GPUT2V/I2V`, `Veo3Step20T2V/I2V`, `VeoModeOmni` — i.e. multiple Veo3 quality / latency tiers, including a "GPU" fast tier and a 20-step variant.

## 6. Features confirmed in the client

From symbol/string evidence (constants, async client IDs, capability enums, feature-flag strings):

- **Gemini Live** — full live mode pipeline. Camera + screen capture managers in `GeminiLive/Impl`. Live extensions: Drive, Gmail, Google Home, Maps Navigation, Photos, Spotify, Scheduled Actions, YouTube, YouTube Music. Per-stream Robin op execution, post-conversation summary, captions v3, tool consent dialog. TOS versions 1–8.
- **Agent Mode** — `AgentMode__enable_agent_mode`, `RobinApolloAgentMode__enable_*` flags (activity UI, immersive toolbar status chip, mutating-op buttons, visible notifications). `ENABLE_AGENT_MODE_ACTIVITY_BLOCK`. Status text "Agent mode in progress for chat …".
- **Computer Use** — `ASYNC_CLIENT_ID_GEMINI_COMPUTER_USE`. (This is what motivates the disabled sandbox.)
- **Deep Research** — `ASYNC_CLIENT_ID_GEMINI_DEEP_RESEARCH(_API)`, plus internal-API callers `AIR`, `AUTOKAGGLE`, `COSCIENTIST`, `EVAL`, `GENAI`, `TAILWIND`, `WORKSPACE_WORKFLOWS`. Banners, link sharing, copy-rich-text, ToC, persisted scroll position, immersive artifact type.
- **Canvas / Artifacts** — embedded React+Tailwind WebView with strict CSP and CDN allowlist:
  ```
  script-src ... blob: https://cdn.tailwindcss.com https://cdnjs.cloudflare.com
              https://unpkg.com https://esm.sh
  connect-src https://esm.sh https://unpkg.com https://cdnjs.cloudflare.com
  ```
  Imports e.g. `lucide-react` from `esm.sh`. Two HTML shells: a strict-CSP "React Canvas" host and a permissive Tailwind "Canvas" host. So Gemini's Canvas hot-loads React + libs at runtime per artifact (analogous to Claude Desktop's Artifacts but pulling from public CDNs).
- **NotebookLM** — `Notebooks__enable_notebooks`, `CAN_USE_GEMINI_NOTEBOOKS`, `has_accepted_gemini_notebooks_eea_disclaimer`, `Services/NotebookLM/NotebookLMService`.
- **Audio Gen / Video Gen** — `ASYNC_CLIENT_ID_GEMINI_AUDIO_GEN`, `_VIDEO_GEN`. Lyria3 caller `CALLER_DEEPMIND_GEMINI_API_LYRIA3`; Vertex callers `VERTEX_AI_LYRIA3`, `MUSIC_LM`, `VISION_IRIS`, `VISION_VEO`.
- **Duplex** — `ASYNC_CLIENT_ID_GEMINI_DUPLEX` (phone-call agent surface).
- **Scheduled tasks** — `ASYNC_CLIENT_ID_GEMINI_SCHEDULED_TASKS`, `Services/TaskService`.
- **Math + LaTeX rendering** — XITS Math + Latin Modern Math + TeX Gyre Termes Math fonts shipped.
- **Code rendering** — Highlight.js bundle (~1.1 MB) shipped locally, no fetch needed.

## 7. Chrome Native Messaging bridge

`Resources/com.google.gemini.client.json`:

```json
{
  "name": "com.google.gemini.client",
  "description": "Native helper process for the Gemini for desktop companion component extension.",
  "path": "{BINARY_LOCATION}",
  "type": "stdio",
  "allowed_origins": [
    "chrome-extension://elodpogpfcinehhjanfmoocfkmojjmce/",
    "chrome-extension://plaeniloeifmajgdcaonhdnolpfjfhdg/",
    "chrome-extension://bcchonjkefjaplocnihbnnfegialmkpl/"
  ]
}
```

The Mac app exposes itself as a Chrome NativeMessaging host. Three Chrome extension IDs are whitelisted to attach over stdio to a "companion component" — i.e. a browser-side extension can speak to the desktop Gemini binary directly. The manifest is delivered to disk via Janus's `GMacChecker` / installer logic at runtime (the `path` is templated `{BINARY_LOCATION}`, suggesting the app installs the resolved manifest into Chrome's NativeMessagingHosts directory on first launch).

## 8. Telemetry / experiments

- **Phenotype + Mendel** experimentation: `PHTExperimentStateBaseImpl`, `PHTFlatFilePhenotype`, `JANExperimentFlagsRegistrationServiceImpl`, `phenotype_sync_count`. Server flags ship through Phenotype.
- **Streamz client metrics** under `/client_streamz/janus_macos/`:
  - `chat_created_count`, `chat_deleted_count`
  - `phenotype_sync_count`
  - `user_prompts_with_file_attachments`
  - `zero_state_view_impression_count`
- **Clearcut** event logging via `CCTClearcutLogEvent` and `PRMClearcutTransmitter` (`GAIA-id-aware`).
- **MetricKit** linked (Apple-side perf telemetry).
- **gRPC over xDS**: traffic-director-p2p.xds.googleapis.com listener template found — the client speaks gRPC with xDS service discovery against Google's Traffic Director.

## 9. Curiosity: shipped Google-internal corp link rewriter

A large regex rewrite table is embedded in cleartext, mapping link patterns to internal Google URLs:

```
https://b.corp.google.com/issues/\1
https://moma.corp.google.com/{team,system,product}/\1
https://fusion2.corp.google.com/invocations/\1
https://cs.corp.google.com/piper///depot/{,google3/}\1
https://gke-internal-review.googlesource.com/\1
https://issuetracker.google.com/issues/\1
https://privacysandbox-review.git.corp.google.com/c/+/\1/\2
https://uma.googleplex.com/p/chrome/finch/\1
https://dataeng.corp.google.com/flume?execution_id=\1
https://geo-imagery-inspector.corp.google.com/details?id=\1
... ~50 more
```

…plus a comment hinting at internal `go/` documentation:

> "You are probably mixing debug and production versions of the ThinMint signing/verification keys. For more information, see http://go/ThinMintTesting. If this happens in production, see http://go/thinmint-missing-key."

This is the same corp-link library that ships in many Google apps, but it is unredacted in production. Not exploitable, but it confirms a lot of internal infrastructure (ThinMint = signing key infra) that you wouldn't normally see surfaced.

## 10. Helpers

- **`GeminiAppLauncher.app`** — `com.google.GeminiMacOS.launcher`, version 1.53.00.0. Tiny app whose only job is to be registered as a login item via `SMAppService` (matches the linked `ServiceManagement` framework). `NSPrincipalClass = NSApplication`, `NSSupportsSuddenTermination = true`.
- **`crashpad_handler`** — Google's standard Crashpad out-of-process crash reporter.

## 11. Distribution / update mechanism

- Channel `m1-prod` is the Google Keystone (`com.google.Keystone`) M1 production channel. Updates flow through Keystone, not the App Store — consistent with the Developer ID signature and the `Distribution` provisioning profile.
- Version string `1.53.0.262` and minor "1.53" suggests a fast release train (similar to Chrome's pace), and `262` is the build number for this minor.

## 12. Quick threat-model notes

(For context only — nothing here is a vulnerability, but worth remembering since the app is unusually privileged.)

- **No sandbox**, full disk reach (subject to TCC). The expanded-entitlements file *itself states* this is a deliberate trade-off for a11y / input-control needs.
- App registers itself as a **NativeMessaging host for three Chrome extensions**; any compromise of those extension IDs would talk directly to the desktop binary.
- App can drive the OS via Accessibility (Computer Use / Agent Mode) — adds a strong reason to gate Agent Mode behind explicit per-action consent (and the `enable_mutating_op_buttons` / `enable_visible_notifications_in_agent_mode` flags suggest they do).
- Embedded React canvas pulls scripts from `cdn.tailwindcss.com`, `unpkg.com`, `esm.sh`, `cdnjs.cloudflare.com` at runtime under `'unsafe-inline'` + `'unsafe-eval'` + `'strict-dynamic'`. CSP nonces are in place for the strict shell, but the secondary "Tailwind Canvas" shell is permissive.
- gRPC roots.pem is pinned in-bundle (good).

## 13. Comparison to Claude Desktop

| Aspect | Gemini for macOS 1.53 | Claude Desktop (Electron) |
|---|---|---|
| Runtime | Native Swift / SwiftUI / AppKit, arm64 | Electron + Vite bundle (.asar) |
| Sandbox | Off (deliberate, for a11y) | Off (Electron) |
| Login item | `GeminiAppLauncher.app` via SMAppService | Electron auto-launch |
| Update | Google Keystone (`m1-prod`) | Squirrel.Mac |
| Voice / video / screen | Native: ScreenCaptureKit, AVF, Accessibility | WebRTC inside Electron |
| In-app artifacts | React+Tailwind WebKit "Canvas" with CDN allowlist | React in renderer process |
| LLM API | gRPC + `generativelanguage.googleapis.com v1main` ROBIN proto | HTTPS to api.anthropic.com |
| Telemetry | Streamz + Clearcut + Phenotype + MetricKit | Sentry + custom |

Both intentionally drop the macOS sandbox to enable agent-style features; both use a hardened runtime; both are signed Developer ID and update outside the App Store.

---

## Appendix: workspace

```
/Users/marcokotrotsos/PERSONAL/reverse-engineer/gemini-mac-1.53.0.262/
  REPORT.md                    (this file)
  strings/main.strings.txt     (572k strings from main binary, ~11 MB)
  notes/                       (empty; for future passes)
  extracted/
  resources/
```
