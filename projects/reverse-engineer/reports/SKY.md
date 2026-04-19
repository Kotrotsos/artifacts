# Sky — the separately-signed Computer Use service hiding inside Codex

Reverse-engineered from Codex.app v26.415.40636 (build 1799), `com.openai.codex`.  
Source: `/Applications/Codex.app/Contents/Resources/app.asar`, `.vite/build/main-CUDSf52Z.js`, `.vite/build/product-name-C630vpQ6.js`.

---

## One-line summary

Inside Codex there is a second, differently-signed macOS application named **Sky** (`com.openai.sky.CUAService`) that does Computer Use. It has its own Sparkle auto-update feed, its own EdDSA signing key, runs on an "alpha" channel while Codex ships on production, and exposes an App-Group-Container approval store that Codex reads from. Sky is the CUA backend. It did not come with Codex — Codex talks to it as if it were an independent product.

---

## 1. Identity

| Property | Value |
|---|---|
| Bundle ID | **`com.openai.sky.CUAService`** |
| Apple Team prefix | `2DC432GLL2` (shared with Codex) |
| Team-prefixed identifier | `2DC432GLL2.com.openai.sky.CUAService` |
| Product name | Sky (internal), surfaces as "Computer Use" in UI |
| Sparkle public key | `5Yw9jMXMH6O3mJZmpFuQT6ECfC3ZKBfVjWUVMNrElRo=` |
| Update feed URL | `https://oaisidekickupdates.blob.core.windows.net/mac/cua/alpha/appcast.xml` |
| Release channel | **alpha** (while outer Codex is on prod) |
| macOS entitlements | AppleScript/AppleEvents automation, ScreenCaptureKit, AXUIElement |
| Sandbox status | **Sandboxed** (Codex itself is not) |
| Distribution | Shipped inside Codex.app bundle OR installed alongside |

---

## 2. How Codex locates Sky

### 2a. Direct bundle ID lookup

From `main-CUDSf52Z.js:76172`:

```js
var Yi = `com.openai.sky.CUAService`;
var Xi = `computerUseSoundMode`;

async function Zi() {
  if (process.platform !== `darwin`) return null;
  let { stdout } = await execFile(`/usr/bin/defaults`,
    [`read`, Yi, Xi]);   // read Sky's preference
  return stdout.trim() || null;
}
async function Qi(e) {
  if (process.platform === `darwin`)
    await execFile(`/usr/bin/defaults`,
      [`write`, Yi, Xi, e]);   // write to Sky's preference
  return e;
}
```

Codex reads and writes a preference key (`computerUseSoundMode`) directly on Sky's bundle using `/usr/bin/defaults`. This is how two signed apps share settings without a network call.

### 2b. Shared app-group container for approvals

From `main-CUDSf52Z.js:73515`:

```js
var Ti = `ComputerUseAppApprovals.json`;
var Ei = join(`Library`, `Application Support`, `Software`);
var Di = join(`Library`, `Group Containers`);
var Oi = `2DC432GLL2.com.openai.sky.CUAService`;   // app-group identifier

var ki = zi({
  approvedBundleIdentifiers: Pi(Vi())
});   // schema: array of bundle IDs the user approved

async function Mi(homedir) {
  let t = Ii(homedir);
  if (!existsSync(t.filePath)) return null;
  let r = Ri(await Li(t.filePath));
  return {
    approvedApps:                await zi(r),
    approvedBundleIdentifiers:   r
  };
}

async function Ni(homedir) {
  return existsSync(Ii(homedir).filePath);
}
```

Translation: Sky and Codex share an Apple App-Group container at:

```
~/Library/Group Containers/2DC432GLL2.com.openai.sky.CUAService/ComputerUseAppApprovals.json
```

This file holds the list of macOS bundle IDs that the user has approved for Computer Use. Both apps read it. The schema is a simple JSON array of strings.

### 2c. Finder integration marker

From `product-name-….js:731538`:

```js
var jk = [
  `com.apple.finder.Open-Computer`,
  `com.openai.sky.CUAService`
];
```

These two bundle IDs are treated as "app open" events — Codex's recent-apps logic knows about Sky specifically and skips it from lists.

---

## 3. The update topology — Sidekick umbrella

Sky is part of a three-product family that all auto-update from the same Azure blob storage endpoint:

| Product | Codename | Update URL | Channel | Signing key |
|---|---|---|---|---|
| Codex (this app) | — | `oaisidekickupdates.blob.core.windows.net/mac/codex/…/appcast.xml` | prod | `rhcBvttuqDFriyNqwTQJR3L4UT1WjIK4QxtwtwusVic=` |
| **Sky** (Computer Use) | Sky | `oaisidekickupdates.blob.core.windows.net/mac/cua/alpha/appcast.xml` | **alpha** | `5Yw9jMXMH6O3mJZmpFuQT6ECfC3ZKBfVjWUVMNrElRo=` |
| Owl (runtime) | Owl | `oaisidekickupdates.blob.core.windows.net/owl/latest/<os>-<arch>/…` | latest / latest-alpha | different, handled inside Codex main bundle |

From `product-name-….js:853427`:

```js
function BP(e, t, n) {
  return [
    (t.baseUrl ?? (n === `latest`
      ? FP
      : `https://oaisidekickupdates.blob.core.windows.net/owl`
    )).replace(/\/+$/, ``),
    NP,
    ...(n === `latest-alpha` ? [`alpha`] : []),
    `latest`,
    zP(e),   // <os>-<arch>
    PP
  ].join(`/`);
}
```

This is the URL builder for **Owl** updates. Owl is the bundled Node runtime and skills package — it ships `node`, `node_repl`, and the skill library as a separately-updateable blob.

"Sidekick" is the family codename. The origin `oaisidekickupdates.blob.core.windows.net` is common across all three; the prefix paths fork by product.

### 3a. Why alpha while the outer app is prod?

A CUA backend moves fast. Decoupling its release channel from the main app means Sky can ship weekly fixes without requiring Codex to rebuild. It also means a single corrupted Sky update could silently affect Codex users even on production — they share an automation path but not a release channel.

---

## 4. Capabilities — what Sky actually does

Inferred from entitlements and IPC handlers:

| Capability | Evidence |
|---|---|
| Screenshot capture | ScreenCaptureKit + `computer-use-app-approvals-*` IPC; requires bundle-ID allowlist |
| AX tree inspection | AXUIElement entitlement; used to find clickable targets without needing pixels |
| Synthetic mouse / keyboard | Accessibility API; injected into the frontmost-then-allowed app |
| AppleScript / AppleEvents | `automation.apple-events` entitlement |
| App launch | `open_application` tool (shared with Codex) |
| Sound-mode toggle | `computerUseSoundMode` pref — Sky controls whether it chirps on action |

Sky is signed and sandboxed. Codex is signed but *not* sandboxed. This split lets Codex do arbitrary local filesystem work while Sky operates within a TCC-hardened boundary. If Sky gets compromised, it still can't write to `~/Documents`; if Codex gets compromised, it can, but it can't take screenshots without asking Sky. Clean blast-radius separation.

---

## 5. The IPC surface Codex uses to talk to Sky

From `main-CUDSf52Z.js:145200–145450`, these handlers exist in the main process:

```
computer-use-app-approvals-visibility    → boolean (approval store exists?)
computer-use-app-approvals-read          → { approvedApps, approvedBundleIdentifiers }
computer-use-app-approval-remove         → (bundleIdentifier) → void
computer-use-sound-mode-read             → { value: string|null }
computer-use-sound-mode-write            → (value) → void
```

No direct "click", "type", or "screenshot" IPC in *this* layer — Codex doesn't drive Sky directly. Instead, when a Codex session needs Computer Use, it hands off to the `computerUse` plugin (see `PLUGINS.md`), which in turn communicates with Sky through XPC or an Apple Event. The handlers above are the *management* surface: approvals and sound mode.

---

## 6. What Sky's approvals file actually contains

The shared JSON at `~/Library/Group Containers/2DC432GLL2.com.openai.sky.CUAService/ComputerUseAppApprovals.json` holds:

```json
{
  "approvedBundleIdentifiers": [
    "com.apple.Safari",
    "com.microsoft.VSCode",
    "com.tinyspeck.slackmacgap",
    ...
  ]
}
```

One array. The user approves bundles either through Sky's own prompt or through Codex's UI, and both apps respect the same list.

**Operational gotcha**: if the user reinstalls Codex but keeps Sky, their approvals survive (they live in the shared container). If they reinstall both, they must re-approve everything.

---

## 7. Comparison to Anthropic's Computer Use

| Axis | OpenAI Sky | Anthropic Claude Desktop |
|---|---|---|
| Architecture | Separate sandboxed app with own signing, own updater | In-process, same Electron main process |
| Approval storage | Shared App-Group container (both apps see it) | Per-app TCC + in-app dev pref (`chicagoUserDeniedBundleIds`) |
| Bundle ID trick | Separate signed identity (`com.openai.sky.CUAService`) gets its own TCC grants cleanly | A helper called `disclaimer` uses `responsibility_spawnattrs_setdisclaim` to spoof distinct identities for children |
| Release cadence | Sky on `alpha`, Codex on prod, independent | Locked — ships with the Claude app version |
| Input injection | Apple Accessibility API | Rust `enigo` compiled as a `.node` module |
| Screen capture | ScreenCaptureKit (via Sky) | ScreenCaptureKit (via `ScreenshotForComputerUse` C++ wrapper) |
| Screenshot hygiene | Sky captures without the outer Codex window (it's a separate app) | `hideBeforeAction: true` — Claude hides its own window before the shot |
| Sandbox | Yes (Sky) | No (Claude main) |
| AX entitlement | Yes (Sky) | Yes (via `disclaimer` helper) |
| User-visible name | "Computer Use" | "Computer Use" / Teach Mode |

The separate-signed-app design is architecturally cleaner than Anthropic's disclaimer helper trick. It also means an enterprise admin can push an MDM policy against `com.openai.sky.CUAService` specifically, without affecting Codex.

---

## 8. Source references (by character offset)

| File | Offset | What |
|---|---|---|
| `main-CUDSf52Z.js` | 73,515 | App-group container path, schema, read/write helpers |
| `main-CUDSf52Z.js` | 76,157 | `com.openai.sky.CUAService` direct defaults read/write; `computerUseSoundMode` key |
| `main-CUDSf52Z.js` | 145,200 | IPC handlers for approvals and sound mode |
| `product-name-….js` | 731,538 | Finder-open event handling including Sky bundle ID |
| `product-name-….js` | 853,427 | Owl update-URL builder (the Sidekick umbrella origin) |
| Bundle binary (strings) | — | Sparkle public keys for Codex and Sky |
| `Info.plist` | — | Apple Team ID `2DC432GLL2` confirming shared provisioning |

---

## 9. What's interesting for an article

1. **Two separately signed apps on one disk, talking through a shared app-group container.** Clean engineering; rare in consumer apps. Typical of Apple system services more than third-party software.
2. **Different release channels on purpose.** Sky rides the alpha track inside a prod product. If you want OpenAI's freshest CUA fixes, you already have them.
3. **The `disclaimer` trick vs. separate-app design.** Worth contrasting: Anthropic worked around Apple's single-app TCC grant by spoofing child identities; OpenAI solved the same problem by shipping a second real app.
4. **"Sidekick" as a product family.** The URL `oaisidekickupdates.blob.core.windows.net` reveals OpenAI's internal product taxonomy: Codex, Sky (CUA), Owl (runtime). Sidekick is the umbrella.
5. **Sound mode as a shared setting.** Small detail, but telling: the two apps need to agree on whether Computer Use makes noise. The design decision was "share a preference" rather than "duplicate state".

---

*Document generated 2026-04-19 from Codex.app v26.415.40636 (build 1799). Offsets will change on newer builds.*
