# Telepathy — OpenAI Codex's ambient screen observer, shipped dormant

Reverse-engineered from Codex.app v26.415.40636 (build 1799), `com.openai.codex`.  
Source: `/Applications/Codex.app/Contents/Resources/app.asar`, unpacked to `asar_full/`.

Character offsets below refer to `.vite/build/main-CUDSf52Z.js` (main process, 662 KB), `.vite/build/product-name-C630vpQ6.js` (5.0 MB worker/runtime), and `webview/assets/general-settings-BZQqrI-r.js` (renderer).

---

## One-line summary

Telepathy is OpenAI's answer to Anthropic's "Chicago" — a research-preview screen-capture background agent that builds a local memory of the user's workflows. It is *fully shipped and consent-UI ready* in the Codex binary, but the native sidecar that does the capture (`codex_telepathy`) **is not in this bundle**. Statsig gate `2574306096` and a sidecar binary drop will switch it on remotely.

---

## 1. Identity

| Property | Value |
|---|---|
| User-facing name | **Telepathy** |
| Product line | "experimental features" tab in Codex settings |
| Self-description | *"a research preview that passively captures your screen to teach Codex about your workflows and active projects"* |
| Internal category | ambient / background observer |
| Statsig gate | `2574306096` (per `STATSIG_FLAGS.md`) |
| Platform gate | **macOS only** (Windows not implemented — no `CODEX_TELEPATHY_STARTED_PID_PATH` branch on win32) |
| Sidecar binary name | **`codex_telepathy`** (constant `gS = "codex_telepathy"` at `product-name-….js:412966`) |
| Sidecar in this build | **NO** — executable not present in `/Applications/Codex.app/Contents/Resources/` |
| Required macOS permission | Screen Recording (TCC) |
| Kill-switch label | *"Pause Telepathy"* — menu-bar item |
| Adjacent codename | `codex_tape_recorder` (the parent tmp dir for Telepathy's PID file — presumably an older or sister recording mechanism) |

---

## 2. Evidence — what's hardcoded

### 2a. User-preference keys (stored in Codex's local prefs store)

From `product-name-C630vpQ6.js` @238949 and `worker.js` @640654:

```js
TELEPATHY_CONSENT_ACCEPTED:            "telepathy-consent-accepted"
TELEPATHY_SETUP_COMPLETION_PENDING:    "telepathy-setup-completion-pending"
AMBIENT_SUGGESTIONS_ENABLED:           "ambient-suggestions-enabled"   // related, sits next to Telepathy keys
```

`ambient-suggestions-enabled` is a distinct but related feature — Codex can surface passive suggestions without necessarily using screen capture. Telepathy appears to be the screen-capture implementation layer that feeds ambient suggestions.

### 2b. Sidecar launcher

From `product-name-….js:412966` (discovery logic):

```js
var gS = `codex_telepathy`;
function _S({ resourcesPath, executableName, windowsDestinationPath }) {
  if (!resourcesPath) return null;
  let r = join(resourcesPath, executableName);
  if (!DS(r)) return null;             // DS = existsSync
  if (process.platform !== `win32` || !TS(r) || windowsDestinationPath == null)
    return r;
  // ...windows path branch (currently dead code)
}
```

The launcher looks for `codex_telepathy` adjacent to the Electron resources. **On this build, that file is absent.** When `_S()` returns `null`, `getTelepathySidecarControlState()` reports `disabled`.

### 2c. Cross-process signal — PID file

From `product-name-….js:477427`:

```js
var $T = join(tmpdir(), `codex_tape_recorder`, `telepathy-started.pid`);
var eE = `CODEX_TELEPATHY_STARTED_PID_PATH`;
```

Translation: the sidecar writes its PID to `$TMPDIR/codex_tape_recorder/telepathy-started.pid`, and the main app watches that path. The env var `CODEX_TELEPATHY_STARTED_PID_PATH` lets the path be overridden — useful for dev/test scenarios.

Note the parent dir: **`codex_tape_recorder`**. Internal codename leak — the recorder concept predates or underpins the user-facing "Telepathy" name.

### 2d. IPC surface (main ↔ renderer)

From `main-CUDSf52Z.js:145114`:

```js
"telepathy-permissions": async () => {
  let sidecarPresent = cu();                                     // probes resources dir
  let ctrl = this.appServerConnectionRegistry
                 .getMaybeConnection(I)
                 ?.getTelepathySidecarControlState();
  return {
    screenRecording:             su(),                           // TCC status
    telepathySidecarPresent:     sidecarPresent,
    telepathySidecarProcessState: ctrl?.state ?? `disabled`,
  };
}
```

### 2e. Process states the renderer can observe

From the settings UI string table (`browser-sidebar-comment-preload.js`):

```
permissionStatus.checking
permissionStatus.denied
permissionStatus.granted
permissionStatus.notDetermined
permissionStatus.paused
permissionStatus.restricted
permissionStatus.running
permissionStatus.starting
permissionStatus.stopping
permissionStatus.unknown
```

10-state FSM. `paused` is a first-class state, so the menu-bar kill switch doesn't kill the process — it suspends capture.

### 2f. Sentry log tags

The main-process Sentry wrapper logs:

```
Failed to enable Telepathy
Failed to update Telepathy config
```

So Telepathy has mutable remote configuration, not just an on/off bit.

---

## 3. The consent dialog — full English copy

Pulled verbatim from `webview/assets/general-settings-BZQqrI-r.js` (`defaultMessage` literals). This is what a user would read before flipping Telepathy on.

**Dialog title**

> Enable Telepathy

**Intro paragraph**

> Telepathy is a research preview that passively captures your screen to teach Codex about your workflows and active projects. Before enabling Telepathy, be mindful of the following considerations:

**Cost bullet** (bolded label in UI)

> **Cost**: Telepathy runs agents in the background, which will consume rate limits faster.

**Prompt-injection bullet**

> **Prompt injection**: Telepathy ingests screen contents into your Codex session. Telepathy agents are sandboxed to prevent direct injections; however, in rare cases screen information might affect later Codex agents.

**Storage bullet**

> Screen captures are ephemeral and will not be saved. **Telepathy does not have access to your microphone or system audio.** Generated memories will be stored locally on your computer.

**Disable-scenarios intro**

> You can disable Telepathy at any time, which will stop screen captures. Disable Telepathy in the following scenarios:

- *When taking meetings on your computer, except in accordance with applicable laws and policies*
- *When viewing any content you would prefer not be remembered in memories*

**Permissions notice**

> Telepathy requires Screen Recording permissions on macOS. You will be prompted to grant permissions after enabling Telepathy. If you choose not to grant permissions, Telepathy will not function.

**Buttons**: `Cancel` / `Continue`

**Post-enable menu-bar UI** (discovered via the DA/NL locales):

> You can pause Telepathy at any time by clicking "Pause Telepathy" in the Codex menu bar.

---

## 4. Comparison to Anthropic's "Chicago"

| Axis | OpenAI Telepathy | Anthropic Chicago |
|---|---|---|
| Internal codename | `codex_tape_recorder`, `Telepathy` | `Chicago`, sub-gates `chicago_config.*` |
| Public/exp status | Research preview, consent UI shipped | Experimental, no UI |
| Gate | Statsig `2574306096` | GrowthBook `1291166712` |
| Platform | macOS only | macOS only (Windows fenced) |
| Activation | Sidecar binary + Statsig + user consent | GrowthBook + user pref + dev override flag |
| Screen read mechanism | Separate sidecar process | In-process ScreenCaptureKit via `claude-native-binding.node` |
| Ambient injection | *Memories* — generated and stored locally, referenced in later turns | `<cu_window_hints>` tag prepended to the next user message |
| Hide-before-action | Not present (Telepathy is passive) | Present — auto-hide Claude's own window |
| Kill switch | Menu-bar "Pause Telepathy" | Per-turn auto-unhide of apps |
| Consent UI | Full dialog with prompt-injection disclosure | None (dev-only flag in current build) |
| Sandboxing claim | "Telepathy agents are sandboxed" — likely runs in the Owl runtime | Runs inline in the Electron main process |

The headline difference: Telepathy records *continuously* and builds a *local memory store* that is referenced in subsequent Codex sessions. Chicago, by contrast, only observes when the user "points at" a window. Telepathy is the heavier-weight design.

---

## 5. The missing piece — `codex_telepathy`

**This binary is not in the current bundle.** Expected location: `/Applications/Codex.app/Contents/Resources/codex_telepathy`.

Likely destinations, based on the update URL pattern:

- `https://oaisidekickupdates.blob.core.windows.net/mac/telepathy/alpha/…`
- Or delivered as part of the **Owl** primary-runtime bundle (`/owl/latest/`), which *is* an independently updateable payload fetched from `oaisidekickupdates.blob.core.windows.net/owl`.

Once the binary ships and Statsig gate `2574306096` flips for a user, the flow is:

1. Codex reads the gate value on startup.
2. If enabled and `codex_telepathy` exists, the settings UI shows a "Telepathy" toggle.
3. User accepts consent → `telepathy-consent-accepted` pref is set.
4. Codex launches the sidecar; the sidecar writes its PID to `$TMPDIR/codex_tape_recorder/telepathy-started.pid`.
5. macOS TCC prompts for Screen Recording.
6. Sidecar begins capturing; the main app polls for state via `telepathy-permissions`.
7. Captures are summarized into "memories" stored locally (see storage bullet above).
8. Subsequent Codex sessions read those memories and can reference them in answers.

---

## 6. How to know it's live (when it ships)

In `~/Library/Preferences/com.openai.chat.StatsigService.plist`, look for an entry keyed on gate `2574306096`. Value `true` means the sidecar binary, if present, will be launched.

Or, more reliably, watch for the file `/tmp/codex_tape_recorder/telepathy-started.pid` to appear.

---

## 7. Privacy / regulatory read

A few things jump out, useful for an article:

1. **Explicit prompt-injection disclosure**. The consent copy literally tells the user that screen content could affect later agents. Legally defensive and unusually candid.
2. **No mic or system audio.** Stated explicitly. Only visible screen content.
3. **Local memory storage.** Generated memories stored "locally on your computer" — not the raw captures, which are ephemeral. This distinction matters for GDPR/AVG: the raw capture is transient; the derived memory is a personal data record.
4. **Meetings carve-out.** The text advises users to pause Telepathy during meetings "except in accordance with applicable laws and policies" — shifts liability to the user for compliance with Dutch / EU wiretapping rules.
5. **Not sandboxed from the rest of Codex.** The consent says "Telepathy agents are sandboxed" but this refers to the *agent* that reasons over memories, not the capture itself. The capture runs as a native macOS process with full Screen Recording rights.

---

## 8. Source references (by character offset)

| File | Offset | What |
|---|---|---|
| `product-name-….js` | 238,949 | `TELEPATHY_CONSENT_ACCEPTED`, `TELEPATHY_SETUP_COMPLETION_PENDING`, `AMBIENT_SUGGESTIONS_ENABLED` pref keys |
| `product-name-….js` | 412,966 | `gS = "codex_telepathy"` sidecar name; `_S()` discovery; windows destination path (currently dead) |
| `product-name-….js` | 477,427 | `$TMPDIR/codex_tape_recorder/telepathy-started.pid`, env var `CODEX_TELEPATHY_STARTED_PID_PATH` |
| `main-CUDSf52Z.js` | 145,114 | `telepathy-permissions` IPC handler; FSM states |
| `worker.js` | 640,654 | Mirrored pref-key constants (same as above, in the worker bundle) |
| `general-settings-BZQqrI-r.js` | whole file | English consent copy via `defaultMessage` literals |
| `browser-sidebar-comment-preload.js` | ~471,613, ~472,493 | Multi-language consent copy (47+ locales) |

---

*Document generated 2026-04-19 from Codex.app v26.415.40636 (build 1799). Offsets will change on newer builds.*
