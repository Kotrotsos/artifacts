# VS Code: Electron, Native Layer, and Security Model

**Application:** Visual Studio Code 1.111.0  
**Bundle ID:** com.microsoft.VSCode  
**Build Date:** March 7, 2026  
**Signed by:** Developer ID Application: Microsoft Corporation (UBF8T346G9)  

---

## 1. Electron Version

**Electron 39.6.0**

- Identified from the `devDependencies` in `package.json` (`"electron": "39.6.0"`) and confirmed via binary strings in the Electron Framework (`Electron/39.6.0`).
- Entry point: `./out/main.js` (ESM module, `"type": "module"`)
- The `NSPrincipalClass` in Info.plist is still `AtomApplication`, a legacy from Atom/Electron origins.

## 2. Chromium and V8 Versions

| Component | Version |
|-----------|---------|
| Chromium  | 142.0.7444.265 |
| V8        | Bundled with Chromium 142 (version string not separately exposed) |
| Node.js   | v22.22.0 |

The Chromium version was extracted from the Electron Framework binary user-agent string: `Chrome/142.0.7444.265 Electron/39.6.0`. Node.js version confirmed from binary strings (`v22.22.0`).

## 3. Binary Architecture

```
Mach-O universal binary with 2 architectures:
  - x86_64 (Intel)
  - arm64  (Apple Silicon)
```

All binaries (main executable, helpers, Electron Framework) are universal fat binaries supporting both architectures. The main executable is at `Contents/MacOS/Electron` with a symlink/wrapper at `Contents/MacOS/Code`.

- **Minimum macOS version:** 12.0
- **SDK:** macosx26.1 (Xcode 26.11, build 17B100)
- **Runtime version:** 26.0.0 (hardened runtime enabled)

## 4. Native Node.js Modules

### 4.1 Unpacked Native Addons (.node files)

| Module | Binary | Purpose |
|--------|--------|---------|
| `vsda` | `vsda.node` | VS Dev Authentication, Microsoft internal signing/validation |
| `kerberos` | `kerberos.node` | Kerberos/SPNEGO authentication for enterprise networks |
| `@vscode/policy-watcher` | `vscode-policy-watcher.node` | Group policy monitoring (enterprise management) |
| `@vscode/native-watchdog` | `watchdog.node` | Process watchdog, detects unresponsive main process |
| `@vscode/spdlog` | `spdlog.node` | High-performance structured logging (spdlog C++ library) |
| `@vscode/sqlite3` | `vscode-sqlite3.node` | SQLite3 database for state/storage persistence |
| `@vscode/deviceid` | `windows.node` | Device identification |
| `node-pty` | `pty.node` | Pseudo-terminal for integrated terminal |
| `windows-foreground-love` | `foreground_love.node` | Window focus management (Windows-specific) |
| `native-keymap` | `keymapping.node` | Native keyboard layout detection |
| `@parcel/watcher` | `watcher.node` | High-performance recursive file watching |
| `native-is-elevated` | `iselevated.node` | Checks if process runs with elevated privileges |
| `msal-node-runtime` | `msal-node-runtime.node` | Microsoft Authentication Library (in microsoft-authentication extension) |

### 4.2 Key Non-Native Dependencies

| Package | Purpose |
|---------|---------|
| `@anthropic-ai/sandbox-runtime` | Anthropic AI sandbox runtime (v0.0.23), for Copilot/AI features |
| `@microsoft/1ds-core-js` / `1ds-post-js` | Microsoft telemetry SDK |
| `@xterm/*` | Terminal emulation (xterm.js v6.1.0-beta) |
| `@vscode/ripgrep` | Bundled ripgrep for fast text search |
| `@vscode/tree-sitter-wasm` | Tree-sitter parsing via WASM |
| `@vscode/vscode-languagedetection` | Language detection ML model |
| `@vscode/proxy-agent` | Proxy support with SOCKS/HTTP |
| `playwright-core` | Browser automation (v1.59.0-alpha) |
| `katex` | Math rendering |
| `undici` | HTTP/1.1 and HTTP/2 client |

### 4.3 ASAR Archives

- `node_modules.asar` - Main bundled node_modules (read-only archive)
- `node_modules.asar.unpacked/` - Contains only `vsda` (native module that cannot be packed into asar)
- 101 directories in the unpacked `node_modules/` alongside the asar

## 5. Security Configuration

### 5.1 Content Security Policy

The workbench HTML (`workbench.html`) enforces a strict CSP:

```
default-src 'none';
img-src 'self' data: blob: vscode-remote-resource: vscode-managed-remote-resource: https:;
media-src 'self';
frame-src 'self' vscode-webview:;
script-src 'self' 'unsafe-eval' blob:;
style-src 'self' 'unsafe-inline';
connect-src 'self' https: ws:;
font-src 'self' vscode-remote-resource: vscode-managed-remote-resource: https://*.vscode-unpkg.net;
require-trusted-types-for 'script';
trusted-types amdLoader cellRendererEditorText collapsedCellPreview defaultWorkerFactory
  diffEditorWidget diffReview domLineBreaksComputer dompurify editorGhostText editorViewLayer
  notebookRenderer stickyScrollViewLayer tokenizeToString notebookChatEditController
  richScreenReaderContent chatDebugTokenizer;
```

Key observations:
- `default-src 'none'` - deny by default
- `script-src` allows `'unsafe-eval'` (needed for the JavaScript engine internals, extension host)
- Trusted Types are enforced with 16 named policies
- `connect-src` allows `https:` and `ws:` for remote connections and WebSocket
- `frame-src` restricted to `self` and `vscode-webview:` custom protocol

### 5.2 Sandbox and Context Isolation

The preload script (`vs/base/parts/sandbox/electron-browser/preload.js`) reveals the security model:

- **Context isolation is enabled** - the preload uses `contextBridge.exposeInMainWorld("vscode", ...)` to expose a controlled API
- The exposed `vscode` global provides:
  - `ipcRenderer` - restricted, only allows channels starting with `vscode:`
  - `ipcMessagePort` - for MessageChannel-based communication
  - `webFrame` - limited to `setZoomLevel`
  - `webUtils` - limited to `getPathForFile`
  - `process` - read-only subset (platform, arch, env, versions, type, execPath)
  - `context` - window configuration access

The IPC channel validation is strict:
```javascript
function o(e) {
    if (!e?.startsWith("vscode:"))
        throw new Error(`Unsupported event IPC channel '${e}'`);
    return true;
}
```

### 5.3 Additional Preload Scripts

- `preload.js` - Main window preload (context bridge)
- `preload-aux.js` - Auxiliary window preload
- `preload-browserView.js` - BrowserView preload (for embedded web content)

### 5.4 Network Security

From `Info.plist`:
```xml
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsArbitraryLoads</key>
    <true/>
</dict>
```
App Transport Security is disabled, allowing arbitrary HTTP/HTTPS connections (necessary for extension marketplace, remote development, etc.).

## 6. IPC Protocol

### 6.1 Electron IPC Channels (vscode: namespace)

These are the IPC channels used between the main process and renderer/utility processes:

**Window lifecycle:**
- `vscode:hello` - Initial handshake
- `vscode:disconnect` - Window disconnect
- `vscode:onBeforeUnload` / `vscode:onWillUnload` - Unload lifecycle
- `vscode:enterFullScreen` / `vscode:leaveFullScreen`
- `vscode:notifyZoomLevel`
- `vscode:accessibilitySupportChanged`
- `vscode:registerAuxiliaryWindow`

**File operations:**
- `vscode:openFiles`
- `vscode:addRemoveFolders`

**Process communication:**
- `vscode:createSharedProcessChannelConnection` / `Result`
- `vscode:createSharedProcessRawConnection` / `Result`
- `vscode:createPtyHostMessageChannel` / `Result`
- `vscode:createUtilityProcessWorkerMessageChannelResult`
- `vscode:startExtensionHostMessagePortResult`
- `vscode:reportSharedProcessCrash`

**UI:**
- `vscode:contextmenu` / `vscode:onCloseContextMenu`
- `vscode:openDevTools` / `vscode:toggleDevTools` / `vscode:reloadWindow`
- `vscode:runAction` / `vscode:runKeybinding`

**Dialogs and messages:**
- `vscode:showArgvParseWarning`
- `vscode:showCredentialsError`
- `vscode:showInfoMessage`
- `vscode:showResolveShellEnvError`
- `vscode:showTranslatedBuildWarning`
- `vscode:openProxyAuthenticationDialog`
- `vscode:configureAllowedUNCHost`
- `vscode:disablePromptForProtocolHandling`

**AI/Chat:**
- `vscode:handleChatRequest`
- `vscode:openChatSession`

**Diagnostics:**
- `vscode:getDiagnosticInfo`
- `vscode:triggerIssueData` / `vscode:triggerIssueDataResponse`
- `vscode:message`
- `vscode:uninstall`
- `vscode:prepublish`

### 6.2 Shared Process Service Channels

The shared process (`sharedProcessMain.js`) registers these named channels for service communication:

| Channel | Service |
|---------|---------|
| `extensions` | Extension management |
| `extensionGalleryManifest` | Extension marketplace manifest |
| `extensionTipsService` | Extension recommendations |
| `languagePacks` | Language pack management |
| `telemetryAppender` | Telemetry collection |
| `customEndpointTelemetry` | Custom telemetry endpoints |
| `diagnostics` | Diagnostics service |
| `userDataSync` | Settings Sync |
| `userDataAutoSync` | Automatic settings sync |
| `userDataSyncAccount` | Sync account management |
| `userDataSyncMachines` | Sync machine management |
| `userDataSyncStoreManagement` | Sync store management |
| `IUserDataSyncResourceProviderService` | Sync resource provider |
| `remoteTunnel` | Remote tunnel service |
| `checksum` | File checksum validation |
| `v8InspectProfiling` | V8 profiler |
| `mcpGalleryManifest` | MCP gallery manifest |
| `mcpManagement` | MCP server management |
| `playwright` | Playwright automation |
| `sharedWebContentExtractor` | Web content extraction |

### 6.3 Main Process Service Channels (consumed by workbench)

| Channel | Purpose |
|---------|---------|
| `nativeHost` | Native platform APIs |
| `storage` | Persistent storage |
| `logger` | Logging service |
| `keyboardLayout` | Keyboard layout info |
| `policy` | Enterprise policy |
| `url` | URL handling |
| `watcher` | File system watching |
| `webview` | Webview management |
| `workspaces` | Workspace management |
| `ptyHostWindow` | Terminal host |
| `sign` | Request signing (vsda) |
| `browserElements` | Browser element management |
| `profileStorageListener` | Profile storage events |
| `userDataProfiles` | User data profile management |

## 7. File System Access

### 7.1 File Watching

VS Code uses a dual-watcher strategy:

1. **`@parcel/watcher`** (v2.5.6) - Primary recursive file watcher. Uses native OS APIs:
   - macOS: FSEvents
   - Linux: inotify
   - Windows: ReadDirectoryChangesW
   
2. **`fs.watch`** (Node.js built-in) - Fallback/supplementary watcher

The file watcher runs in a dedicated process (`vs/platform/files/node/watcher/watcherMain.js`) and communicates over the `watcher` IPC channel.

### 7.2 Other File System Components

- **`@vscode/sqlite3`** - SQLite for state database, extension storage, search index
- **`fs-extra`** - Extended file system operations
- **`yauzl`/`yazl`** - ZIP file reading/writing (extension installation)
- **`tar`** - Tarball handling
- **`@vscode/ripgrep`** - Native ripgrep binary for workspace text search

## 8. Code Signing and Entitlements

### 8.1 Signing Details

```
Identifier: com.microsoft.VSCode
Authority chain:
  1. Developer ID Application: Microsoft Corporation (UBF8T346G9)
  2. Developer ID Certification Authority
  3. Apple Root CA
Team Identifier: UBF8T346G9
Notarization: Stapled ticket present
Code Directory: v=20500, flags=0x10000 (hardened runtime)
Sealed Resources: v2, 13 rules, 4415 files
```

### 8.2 Main App Entitlements

| Entitlement | Value | Purpose |
|-------------|-------|---------|
| `com.apple.security.automation.apple-events` | true | AppleScript/osascript automation |
| `com.apple.security.cs.allow-jit` | true | JIT compilation (V8 engine) |
| `com.apple.security.device.audio-input` | true | Microphone access (extensions, webviews) |
| `com.apple.security.device.camera` | true | Camera access (extensions, webviews) |

### 8.3 Helper Process Entitlements

**Code Helper (main):** Same as main app (JIT, audio, camera, AppleEvents)

**Code Helper (Renderer):** `com.apple.security.cs.allow-jit` only

**Code Helper (GPU):** `com.apple.security.cs.allow-jit` only

**Code Helper (Plugin):** Most permissive:
| Entitlement | Value | Purpose |
|-------------|-------|---------|
| `com.apple.security.cs.allow-jit` | true | JIT for V8 |
| `com.apple.security.cs.allow-unsigned-executable-memory` | true | Extensions may need writable+executable memory |
| `com.apple.security.cs.disable-library-validation` | true | Load third-party extension native modules without code signing |

The Plugin helper is intentionally the least restricted because it runs untrusted extension code that may load arbitrary native modules.

### 8.4 Info.plist Privacy Declarations

- `NSAudioCaptureUsageDescription` - Audio capture
- `NSCameraUsageDescription` - Camera access
- `NSMicrophoneUsageDescription` - Microphone access
- `NSBluetoothAlwaysUsageDescription` - Bluetooth access
- `NSBluetoothPeripheralUsageDescription` - Bluetooth peripheral access

## 9. Frameworks

### 9.1 Electron Framework

**Location:** `Contents/Frameworks/Electron Framework.framework/Versions/A/`

Contains the core Chromium + Node.js runtime. Bundled libraries:

| Library | Size | Purpose |
|---------|------|---------|
| `libEGL.dylib` | 250 KB | OpenGL ES / EGL abstraction layer |
| `libGLESv2.dylib` | 14.3 MB | OpenGL ES 2.0 implementation (ANGLE) |
| `libvk_swiftshader.dylib` | 20.5 MB | Vulkan software rasterizer (SwiftShader) |
| `libffmpeg.dylib` | 4.0 MB | Media codec library |
| `vk_swiftshader_icd.json` | 110 B | SwiftShader ICD manifest |

### 9.2 Auto-Update Frameworks

| Framework | Purpose |
|-----------|---------|
| `Squirrel.framework` | Auto-update framework (Squirrel for macOS) |
| `Mantle.framework` | Objective-C model framework (Squirrel dependency) |
| `ReactiveObjC.framework` | Reactive extensions for Objective-C (Squirrel dependency) |

These three frameworks handle VS Code's self-update mechanism on macOS.

### 9.3 Extension-Bundled Native Libraries

| Library | Location | Purpose |
|---------|----------|---------|
| `libmsalruntime.dylib` | `extensions/microsoft-authentication/dist/` | Microsoft Authentication Library native runtime |

## 10. Helper Processes

All helper processes are universal binaries (x86_64 + arm64) located in `Contents/Frameworks/`:

| Helper | Role | Security |
|--------|------|----------|
| **Code Helper.app** | General-purpose helper (network, file I/O) | Full entitlements (JIT, audio, camera, AppleEvents) |
| **Code Helper (Renderer).app** | Renders VS Code UI (workbench, editors) | JIT only |
| **Code Helper (GPU).app** | GPU compositing, WebGL | JIT only |
| **Code Helper (Plugin).app** | Extension host process | Most permissive (JIT, unsigned memory, no library validation) |

### 10.1 Utility Processes

In addition to helper apps, VS Code spawns several utility processes:

- **Shared Process** (`sharedProcessMain.js`) - Runs in an Electron utility process, hosts shared services (extension management, settings sync, telemetry)
- **PTY Host** (`ptyHostMain.js`) - Dedicated process for terminal pseudo-terminal management
- **File Watcher** (`watcherMain.js`) - Dedicated file system watcher process
- **Extension Host Worker** (`extensionHostWorkerMain.js`) - Web worker-based extension host
- **Extension Host Process** (`extensionHostProcess.js`) - Node.js-based extension host
- **Profile Analysis Worker** (`profileAnalysisWorkerMain.js`) - V8 CPU profile analysis

## 11. Process Architecture Summary

```
Code (main process)
  |
  +-- Code Helper (Renderer) ---- Workbench UI
  |
  +-- Code Helper (GPU) --------- GPU compositing
  |
  +-- Code Helper (Plugin) ------ Extension Host (Node.js extensions)
  |
  +-- Code Helper ---------------- General helper
  |
  +-- Shared Process ------------- Extension mgmt, sync, telemetry
  |
  +-- PTY Host ------------------- Terminal sessions
  |
  +-- File Watcher --------------- @parcel/watcher + fs.watch
  |
  +-- Extension Host Worker ------ Web Worker extensions
```

## 12. Notable Security Observations

1. **Hardened Runtime** is enabled (flags=0x10000), which is required for notarization.
2. **Context Isolation** is enforced, the renderer cannot directly access Node.js APIs.
3. **IPC channel filtering** prevents renderer from using arbitrary Electron IPC, only `vscode:` prefixed channels are allowed.
4. **Trusted Types** are enforced in the workbench CSP with 16 named policies.
5. **The Plugin helper** has the weakest security posture by design, it must load unsigned third-party native extension modules.
6. **No App Sandbox** (com.apple.security.app-sandbox), VS Code runs outside the macOS App Sandbox. This is expected for a developer tool that needs unrestricted file system and process access.
7. **ATS disabled**, arbitrary network loads are permitted.
8. **`vsda`** (VS Dev Auth) is the only module in `node_modules.asar.unpacked/`, suggesting it has special loading requirements or needs to avoid asar path resolution issues.
