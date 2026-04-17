# Chicago — Anthropic's ambient / passive Computer-Use mode

Reverse-engineered from Claude for Desktop v1.2773.0 (`com.anthropic.claudefordesktop`).  
Source: `/Applications/Claude.app/Contents/Resources/app.asar` → `.vite/build/index.js` (9.5 MB minified).

All code excerpts are verbatim from that file. Line-position references use character offsets because the bundle is minified to ~2 lines.

---

## One-line summary

Chicago is the macOS-only feature gate that turns Claude into a screen-aware agent: auto-hiding its own window to take clean screenshots, animating the mouse so the user can watch, guarding the clipboard, and injecting hints about the windows the user "pointed at". Behind the same gate sits **Teach Mode** (user demonstrates, model replays). It is off by default and rolled out via GrowthBook; a developer flag can force it on immediately.

---

## 1. Identity

| Property | Value |
|---|---|
| Codename | **Chicago** |
| GrowthBook feature ID | `1291166712` (`chicago_config`) |
| Secondary GrowthBook ID | `2486083521` (related gate; see §9) |
| Dev-override flag | `isChicagoEnabled` in `~/Library/Application Support/Claude/developer_settings.json` |
| Dev-override label | *"Forces chicago_config.enabled and teachModeEnabled regardless of GrowthBook"* |
| Dev-override `requiresRestart` | `true` |
| Companion feature | Teach Mode (`teachModeEnabled`) — same gate forces both |
| Platforms | **macOS only**. Windows is explicitly excluded: `process.platform !== "win32"` guard, and `w2()` returns `false` outside the `ele` platform allow-set |
| Default state | **off** (`chicagoEnabled: false` in the user prefs schema) |

---

## 2. Sub-gates and their defaults

Chicago reads a config object from GrowthBook (`us(kO, key, default, zod)`). Defaults in code:

```js
const YSn = {
  pixelValidation:          false,  // verify target pixel before clicking
  clipboardPasteMultiline:  true,   // paste multi-line text directly
  mouseAnimation:           true,   // animate the cursor so the user can watch
  hideBeforeAction:         true,   // hide Claude window before action (clean screenshots)
  autoTargetDisplay:        true,   // auto-pick the correct monitor
  clipboardGuard:           true,   // stash/restore clipboard around actions
};
const XSn = "pixels";               // coordinateMode; alt: "normalized_0_100"
const eCn = 1800 * 1000;            // dispatchCuGrantTtlMs — 30-minute permission TTL
```

The live values come from GrowthBook and are logged at startup:

```
[chicago] GrowthBook chicago_config: raw=%j → enabled=%s subGates=%j coordMode=%s dispatchTtlMs=%d
```

## 3. Enable/disable logic

```js
const kO  = "1291166712";
const ZSn = "2486083521";

function ytr() { return false; }                            // internal dev stub
function Ile() { return ytr() ? true : us(kO, "enabled", false, Yt()); }
function w2()  {
  return ele.has(process.platform)                          // macOS only
    ? Ile() && Br("chicagoEnabled")                         // GrowthBook + user pref
    : false;
}
function UFe() {
  return ele.has(process.platform) && Ile() && !Br("chicagoEnabled");
}
function tCn() {
  if (UFe() && process.platform !== "win32")
    return _tr() ? "stub" : "settings";
}
function _tr() {
  return Wr(ZSn) ? ele.has(process.platform) && Ile() : w2();
}
```

- `w2()` is the main "is Chicago currently active?" check. Both the GrowthBook experiment and the user's `chicagoEnabled` pref must be on, and platform must be macOS.
- `UFe()` is "Chicago *could* run but user hasn't opted in" — drives the onboarding prompt.

---

## 4. User-facing preferences (stored in `config.json`)

```js
{
  // Schema snippet from the code bundle
  chicagoEnabled:            Yt().optional(),   // boolean, default false
  chicagoAutoUnhide:         Yt().optional(),   // boolean, default true
  chicagoUserDeniedBundleIds: tr(Pe()).optional(), // string[], apps the user blocked
  remoteToolsDeviceName:     Pe().optional(),
}
```

Default record pulled from the bundle:

```js
{
  chicagoEnabled:             false,
  chicagoAutoUnhide:          true,
  chicagoUserDeniedBundleIds: [],
}
```

---

## 5. Developer settings schema

```js
// Validator for ~/Library/Application Support/Claude/developer_settings.json
const SRr = jt({
  isDxtEnabled:           Yt().optional(),
  isDxtDirectoryEnabled:  Yt().optional(),
  isLocalDevMcpEnabled:   Yt().optional(),
  isUvSystemPythonEnabled:Yt().optional(),
  isMidnightOwlEnabled:   Yt().optional(),
  isChicagoEnabled:       Yt().optional(),
}).catchall(Yt()).optional();
```

Each flag has a user-visible label and sublabel:

| Flag | Sublabel |
|---|---|
| `isDxtEnabled` | Allows loading browser extensions in the app |
| `isDxtDirectoryEnabled` | Enables the extensions directory feature |
| `isLocalDevMcpEnabled` | Allows local development MCP servers |
| `isUvSystemPythonEnabled` | Uses system Python instead of downloading managed Python for UV runtime extensions |
| `isMidnightOwlEnabled` | Enables midnightOwl prototype |
| `isChicagoEnabled` | **Forces chicago_config.enabled and teachModeEnabled regardless of GrowthBook** (requires restart) |

---

## 6. Ambient context mechanism (how Claude "watches")

Chicago does **not** continuously poll the screen. Instead, when Chicago is on, the app does two things:

### 6a. "Point-at" window injection

```js
// Renderer -> main IPC
noteCuWindowMentions(sessionId, windows) {
  if (!w2()) return;                                // gated on Chicago
  this.sessions.get(sessionId).cuMentionedWindows = windows;
}
```

Before the next user message is sent to the model, the app prepends a hint tag:

```js
appendCuWindowHint(session, message) {
  if (!session.cuMentionedWindows?.length) return message;
  const hints = session.cuMentionedWindows
    .map(w => `window "${w.title}" (already open; pass ${w.bundleId} to request_access)`)
    .join(", ");
  session.cuMentionedWindows = undefined;
  return `${message}  <cu_window_hints>The user is pointing at: ${hints}. \
Take a screenshot to find it — do not open_application for it.</cu_window_hints>`;
}
```

Claude sees a tag it can act on. It doesn't call a tool to discover the window — the app tells it.

### 6b. Clean-screenshot choreography

When Claude decides to take an action:

```js
async function lA(t, session, cfg, capability) {
  if (cfg.hideBeforeAction) {
    const hidden = await t.executor.prepareForAction(
      session.allowedApps.map(a => a.bundleId),
      session.selectedDisplayId
    );
    if (hidden.length > 0) session.onAppsHidden?.(hidden);
    // hidden bundle IDs are tracked in session.cuHiddenDuringTurn
  }
  // ...screenshot/click here...
}
```

At end-of-turn, anything in `cuHiddenDuringTurn` is auto-unhidden if `chicagoAutoUnhide` is on:

```js
const hidden = session.cuHiddenDuringTurn;
if (hidden?.size > 0 && Br("chicagoAutoUnhide")) {
  SRe([...hidden]).catch(err => 
    T.warn("[computer-use] auto-unhide at turn end failed", err)
  );
}
```

---

## 7. Per-session state (lifecycle)

| Field | Purpose |
|---|---|
| `cuHiddenDuringTurn` | `Set<bundleId>` of apps hidden this turn; auto-unhidden at end |
| `cuHiddenPendingNote` | Deferred note about hidden apps to show the user |
| `cuMentionedWindows` | Windows the user pointed at; consumed once, then cleared |
| `cuClipboardStash` | Clipboard contents saved before action, restored at end-of-turn |
| `teachModeActive` | Whether the session is currently in Teach Mode |
| `teachModeEnteredAt` | Timestamp for `cu_teach_session` duration telemetry |

---

## 8. Tool surface

### 8a. Computer Use tool actions (what the LLM can invoke)

```
enum: [
  "left_click", "right_click", "type", "screenshot", "wait",
  "scroll", "key", "left_click_drag", "double_click", "triple_click",
  "middle_click", "mouse_move", "cursor_position", "hold_key", "zoom"
]
```

### 8b. Internal MCP-style server exposed to the model

```js
{
  getMouseAnimationEnabled:  () => CX().mouseAnimation,
  getHideBeforeActionEnabled:() => CX().hideBeforeAction,
  hostBundleId:              SCn(),      // which Electron helper
  ensureOsPermissions:       oPe,         // prompts for TCC if missing
  isDisabled:                () => !w2(),
  getAutoUnhideEnabled:      () => Br("chicagoAutoUnhide") ?? true,
  getSubGates:               CX,
  cropRawPatch: (b64, rect) => {
    const img = Ee.nativeImage.createFromBuffer(Buffer.from(b64, "base64"));
    return img.crop(rect).toBitmap();
  },
  listInstalledApps,        // list of installable app bundles on this macOS
  getUserDeniedBundleIds:   () => Br("chicagoUserDeniedBundleIds"),
  getSelectedDisplayId,
  getDisplayPinnedByModel,
  getDisplayResolvedForApps,
  getTeachModeActive,
  getLastScreenshotDims,
  onPermissionRequest,
  onTeachPermissionRequest,
}
```

### 8c. Permission grant (`request_access`)

The LLM calls `request_access` with an array of app bundle IDs. The user approves; the grant lasts 30 minutes (`dispatchCuGrantTtlMs`). Subsequent actions against those apps no longer prompt.

Frozen fragment of the guidance the LLM sees in its system prompt:

> "Cursor clicks or double-click/right-click/hover/drag/enter on desktop items can launch applications outside the allowlist. To interact with the desktop, taskbar, Start menu, Search, or file manager, call `request_access` with exactly \"File Explorer\" (Windows) or \"Finder\" (macOS) in the apps array — that single grant covers all of them. To interact with a different app, use `open_application` to bring it forward."

### 8d. IPC surface (renderer ↔ main)

All calls are namespaced under `claude.web_$_ComputerUseTcc_$_…` and `claude.web_$_LocalAgentModeSessions_$_…`. Examples:

```
ComputerUseTcc.getState
ComputerUseTcc.listInstalledApps
ComputerUseTcc.revokeGrant
LocalAgentModeSessions.noteCuWindowMentions
LocalAgentModeSessions.triggerVertexAuth
WindowControl.captureScreenshot
```

Each handler calls `rt(senderFrame)` for origin validation — calls from non-`claude.ai` frames are rejected.

---

## 9. Telemetry events

Chicago/Teach emits:

```
cu_tool_call           { session_id, session_type, user_message_uuid,
                         tool_name, is_error, error_kind, duration_ms,
                         is_teach_mode, coordinate_mode }
cu_teach_session       { session_id, session_type, duration_ms, exit_trigger }
teachModeChanged       { sessionId, active }
teachStepRequested     (internal)
teachStepWorking       (internal)
```

Runtime logs:

```
[chicago] GrowthBook chicago_config: raw=… → enabled=… subGates=… coordMode=… dispatchTtlMs=…
[computer-use] auto-unhide at turn end failed
[computer-use] clipboard restore at turn end failed
[computer-use] screenshot implausibly small (<1024 bytes), retrying once
```

---

## 10. How to enable it locally (macOS)

Chicago is **off by default**. Two paths:

### Option A — Dev flag (immediate, regardless of GrowthBook)

1. Quit Claude fully (`cmd-Q`).
2. Edit `~/Library/Application Support/Claude/developer_settings.json`:

    ```json
    {
      "allowDevTools": true,
      "isChicagoEnabled": true
    }
    ```

3. Launch Claude. The override "Forces chicago_config.enabled and teachModeEnabled". Teach Mode becomes visible.
4. You still need to toggle `chicagoEnabled` in the user prefs (step B) for the full runtime path.
5. To revert: remove the key or set it to `false`, quit/relaunch.

### Option B — User preference (normal path, works once Chicago rolls out via GrowthBook)

Add to `~/Library/Application Support/Claude/config.json`:

```json
{
  "chicagoEnabled": true,
  "chicagoAutoUnhide": true,
  "chicagoUserDeniedBundleIds": []
}
```

Relaunch Claude. If GrowthBook hasn't rolled Chicago to your account yet, `w2()` still returns `false` unless Option A is also set.

### macOS permissions you'll be asked to grant

On first use Chicago will trigger these TCC prompts (the "disclaimer" helper is what makes the prompts show up with per-tool identity):

- **Screen Recording** (for screenshots)
- **Accessibility** (for synthetic mouse and keyboard events via the Rust `enigo` binding)
- optionally **Automation / AppleEvents** (for `open_application`, `prepareForAction`)

### Sanity check that Chicago is live

With DevTools enabled (`allowDevTools: true` was already in your file), open Claude, open DevTools, and look for this line in the logs:

```
[chicago] GrowthBook chicago_config: raw={...} → enabled=true subGates={...} coordMode=pixels dispatchTtlMs=1800000
```

If `enabled=true`, Chicago is on. If it says `enabled=false` despite the dev flag, restart may have been missed.

---

## 11. Caveats

- **Experimental, subject to change.** The flag name, user prefs, and sub-gate defaults changed between recent builds — your results will vary on a later version.
- **macOS only.** Nothing here runs on Windows. The Chicago path is explicitly fenced off for `process.platform === "win32"`.
- **Requires GrowthBook OR the dev flag.** The user pref alone does nothing; `w2()` needs `Ile()` true.
- **Chicago forces Teach Mode on too.** You cannot enable Chicago without also enabling Teach Mode via the dev override path. Separately, the user pref `teachModeEnabled` exists but is not currently documented.
- **TCC grants happen once per app.** Revoke from *System Settings → Privacy & Security → Screen Recording / Accessibility → Claude*.
- **30-minute grant TTL.** Permission grants auto-expire after 30 min of inactivity (`dispatchCuGrantTtlMs = 1800000`).
- **`louderPenguinEnabled`, `proactive` flag, `midnightOwl`, `operon`, `cowork`** are adjacent but not Chicago. See `claude-app-analysis/FEATURES.md` for those.

---

## 12. Source references (by character offset in `index.js`)

| Offset | What |
|---|---|
| ~39,254 | Default user-prefs record including `chicagoEnabled`, `chicagoAutoUnhide`, `chicagoUserDeniedBundleIds` |
| ~1,011,461 | Zod schema for user prefs |
| ~1,014,693 | Zod schema and labels for developer settings (including `isChicagoEnabled`) |
| ~4,299,186 | GrowthBook IDs, sub-gate defaults, `Ile()`/`w2()`/`UFe()`/`_tr()`/`tCn()` gating functions |
| ~4,299,906 | `CX()` reads each sub-gate from GrowthBook |
| ~4,312,606 | Chicago's internal MCP-style server surface and executors |
| ~4,314,394 | `getAllowedApps` / `getUserDeniedBundleIds` wiring into Computer Use |
| ~2,750,302 | `hideBeforeAction` implementation, `prepareForAction`, `onAppsHidden` |
| ~2,770,710 | Teach Mode action dispatch using the same hide-before-action flow |
| ~4,805,235 | Auto-unhide on turn end, clipboard restore, `chicagoAutoUnhide` pref read |
| ~8,919,999 | `noteCuWindowMentions` and `appendCuWindowHint` — the `<cu_window_hints>` injection |
| ~477,246 | `ComputerUseTcc.listInstalledApps` IPC registration |

---

*Document generated 2026-04-17 from a fresh extraction of Claude.app v1.2773.0. The app.asar has a SHA-256 integrity hash pinned in `Info.plist`; if you regenerate this document on a newer build, expect offsets and IDs to change.*
