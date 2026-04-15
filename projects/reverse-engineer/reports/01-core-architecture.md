# VS Code Core Architecture - Reverse Engineering Report

**Version analyzed:** 1.111.0
**Build commit:** `ce099c1ed25d9eb3076c11e4a280f3eb52b4fbeb`
**Build date:** 2026-03-06T23:06:10Z
**Electron version:** 39.6.0
**Quality channel:** stable
**Platform:** macOS (darwin)
**Bundle identifier:** com.microsoft.VSCode

---

## 1. Application Bootstrap

### 1.1 Native Binary Entry

The macOS app starts from `Contents/MacOS/Code` (the native Electron binary, aliased alongside `Contents/MacOS/Electron`). The `Info.plist` sets `NSPrincipalClass` to `AtomApplication` (legacy name from Atom Shell / Electron heritage).

### 1.2 JavaScript Entry Point

`package.json` declares:

```
"main": "./out/main.js"
"type": "module"
```

`out/main.js` is the Electron main process entry. In the production build it is a tslib helpers file that re-exports the TypeScript runtime helpers. The actual application code is bundled into large single-file ESM bundles deeper in the tree. This is a build artifact of VS Code's custom bundling pipeline: each process target gets a single concatenated JS file with tslib prepended.

### 1.3 Startup Sequence

The startup flow, traced from the source files:

1. **Electron launches** `out/main.js` in the main (Node.js) process
2. **Main process** (the code formerly in `vs/code/electron-main/main.ts`) initializes:
   - Parses CLI arguments
   - Sets up crash reporting (`crashReporter.productName: "VSCode"`, `companyName: "Microsoft"`)
   - Loads `product.json` and `package.json` for configuration
   - Creates the `CodeApplication` instance
   - Instantiates the dependency injection container (ServiceCollection + InstantiationService)
   - Opens BrowserWindow(s) for each workspace/window
3. **Preload script** (`vs/base/parts/sandbox/electron-browser/preload.js`) runs in each renderer before page load:
   - Exposes a controlled `window.vscode` API via `contextBridge.exposeInMainWorld()`
   - Provides sandboxed IPC: `ipcRenderer` (filtered to `vscode:` prefixed channels only)
   - Provides `ipcMessagePort` for MessagePort acquisition
   - Provides `webFrame` (zoom control), `webUtils` (file path resolution)
   - Provides `process` (platform, arch, env, versions, shell env, memory info)
   - Provides `context.resolveConfiguration()` to fetch window config from main process via IPC
   - Fetches `vscode-window-config` from main process via `ipcRenderer.invoke()`
   - Fetches shell environment via `vscode:fetchShellEnv` IPC channel
   - Sets initial zoom level from configuration
4. **Workbench HTML** (`vs/code/electron-browser/workbench/workbench.html`) loads:
   - Sets strict Content Security Policy (Trusted Types enforced)
   - Loads `workbench.desktop.main.css` stylesheet
   - Loads `workbench.js` as ES module
5. **Workbench bootstrap** (`vs/code/electron-browser/workbench/workbench.js`) executes:
   - Marks `code/didStartRenderer` performance mark
   - Resolves window configuration from preload context (with 10s timeout warning)
   - Sets up developer keybindings (Ctrl+Shift+I / Cmd+Alt+I for DevTools, Ctrl+R / Cmd+R for reload)
   - Configures NLS (internationalization) globals
   - Renders the "parts splash" (cached layout from previous session: title bar, activity bar, sidebar, status bar, auxiliary bar with correct colors and dimensions)
   - Sets up CSS import map for module-based CSS loading
   - Dynamically imports `vs/workbench/workbench.desktop.main.js` (the ~16MB monolithic workbench bundle)
   - Calls `result.main(configuration)` to start the workbench

### 1.4 Performance Marks

The bootstrap uses `performance.mark()` extensively:
- `code/didStartRenderer`
- `code/willShowPartsSplash` / `code/didShowPartsSplash`
- `code/willWaitForWindowConfig` / `code/didWaitForWindowConfig`
- `code/willAddCssLoader` / `code/didAddCssLoader`
- `code/willLoadWorkbenchMain` / `code/didLoadWorkbenchMain`

---

## 2. Process Architecture

VS Code uses a multi-process architecture built on Electron. Each process has a dedicated entry point JS file:

### 2.1 Process Types and Entry Points

| Process | Entry Point File | Purpose |
|---------|-----------------|---------|
| **Main Process** | `out/main.js` | Electron main process. Window management, lifecycle, native APIs |
| **Renderer Process** | `vs/code/electron-browser/workbench/workbench.js` -> imports `vs/workbench/workbench.desktop.main.js` (5,338 lines, ~16MB) | The workbench UI. Each window is a renderer |
| **Extension Host (Node)** | `vs/workbench/api/node/extensionHostProcess.js` | Runs extensions in isolated Node.js process |
| **Extension Host (Worker)** | `vs/workbench/api/worker/extensionHostWorkerMain.js` | Runs web extensions in a Web Worker |
| **Shared Process** | `vs/code/electron-utility/sharedProcess/sharedProcessMain.js` (494 lines) | Singleton utility process shared across windows. Handles extensions gallery, telemetry, update checks |
| **CLI Process** | `vs/code/node/cliProcessMain.js` (462 lines) | Handles `code` CLI commands |
| **PTY Host** | `vs/platform/terminal/node/ptyHostMain.js` | Manages pseudo-terminal processes for the integrated terminal |
| **File Watcher** | `vs/platform/files/node/watcher/watcherMain.js` | File system watching (uses `@parcel/watcher`) |
| **Profile Analysis Worker** | `vs/platform/profiling/electron-browser/profileAnalysisWorkerMain.js` | V8 CPU profile analysis |
| **Editor Web Worker** | `vs/editor/common/services/editorWebWorkerMain.js` | Editor computation offloading |
| **Language Detection Worker** | `vs/workbench/services/languageDetection/browser/languageDetectionWebWorkerMain.js` | Automatic language detection |
| **TextMate Tokenization Worker** | `vs/workbench/services/textMate/browser/backgroundTokenization/worker/textMateTokenizationWorker.workerMain.js` | Background syntax highlighting |
| **File Search Worker** | `vs/workbench/services/search/worker/localFileSearchMain.js` | Local file search |
| **Output Link Computer** | `vs/workbench/contrib/output/common/outputLinkComputerMain.js` | Link detection in output panel |
| **Notebook Web Worker** | `vs/workbench/contrib/notebook/common/services/notebookWebWorkerMain.js` | Notebook computation |
| **Debug Telemetry** | `vs/workbench/contrib/debug/node/telemetryApp.js` | Debug adapter telemetry |
| **Preload (auxiliary)** | `vs/base/parts/sandbox/electron-browser/preload-aux.js` | Preload for auxiliary windows |
| **BrowserView Preload** | `vs/platform/browserView/electron-browser/preload-browserView.js` | Preload for embedded browser views |
| **Sessions Window** | `vs/sessions/sessions.desktop.main.js` (4,956 lines) + `vs/sessions/electron-browser/sessions.js` | Separate "Sessions" window (Copilot agent sessions) |

### 2.2 Inter-Process Communication (IPC)

VS Code uses multiple IPC mechanisms:

**Electron IPC (Main <-> Renderer):**
- All IPC channels are prefixed with `vscode:` and validated in the preload script
- The preload script filters: `if (!e?.startsWith("vscode:")) throw new Error(...)`
- Key channels observed: `vscode:toggleDevTools`, `vscode:reloadWindow`, `vscode:openDevTools`, `vscode:fetchShellEnv`, `vscode:handleChatRequest`, `vscode:openChatSession`

**MessagePort (Renderer <-> Extension Host, Renderer <-> Shared Process):**
- Used for high-throughput communication between processes
- The preload exposes `ipcMessagePort.acquire(channelName, nonce)` for port acquisition
- Ports are transferred via `window.postMessage` with `Transferable` ports

**Named Channels (Service-level IPC):**
- Services are proxied across process boundaries using typed channel abstractions
- Pattern: `ProxyChannel.toService(connection.getChannel(channelName))`
- Observed channel usage for MCP gateway: services registered and consumed across main/shared/remote processes

**Shared Process:**
- Runs as an Electron utility process (`electron-utility`)
- Single instance shared across all windows
- Connected via the main process's channel infrastructure

### 2.3 Sandbox Model

VS Code runs renderers in sandboxed mode:
- `contextIsolation` is enabled
- The preload script is the only bridge between Node.js and the renderer
- `window.vscode` is the sole API surface exposed to the renderer
- Trusted Types are enforced (policies listed in the CSP: `amdLoader`, `defaultWorkerFactory`, `diffEditorWidget`, `dompurify`, `notebookRenderer`, etc.)

---

## 3. Dependency Injection System

VS Code uses a custom dependency injection system. The analysis of the minified workbench bundle reveals:

### 3.1 Core Concepts

**Service Identifiers:**
- Created via `createDecorator<T>(serviceId: string)` which returns a decorator function
- Each service gets a unique string identifier
- The decorator doubles as both a TypeScript type annotation and a runtime service locator key

**InstantiationService:**
- 63 references to `InstantiationService` in the workbench bundle
- Central DI container that resolves service dependencies
- Supports constructor injection via `@IServiceIdentifier` parameter decorators

**ServiceCollection:**
- Map-like container that holds service registrations (identifier -> instance or descriptor)
- Built up during startup with platform services, then passed to InstantiationService

### 3.2 Registration Pattern

From the minified code, the registration pattern is visible:

```javascript
// Minified form observed:
Ce(ServiceInterface, ImplementationClass, instantiationType)
// Where instantiationType: 0 = Eager, 1 = Delayed

// Workbench contribution registration:
yt(ContributionClass.ID, ContributionClass, InstantiationType)
// Where InstantiationType corresponds to lifecycle phases:
// It.AfterRestored, It.Eventually, It.BlockRestore
```

**Observed registration examples from the bundle:**
```javascript
Ce(T4, new Nt(XOt, [[]], true));           // Service with descriptor
Ce(fAe, A2e, 1);                            // Delayed instantiation
Ce(HTe, O2e, 1);                            // Delayed instantiation
yt("mcpGatewayToolBrokerLocal", F2e, It.AfterRestored);  // Workbench contribution
yt(Bk.ID, Bk, It.AfterRestored);
yt(tbe.ID, tbe, It.AfterRestored);
yt(L2e.ID, L2e, It.BlockRestore);
yt(R2e.ID, R2e, It.AfterRestored);
yt(M2e.ID, M2e, It.AfterRestored);
```

### 3.3 Parameter Decoration (Constructor Injection)

The `__param` and `__decorate` tslib helpers are used at runtime. In the minified code, service injection appears as:

```javascript
// Minified pattern (I = __decorate, m = __param):
ClassName = I([m(0, ServiceId1), m(1, ServiceId2), ...], ClassName);
```

For example:
```javascript
tbe = I([m(0,zn),m(1,F),m(2,kD),m(3,ve),m(4,le),m(5,hn),m(6,it)], tbe);
```

This means the class `tbe` has 7 constructor parameters, each injected with a different service.

### 3.4 Lifecycle Phases

Workbench contributions are registered with lifecycle timing:
- `It.BlockRestore` - Must complete before workspace restore
- `It.AfterRestored` - Instantiated after the workspace is restored
- `It.Eventually` - Instantiated lazily, when idle

---

## 4. Window Management

### 4.1 Window Creation

- Each VS Code window is an Electron `BrowserWindow`
- The main process manages window lifecycle (open, close, focus, bounds)
- Each window loads `vs/code/electron-browser/workbench/workbench.html`
- Window configuration is passed from main process to renderer via the `vscode-window-config` IPC channel

### 4.2 Window Configuration

The preload script fetches window configuration which includes:
- `windowId` - Unique window identifier (exposed as `window.vscodeWindowId`)
- `zoomLevel` - Initial zoom level
- `nls` - Localization data (messages, language)
- `workspace` - Workspace information
- `partsSplash` - Cached layout for instant splash screen
- `colorScheme` - Dark/light/high-contrast detection
- `autoDetectHighContrast` / `autoDetectColorScheme` - Theme auto-detection
- `extensionDevelopmentPath` - For extension development
- `extensionTestsPath` - For extension testing
- `appRoot` - Application root path
- `cssModules` - CSS module paths for import map setup
- `userEnv` - User environment variables
- `enable-smoke-test-driver` - Smoke test mode

### 4.3 Parts Splash (Instant Startup Feel)

The workbench.js implements an elaborate splash screen from cached layout data:
- Title bar (with `-webkit-app-region: drag`)
- Activity bar (left or right positioned with border)
- Side bar (with configurable width, capped at available space)
- Auxiliary bar (secondary side bar)
- Status bar (different background for workspace vs no-folder)
- All colors are restored from the previous session's theme
- Layout dimensions are adjusted to current window size
- High contrast and color scheme auto-detection override the cache when needed

### 4.4 Sessions Window

VS Code 1.111 includes a separate "Sessions" window type:
- Bundle identifier: `com.microsoft.VSCodeSessions`
- URL protocol: `vscode-sessions`
- Data folder: `.vscode-sessions`
- Dedicated entry points: `vs/sessions/sessions.desktop.main.js` and `vs/sessions/electron-browser/sessions.js`
- Used for GitHub Copilot agent sessions (Background Agent, Cloud Agent, Codex Agent)

---

## 5. product.json - Full Configuration

### 5.1 Product Identity

| Key | Value |
|-----|-------|
| `nameShort` | `Code` |
| `nameLong` | `Visual Studio Code` |
| `applicationName` | `code` |
| `version` | `1.111.0` |
| `commit` | `ce099c1ed25d9eb3076c11e4a280f3eb52b4fbeb` |
| `date` | `2026-03-06T23:06:10Z` |
| `quality` | `stable` |
| `darwinBundleIdentifier` | `com.microsoft.VSCode` |
| `darwinExecutable` | `VSCode` |
| `urlProtocol` | `vscode` |
| `dataFolderName` | `.vscode` |
| `serverDataFolderName` | `.vscode-server` |
| `serverApplicationName` | `code-server` |
| `tunnelApplicationName` | `code-tunnel` |

### 5.2 URLs and Endpoints

| Purpose | URL |
|---------|-----|
| Download | `https://code.visualstudio.com` |
| Update | `https://update.code.visualstudio.com` |
| Web IDE | `https://vscode.dev` |
| Web CDN | `https://main.vscode-cdn.net` |
| Web CDN Template | `https://{{uuid}}.vscode-cdn.net/{{quality}}/{{commit}}` |
| NLS Base | `https://www.vscode-unpkg.net/nls/` |
| Webview Content | `https://{{uuid}}.vscode-cdn.net/{{quality}}/{{commit}}/out/vs/workbench/contrib/webview/browser/pre/` |
| Extensions Gallery | `https://marketplace.visualstudio.com/_apis/public/gallery` |
| Gallery Items | `https://marketplace.visualstudio.com/items` |
| Gallery Publishers | `https://marketplace.visualstudio.com/publishers` |
| Extension Resources | `https://{publisher}.vscode-unpkg.net/{publisher}/{name}/{version}/{path}` |
| Gallery Control | `https://main.vscode-cdn.net/extensions/marketplace.json` |
| MCP Gallery | `https://api.mcp.github.com` |
| MCP Servers List | `https://main.vscode-cdn.net/mcp/servers.json` |
| Gallery NLS | `https://www.vscode-unpkg.net/_lp/` |
| Profile Templates | `https://main.vscode-cdn.net/core/profile-templates.json` |
| Emergency Alerts | `https://main.vscode-cdn.net/core/stable.json` |
| Chat Participants | `https://main.vscode-cdn.net/extensions/chat.json` |
| Settings Search | `https://bingsettingssearch.trafficmanager.net/api/Search` |
| Config Sync | `https://vscode-sync.trafficmanager.net/` |
| Config Sync (Insiders) | `https://vscode-sync-insiders.trafficmanager.net/` |
| Edit Sessions | `https://vscode-sync.trafficmanager.net/` |
| OAuth Login | `https://login.microsoftonline.com/common/oauth2/authorize` |
| OAuth Token | `https://login.microsoftonline.com/common/oauth2/token` |
| OAuth Redirect | `https://vscode-redirect.azurewebsites.net/` |
| Auth Client Metadata | `https://vscode.dev/oauth/client-metadata.json` |
| A/B Testing | `https://default.exp-tas.com/vscode/ab` |
| Documentation | `https://go.microsoft.com/fwlink/?LinkID=533484#vscode` |
| Release Notes | `https://go.microsoft.com/fwlink/?LinkID=533483#vscode` |
| Keyboard Shortcuts (Mac) | `https://go.microsoft.com/fwlink/?linkid=832143` |
| Keyboard Shortcuts (Linux) | `https://go.microsoft.com/fwlink/?linkid=832144` |
| Keyboard Shortcuts (Win) | `https://go.microsoft.com/fwlink/?linkid=832145` |
| Introductory Videos | `https://go.microsoft.com/fwlink/?linkid=832146` |
| Tips and Tricks | `https://go.microsoft.com/fwlink/?linkid=852118` |
| Newsletter Signup | `https://www.research.net/r/vsc-newsletter` |
| YouTube | `https://aka.ms/vscode-youtube` |
| Request Feature | `https://go.microsoft.com/fwlink/?LinkID=533482` |
| Report Issue | `https://github.com/Microsoft/vscode/issues/new` |
| Report Marketplace Issue | `https://github.com/microsoft/vsmarketplace/issues/new` |
| License | `https://go.microsoft.com/fwlink/?LinkID=533485` |
| Server License | `https://aka.ms/vscode-server-license` |
| Privacy Statement | `https://go.microsoft.com/fwlink/?LinkId=521839` |
| NPS Survey | `https://aka.ms/vscode-nps` |
| Checksum Fail Info | `https://go.microsoft.com/fwlink/?LinkId=828886` |
| Server Documentation | `https://aka.ms/vscode-server-doc` |

### 5.3 Telemetry Configuration

| Key | Value |
|-----|-------|
| `enableTelemetry` | `true` |
| `showTelemetryOptOut` | `true` |
| `aiConfig.ariaKey` | `5bbf946d11a54f6783919c455abaddaf-fd62977b-c92d-4714-a45d-649d06980372-7168` |
| A/B Testing endpoint | `https://default.exp-tas.com/vscode/ab` |
| A/B telemetry event | `query-expfeature` |
| A/B assignment context | `abexp.assignmentcontext` |
| Crash reporter product | `VSCode` |
| Crash reporter company | `Microsoft` |

**Telemetry SDKs:** `@microsoft/1ds-core-js` and `@microsoft/1ds-post-js` (Microsoft 1DS/Aria telemetry)

**App Center IDs:**
- macOS (darwin): `860d6632-f65b-490b-85a8-3e72944f7774`
- macOS (darwin-arm64): `be71415d-3893-4ae5-b453-e537b9668a10`
- macOS (darwin-universal): `de75e3cc-e22f-4f42-a03f-1409c21d8af8`
- Windows x64: `a4e3233c-699c-46ec-b4f4-9c2a77254662`
- Windows ARM64: `3712d786-7cc8-4f11-8b08-cc12eab6d4f7`
- Linux x64: `fba07a4d-84bd-4fc8-a125-9640fc8ce171`

### 5.4 Authentication

**OAuth Client ID:** `aebc6443-996d-45c2-90f0-388ff96faa56`

**Copilot/GitHub Authentication:**
- Entitlement URL: `https://api.github.com/copilot_internal/user`
- Token URL: `https://api.github.com/copilot_internal/v2/token`
- MCP Registry: `https://api.github.com/copilot/mcp_registry`
- Sign-up (limited): `https://api.github.com/copilot_internal/subscribe_limited_user`
- Provider scopes: `read:user`, `user:email`, `repo`, `workflow`
- Auth providers: GitHub, GitHub Enterprise, Google, Apple, Microsoft

**Configuration Sync auth providers:**
- GitHub: scopes `user:email`
- Microsoft: scopes `openid`, `profile`, `email`, `offline_access`

**Tunnel auth (Microsoft):** scope `46da2f7e-b5ef-422a-88d4-2a7f9de6a0b2/.default`

### 5.5 Microsoft Internal Domains

The product detects Microsoft internal network membership via these domains:
- `redmond.corp.microsoft.com`
- `northamerica.corp.microsoft.com`
- `fareast.corp.microsoft.com`
- `ntdev.corp.microsoft.com`
- `wingroup.corp.microsoft.com`
- `southpacific.corp.microsoft.com`
- `wingroup.windeploy.ntdev.microsoft.com`
- `ddnet.microsoft.com`
- `europe.corp.microsoft.com`

### 5.6 Trusted Publishers and Domains

**Trusted extension publishers:** `microsoft`, `github`, `openai`

**Link protection trusted domains:**
- `*.visualstudio.com`, `*.microsoft.com`, `aka.ms`
- `*.gallerycdn.vsassets.io`, `*.github.com`
- `login.microsoftonline.com`
- `*.vscode.dev`, `*.github.dev`, `gh.io`
- `portal.azure.com`
- `raw.githubusercontent.com`, `private-user-images.githubusercontent.com`, `avatars.githubusercontent.com`
- `auth.openai.com`

### 5.7 Copilot Access SKUs

The marketplace gallery checks these Copilot entitlement SKUs:
- `copilot_enterprise_seat` / `_quota` / `_multi_quota`
- `copilot_enterprise_seat_assignment` / `_quota` / `_multi_quota`
- `copilot_enterprise_trial_seat` / `_quota`
- `copilot_for_business_seat` / `_quota` / `_multi_quota`
- `copilot_for_business_seat_assignment` / `_quota` / `_multi_quota`
- `copilot_for_business_trial_seat` / `_quota`

### 5.8 Default Chat Agent (Copilot)

GitHub Copilot is deeply integrated as the default chat agent:
- Provider extension: `vscode.github-authentication`
- Extension: `GitHub.copilot` (completions)
- Chat extension: `GitHub.copilot-chat`
- Documentation: `https://aka.ms/github-copilot-overview`
- Terms: `https://aka.ms/github-copilot-terms-statement`
- Privacy: `https://aka.ms/github-copilot-privacy-statement`
- Plans: `https://aka.ms/github-copilot-plans`

**Chat session recommendations:** Codex (OpenAI), Background Agent, Cloud Agent

### 5.9 Built-in Extensions (downloaded at build time)

| Extension | Version | Repository |
|-----------|---------|------------|
| `ms-vscode.js-debug-companion` | 1.1.3 | github.com/microsoft/vscode-js-debug-companion |
| `ms-vscode.js-debug` | 1.110.0 | github.com/microsoft/vscode-js-debug |
| `ms-vscode.vscode-js-profile-table` | 1.0.10 | github.com/microsoft/vscode-js-profile-visualizer |

### 5.10 Tunnel Server Qualities

| Quality | Server Application Name |
|---------|------------------------|
| stable | `code-server` |
| insider | `code-server-insiders` |
| exploration | `code-server-exploration` |

### 5.11 File Integrity Checksums

The product.json contains SHA-256 checksums for critical files:
- `vs/base/parts/sandbox/electron-browser/preload.js`
- `vs/workbench/workbench.desktop.main.js`
- `vs/workbench/workbench.desktop.main.css`
- `vs/workbench/api/node/extensionHostProcess.js`
- `vs/code/electron-browser/workbench/workbench.html`
- `vs/code/electron-browser/workbench/workbench.js`
- `vs/sessions/sessions.desktop.main.js`
- `vs/sessions/sessions.desktop.main.css`
- `vs/sessions/electron-browser/sessions.html`
- `vs/sessions/electron-browser/sessions.js`

### 5.12 Win32 App IDs

| Variant | App ID |
|---------|--------|
| x64 System | `{EA457B21-F73E-494C-ACAB-524FDE069978}` |
| ARM64 System | `{A5270FC5-65AD-483E-AC30-2C276B63D0AC}` |
| x64 User | `{771FD6B0-FA20-440A-A002-3B3BAC16DC50}` |
| ARM64 User | `{D9E514E7-1A56-452D-9337-2990C0DC4310}` |

---

## 6. package.json

### 6.1 Identity

| Key | Value |
|-----|-------|
| `name` | `Code` |
| `version` | `1.111.0` |
| `distro` | `cd72f8f27b485d65c99f5020caa895a5ac5692eb` |
| `author` | Microsoft Corporation |
| `license` | MIT |
| `repository` | `https://github.com/microsoft/vscode.git` |
| `type` | `module` (ESM) |

### 6.2 Key Dependencies

**Runtime:**
- `@anthropic-ai/sandbox-runtime` 0.0.23 (Anthropic sandbox for Copilot agents)
- `@microsoft/1ds-core-js` / `@microsoft/1ds-post-js` (telemetry)
- `@parcel/watcher` (file watching)
- `@vscode/proxy-agent` (proxy support)
- `@vscode/ripgrep` (search)
- `@vscode/spdlog` (logging)
- `@vscode/sqlite3` (storage)
- `@vscode/tree-sitter-wasm` (incremental parsing)
- `@vscode/vscode-languagedetection` (language detection)
- `@xterm/*` (terminal emulation, v6.1.0-beta)
- `node-pty` (pseudo-terminal)
- `katex` (math rendering)
- `kerberos` (enterprise auth)
- `playwright-core` 1.59.0-alpha (browser automation)
- `vscode-oniguruma` / `vscode-textmate` (syntax highlighting)
- `undici` (HTTP client)

**Dev:**
- `electron` 39.6.0
- `typescript` ^6.0.0-dev
- `@typescript/native-preview` ^7.0.0-dev (native TypeScript compiler)

### 6.3 Notable

The `@anthropic-ai/sandbox-runtime` dependency (v0.0.23) indicates integration with Anthropic's sandboxed execution environment, likely for AI agent tool execution within Copilot features.

---

## 7. Info.plist - Application Metadata

### 7.1 Core Properties

| Key | Value |
|-----|-------|
| `CFBundleDisplayName` | `Code` |
| `CFBundleExecutable` | `Code` |
| `CFBundleIconFile` | `Code.icns` |
| `CFBundleIdentifier` | `com.microsoft.VSCode` |
| `CFBundleShortVersionString` | `1.111.0` |
| `CFBundlePackageType` | `APPL` |
| `LSApplicationCategoryType` | `public.app-category.developer-tools` |
| `LSMinimumSystemVersion` | `12.0` (macOS Monterey) |
| `NSPrincipalClass` | `AtomApplication` |
| `NSHumanReadableCopyright` | `Copyright (C) 2026 Microsoft. All rights reserved` |

### 7.2 Build Information

| Key | Value |
|-----|-------|
| `DTCompiler` | `com.apple.compilers.llvm.clang.1_0` |
| `DTSDKBuild` | `25B74` |
| `DTSDKName` | `macosx26.1` |
| `DTXcode` | `2611` |
| `DTXcodeBuild` | `17B100` |

### 7.3 Security and Permissions

| Key | Value |
|-----|-------|
| `NSAppTransportSecurity.NSAllowsArbitraryLoads` | `true` (allows all HTTP) |
| `NSAudioCaptureUsageDescription` | Access to audio capture |
| `NSBluetoothAlwaysUsageDescription` | Access to Bluetooth |
| `NSCameraUsageDescription` | Access to the camera |
| `NSMicrophoneUsageDescription` | Access to the microphone |
| `NSHighResolutionCapable` | `true` |
| `NSSupportsAutomaticGraphicsSwitching` | `true` |
| `NSRequiresAquaSystemAppearance` | `false` (supports dark mode) |
| `NSQuitAlwaysKeepsWindows` | `false` |
| `NSPrefersDisplaySafeAreaCompatibilityMode` | `false` |

### 7.4 Environment

```
LSEnvironment:
  MallocNanoZone = 0
```

(Disables nano malloc zone, likely for compatibility with Electron/V8 memory management)

### 7.5 URL Scheme Registration

- **Scheme:** `vscode`
- **Role:** Viewer
- **Name:** Visual Studio Code

### 7.6 Document Type Associations (CFBundleDocumentTypes)

VS Code registers as editor for ~60 file types with dedicated icons:

**With custom icons:** C (.h), C (.c), Git configs, HTML templates, Windows scripts (.bat/.cmd), Bower, Config files (.config/.ini/.cfg), C++ (.hpp/.cpp), Objective-C (.m), C# (.cs), CSS, Go, HTML, Jade, Java, JavaScript (.js/.mjs/.cjs), JSON, Less, Markdown (.md and 7 variants), PHP, PowerShell, Python, Ruby, SASS/SCSS, SQL, TypeScript, React (.tsx/.jsx), Vue, XML (.xml/.xaml/.csproj etc.), YAML, Shell scripts (bash/zsh/sh and 12 variants), Clojure

**With default icon:** VS Code workspace (.code-workspace), CoffeeScript, CSV, CMake, Dart, Diff, Dockerfile, Gradle, Groovy, Makefile, Lua, Pug, Jupyter (.ipynb), Lockfile, Log, Plain text, Xcode project/workspace, Visual Basic, R, Rust, Restructured Text, LaTeX, F# (.fs/.fsi/.fsx), SVG, TOML, Swift, and a catch-all "Visual Studio Code document" type covering: containerfile, ctp, dot, edn, handlebars, hbs, ml, mli, pl, pm, pod, pp, properties, psgi, rt, t

**Folder handling:** Also registered as editor for `public.folder` (LSItemContentTypes)

### 7.7 ASAR Integrity

```
ElectronAsarIntegrity:
  Contents/Resources/app/node_modules.asar:
    algorithm: SHA256
    hash: 5402b539cb35ae7faf0b123f8ede1f1c16726ddd8073e162914ab107edd91af4
```

---

## 8. Bundled Extensions

VS Code ships with 95 built-in extensions in `Contents/Resources/app/extensions/`:

**Language support:** bat, clojure, coffeescript, cpp, csharp, css, dart, diff, docker, dotenv, fsharp, go, groovy, handlebars, hlsl, html, ini, java, javascript, json, julia, latex, less, log, lua, make, markdown-basics, objective-c, perl, php, powershell, pug, python, r, razor, restructuredtext, ruby, rust, scss, shaderlab, shellscript, sql, swift, typescript-basics, vb, xml, yaml

**Language services:** css-language-features, html-language-features, json-language-features, markdown-language-features, php-language-features, typescript-language-features

**Tools:** configuration-editing, debug-auto-launch, debug-server-ready, emmet, extension-editing, git, git-base, github, github-authentication, grunt, gulp, ipynb, jake, markdown-math, media-preview, merge-conflict, mermaid-chat-features, microsoft-authentication, notebook-renderers, npm, prompt-basics, references-view, search-result, simple-browser, terminal-suggest, tunnel-forwarding

**Themes:** theme-2026, theme-abyss, theme-defaults, theme-kimbie-dark, theme-monokai, theme-monokai-dimmed, theme-quietlight, theme-red, theme-seti, theme-solarized-dark, theme-solarized-light, theme-tomorrow-night-blue

**Debug:** ms-vscode.js-debug, ms-vscode.js-debug-companion, ms-vscode.vscode-js-profile-table

---

## 9. Architecture Summary

```
+-------------------------------------------------------------------+
|                        ELECTRON MAIN PROCESS                       |
|  Entry: out/main.js                                                |
|  - Window management (BrowserWindow creation)                      |
|  - Lifecycle management                                            |
|  - Native OS integration                                          |
|  - IPC hub (all vscode: channels)                                 |
|  - Shared Process spawning                                        |
|  - Extension Host spawning                                        |
+-------------------------------------------------------------------+
       |              |                |               |
       | IPC          | MessagePort    | IPC            | Utility
       v              v                v               v
+-------------+ +-------------+ +--------------+ +------------------+
| RENDERER    | | EXTENSION   | | EXTENSION    | | SHARED PROCESS   |
| (per window)| | HOST (Node) | | HOST (Worker)| | (singleton)      |
| workbench.  | | extension   | | extension    | | sharedProcess    |
| desktop.    | | HostProcess | | HostWorker   | | Main.js          |
| main.js     | | .js         | | Main.js      | |                  |
| (16MB ESM)  | |             | |              | | - Ext gallery    |
|             | | Runs user   | | Runs web     | | - Telemetry      |
| - Editor    | | extensions  | | extensions   | | - Updates        |
| - Workbench | | in isolated | | in Web       | +------------------+
| - Services  | | Node proc   | | Worker       |
| - DI System | +-------------+ +--------------+
+-------------+
       |
       | Spawns workers
       v
+----------------------------------------------+
| WORKER PROCESSES                              |
| - PTY Host (terminal)                        |
| - File Watcher (parcel/watcher)              |
| - TextMate Tokenization                      |
| - Language Detection                         |
| - File Search                                |
| - Editor Web Worker                          |
| - Profile Analysis                           |
| - Notebook Worker                            |
| - Output Link Computer                       |
+----------------------------------------------+
```
