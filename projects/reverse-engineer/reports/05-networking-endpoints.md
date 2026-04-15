# VS Code Networking, Endpoints, and Data Flows

**Version analyzed:** 1.111.0 (commit `ce099c1ed25d9eb3076c11e4a280f3eb52b4fbeb`, 2026-03-06)  
**Quality channel:** stable  
**Source files examined:** product.json, telemetry-core.json, telemetry-extensions.json, workbench.desktop.main.js, sharedProcessMain.js, cliProcessMain.js, main.js, sessions.desktop.main.js

---

## 1. Update Mechanism

VS Code checks for updates via a centralized update service. The update channel is controlled by the `quality` field in product.json (set to `"stable"`).

| Purpose | Endpoint |
|---------|----------|
| Update check & download | `https://update.code.visualstudio.com` |
| Download mirror (PRSS CDN) | `https://vscode.download.prss.microsoft.com/` |
| Download page | `https://code.visualstudio.com` |
| Release notes | `https://go.microsoft.com/fwlink/?LinkID=533483#vscode` |
| Checksum verification info | `https://go.microsoft.com/fwlink/?LinkId=828886` |
| Emergency alert feed | `https://main.vscode-cdn.net/core/stable.json` |

**Telemetry events related to updates:**
- `update:error`
- `windowsUpdateInit`
- `extensions:updatecheckonproductupdate`

**Update data flow:** VS Code periodically queries `update.code.visualstudio.com` with its current version, platform, commit hash, and quality channel. The server responds with update availability and download URLs. Actual binaries may be fetched from the PRSS CDN mirror.

---

## 2. Telemetry & Data Collection

### 2.1 Telemetry Submission Endpoints

| Purpose | Endpoint |
|---------|----------|
| Primary telemetry (1DS/OneCollector) | `https://mobile.events.data.microsoft.com/OneCollector/1.0` |
| Telemetry health ping | `https://mobile.events.data.microsoft.com/ping` |
| A/B experimentation (TAS) | `https://default.exp-tas.com/vscode/ab` |

**Telemetry is enabled by default:** `"enableTelemetry": true` in product.json, with `"showTelemetryOptOut": true` giving users the opt-out dialog.

### 2.2 Telemetry Key (ARIA/1DS)

The embedded telemetry ingestion key (1DS/ARIA key):

```
5bbf946d11a54f6783919c455abaddaf-fd62977b-c92d-4714-a45d-649d06980372-7168
```

This key is used by the 1DS (One Data Strategy) SDK, which replaced Application Insights. The `sharedProcessMain.js` contains the `instrumentationKey` and `iKey` references for this pipeline.

### 2.3 Common Telemetry Properties (sent with every event)

Every telemetry event includes these 29 properties:

| Property | Description | Classification |
|----------|-------------|----------------|
| `common.machineid` | Hash of MAC address | EndUserPseudonymizedInformation |
| `common.devdeviceid` | SQM Machine ID | EndUserPseudonymizedInformation |
| `common.sqmid` | SQM ID | EndUserPseudonymizedInformation |
| `sessionid` | Random session identifier | SystemMetaData |
| `common.version.renderer` | VS Code version | SystemMetaData |
| `common.version.shell` | Electron version | SystemMetaData |
| `common.platform` | OS platform | SystemMetaData |
| `common.platformdetail` | Detailed OS info | SystemMetaData |
| `common.platformversion` | OS version | SystemMetaData |
| `common.nodearch` | CPU architecture | SystemMetaData |
| `common.nodeplatform` | Node.js platform | SystemMetaData |
| `common.product` | Product identifier | SystemMetaData |
| `common.releasedate` | Build date | SystemMetaData |
| `common.cli` | Launched from CLI | SystemMetaData |
| `common.firstsessiondate` | First session date | SystemMetaData |
| `common.lastsessiondate` | Last session date | SystemMetaData |
| `common.isnewsession` | Is new session | SystemMetaData |
| `common.istouchdevice` | Touch device | SystemMetaData |
| `common.remoteauthority` | Remote connection type | SystemMetaData |
| `common.useragent` | User agent string | SystemMetaData |
| `common.sequence` | Event sequence number | SystemMetaData |
| `common.timesincesessionstart` | Time since start | SystemMetaData |
| `common.snap` | Snap package | SystemMetaData |
| `common.msftinternal` | Microsoft employee | SystemMetaData |
| `commithash` | Git commit hash | SystemMetaData |
| `version` | Product version | SystemMetaData |
| `timestamp` | Event timestamp | SystemMetaData |
| `pluginhosttelemetry` | Extension telemetry flag | SystemMetaData |
| `abexp.assignmentcontext` | A/B experiment assignment | SystemMetaData |

### 2.4 Core Telemetry Events (293 total)

Key event categories tracked by VS Code core:

**Startup & Performance:**
- `workspaceLoad`, `startupLayout`, `startupTimeVaried`, `startupHeapStatistics`
- `startup.timer.mark`, `startup.resource.perf`
- `performance.inputLatency`, `terminalLatencyStats`

**Editor & Workspace:**
- `editorOpened`, `editorActionInvoked`, `workbenchActionExecuted`
- `workspace.stats`, `workspace.stats.configFiles`, `workspace.stats.file`
- `workspace.remotes`, `workspace.azure`
- `workspce.tags` (sic, typo preserved from source)

**Extensions:**
- `extensionGallery:install`, `extensionGallery:uninstall`, `extensionGallery:update`
- `extensionActivationTimes`, `extensionActivationError`
- `extensionHostCrash`, `extensionHostCrashExtension`
- `exthostunresponsive`, `extensionHostStartup`
- `extensions.verifySignature`, `extensionsignature:verification`

**AI/Copilot:**
- `chatEntitlements`, `chatInstallEntitlement`
- `chatViewPaneOpened`, `chatFollowupClicked`, `chatFollowupsRetrieved`
- `chat.editRequestsFinished`, `chat.modeChange`, `chat.modelChange`
- `chat.promptRun`, `chatEditing/workingSetSize`
- `chatDebugPanelOpened`, `chatDebugViewSwitched`
- `languageModelToolInvoked`, `instructionsCollected`
- `editTelemetry.codeAccepted`, `editTelemetry.codeSuggested`
- `editTelemetry.reportEditArc`, `editTelemetry.reportInlineEditArc`
- `agentSessionOpened`, `agentSkillsFound`
- `copilotChat.runTaskTool.run`, `copilotChat.getTaskOutputTool.get`

**MCP (Model Context Protocol):**
- `mcp.addserver`, `mcp.addserver.completed`
- `mcp/serverBoot`, `mcp/serverBootState`, `mcp/serverInstall`
- `mcp/authSetup`, `mcp.elicitationRequested`

**Search:**
- `searchComplete`, `searchResultsFinished`, `searchResultsFirstRender`
- `textSearchComplete`, `cachedSearchComplete`

**Remote & Tunnels:**
- `remoteConnectionSuccess`, `remoteConnectionFailure`
- `remoteConnectionHealth`, `remoteConnectionLatency`
- `remoteReconnectionPermanentFailure`, `remoteReconnectionReload`
- `remoteTunnel.connected`, `remoteTunnel.enablement`

**Settings Sync:**
- `settingsSync:sync`, `sync/error`, `sync/showConflicts`
- `sync.download.latest`

**Terminal:**
- `terminal/createInstance`, `terminal/shellIntegrationActivationSucceeded`
- `terminal.suggest.acceptedCompletion`, `terminal.suggest.completionLatency`

**System:**
- `UnhandledError`, `windowerror`, `utilityprocesscrash`
- `gpu.crash.fallback`, `power.resume`, `power.suspend`

### 2.5 Extension Telemetry Events (72 events)

Extensions (git, js-debug, json-language-features) report separately:
- `git/clone`, `git/git.command`, `git/git.missing`, `git/statusSlow`
- `ms-vscode.node/ClientRequest/*` (attach, completions, launch, evaluate, etc.)
- `json-language-features/json.schema`

### 2.6 Microsoft Internal Domain Detection

VS Code checks if the user is on a Microsoft corporate network by matching DNS suffixes:

```
redmond.corp.microsoft.com
northamerica.corp.microsoft.com
fareast.corp.microsoft.com
ntdev.corp.microsoft.com
wingroup.corp.microsoft.com
southpacific.corp.microsoft.com
wingroup.windeploy.ntdev.microsoft.com
ddnet.microsoft.com
europe.corp.microsoft.com
```

This controls the `common.msftinternal` telemetry flag.

---

## 3. Extension Marketplace / Gallery

| Purpose | Endpoint |
|---------|----------|
| Gallery API | `https://marketplace.visualstudio.com/_apis/public/gallery` |
| Extension item pages | `https://marketplace.visualstudio.com/items` |
| Publisher pages | `https://marketplace.visualstudio.com/publishers` |
| Extension resources (CDN) | `https://{publisher}.vscode-unpkg.net/{publisher}/{name}/{version}/{path}` |
| Extension latest version | `https://www.vscode-unpkg.net/_gallery/{publisher}/{name}/latest` |
| Marketplace control file | `https://main.vscode-cdn.net/extensions/marketplace.json` |
| NLS / language packs | `https://www.vscode-unpkg.net/_lp/` |
| NLS core base | `https://www.vscode-unpkg.net/nls/` |
| Gallery CDN assets | `https://*.gallerycdn.vsassets.io` |
| Report marketplace issues | `https://github.com/microsoft/vsmarketplace/issues/new` |

### MCP Server Gallery

| Purpose | Endpoint |
|---------|----------|
| MCP gallery API | `https://api.mcp.github.com` |
| MCP server catalog | `https://main.vscode-cdn.net/mcp/servers.json` |
| MCP item web page | `https://github.com/mcp/{name}` |
| MCP support | `https://support.github.com` |

### Chat Participant / Agent Registry

| Purpose | Endpoint |
|---------|----------|
| Chat participants | `https://main.vscode-cdn.net/extensions/chat.json` |

### Copilot Marketplace SKUs (access entitlements checked)

```
copilot_enterprise_seat
copilot_enterprise_seat_quota
copilot_enterprise_seat_multi_quota
copilot_enterprise_seat_assignment
copilot_enterprise_seat_assignment_quota
copilot_enterprise_seat_assignment_multi_quota
copilot_enterprise_trial_seat
copilot_enterprise_trial_seat_quota
copilot_for_business_seat
copilot_for_business_seat_quota
copilot_for_business_seat_multi_quota
copilot_for_business_seat_assignment
copilot_for_business_seat_assignment_quota
copilot_for_business_seat_assignment_multi_quota
copilot_for_business_trial_seat
copilot_for_business_trial_seat_quota
```

---

## 4. Authentication

### 4.1 Microsoft Authentication (OAuth 2.0)

| Purpose | Endpoint |
|---------|----------|
| OAuth authorize | `https://login.microsoftonline.com/common/oauth2/authorize` |
| OAuth token exchange | `https://login.microsoftonline.com/common/oauth2/token` |
| OAuth redirect | `https://vscode-redirect.azurewebsites.net/` |
| OAuth client metadata | `https://vscode.dev/oauth/client-metadata.json` |

**Embedded Microsoft OAuth Client ID:**
```
aebc6443-996d-45c2-90f0-388ff96faa56
```

**Scopes requested:**
- Settings Sync (Microsoft): `openid`, `profile`, `email`, `offline_access`
- Settings Sync (GitHub): `user:email`
- Tunnels (Microsoft): `46da2f7e-b5ef-422a-88d4-2a7f9de6a0b2/.default`, `profile`, `openid`
- Tunnels (GitHub): `user:email`, `read:org`

### 4.2 GitHub Authentication (OAuth)

| Purpose | Endpoint |
|---------|----------|
| GitHub OAuth flow | `https://github.com/login/oauth` |
| GitHub API | `https://api.github.com/repos/` |
| GitHub search API | `https://api.github.com/search/issues` |
| GitHub avatar images | `https://avatars.githubusercontent.com` |

**GitHub OAuth scopes requested (for Copilot):**
```
read:user, user:email, repo, workflow  (primary)
user:email                             (fallback 1)
read:user                              (fallback 2)
```

### 4.3 OpenAI Authentication

| Purpose | Endpoint |
|---------|----------|
| OpenAI auth | `https://auth.openai.com` |

### 4.4 Trusted Extensions with Auth Access

Extensions pre-authorized for GitHub auth:
- `vscode.github`, `github.remotehub`, `ms-vscode.remote-server`
- `github.vscode-pull-request-github`, `github.codespaces`
- `github.copilot`, `github.copilot-chat`
- `ms-vsliveshare.vsliveshare`, `ms-azuretools.vscode-azure-github-copilot`

Extensions pre-authorized for Microsoft auth:
- `github.copilot-chat`, `ms-vscode.azure-repos`, `ms-vscode.remote-server`
- `ms-vsliveshare.vsliveshare`, `ms-azuretools.vscode-azure-github-copilot`
- `ms-azuretools.vscode-azureresourcegroups`, `ms-azuretools.vscode-dev-azurecloudshell`
- `ms-edu.vscode-learning`, `ms-toolsai.vscode-ai`, `ms-toolsai.vscode-ai-remote`

### 4.5 Trusted Extension Publishers

Publishers trusted for auto-install/special privileges:
```
microsoft, github, openai
```

---

## 5. Remote Development & Tunnels

| Purpose | Endpoint |
|---------|----------|
| Web editor (tunnels) | `https://vscode.dev` |
| Web redirect | `https://vscode.dev/redirect` |
| GitHub Codespaces | `https://github.com/features/codespaces` |
| Azure Portal | `https://portal.azure.com` |

**Tunnel application names:**
- Stable: `code-server`
- Exploration: `code-server-exploration`
- Insiders: `code-server-insiders`
- Tunnel CLI: `code-tunnel`

**Tunnel authentication scope (Microsoft):**
```
46da2f7e-b5ef-422a-88d4-2a7f9de6a0b2/.default
```
This is the Azure AD application ID for the VS Code tunnel service.

---

## 6. AI / Copilot Integration

### 6.1 Copilot API Endpoints

| Purpose | Endpoint |
|---------|----------|
| User entitlement check | `https://api.github.com/copilot_internal/user` |
| Limited user signup | `https://api.github.com/copilot_internal/subscribe_limited_user` |
| Token exchange | `https://api.github.com/copilot_internal/v2/token` |
| MCP registry data | `https://api.github.com/copilot/mcp_registry` |
| Copilot MCP service | `https://api.githubcopilot.com/mcp` |

### 6.2 Copilot Configuration

**Default chat agent configuration:**
- Provider extension: `vscode.github-authentication`
- Main extension: `GitHub.copilot`
- Chat extension: `GitHub.copilot-chat`

**Auth providers for Copilot:**
- GitHub (default)
- GitHub Enterprise (`github-enterprise`)
- Google
- Apple

**Extensions with forced version pinning:**
- `github.copilot-chat` (excludeVersionRange: `<=0.16.1`)
- `github.copilot` (has pre-release version)

### 6.3 OpenAI / Codex Integration

The `openai.chatgpt` extension is listed in `chatSessionRecommendations` with proposed API access to `languageModelProxy` and `chatSessionsProvider`. This is the OpenAI Codex agent.

---

## 7. Crash Reporting

| Purpose | Detail |
|---------|--------|
| Crash reporter product | `VSCode` |
| Crash reporter company | `Microsoft` |

### App Center IDs (crash reporting destinations)

| Platform | App Center ID |
|----------|--------------|
| win32-x64 | `a4e3233c-699c-46ec-b4f4-9c2a77254662` |
| win32-arm64 | `3712d786-7cc8-4f11-8b08-cc12eab6d4f7` |
| linux-x64 | `fba07a4d-84bd-4fc8-a125-9640fc8ce171` |
| darwin (Intel) | `860d6632-f65b-490b-85a8-3e72944f7774` |
| darwin-arm64 | `be71415d-3893-4ae5-b453-e537b9668a10` |
| darwin-universal | `de75e3cc-e22f-4f42-a03f-1409c21d8af8` |

Crash reports are submitted via the `appcenter://` protocol scheme using these application IDs.

**Crash-related telemetry events:**
- `extensionHostCrash`, `extensionHostCrashExtension`
- `utilityprocesscrash`, `gpu.crash.fallback`
- `windowerror`, `UnhandledError`

The crash reporter endpoint was historically:
```
https://vscode-probot.westus.cloudapp.azure.com
```

---

## 8. DNS / Network Configuration

### 8.1 Proxy Settings

VS Code respects the `http.proxy` setting for all outbound HTTP/HTTPS connections. Environment variables `HTTP_PROXY`, `HTTPS_PROXY`, and `NO_PROXY` are also honored.

**Proxy-related telemetry events:**
- `proxyAuthenticationRequest`
- `proxyResolveStats`
- `additionalCertificates`

### 8.2 Link Protection / Trusted Domains

VS Code maintains a whitelist of trusted domains for link protection:

```
https://*.visualstudio.com
https://*.microsoft.com
https://aka.ms
https://*.gallerycdn.vsassets.io
https://*.github.com
https://login.microsoftonline.com
https://*.vscode.dev
https://*.github.dev
https://gh.io
https://portal.azure.com
https://raw.githubusercontent.com
https://private-user-images.githubusercontent.com
https://avatars.githubusercontent.com
https://auth.openai.com
```

### 8.3 CDN Infrastructure

| Purpose | Domain |
|---------|--------|
| Main VS Code CDN | `https://main.vscode-cdn.net` |
| Dynamic CDN (UUID-based) | `https://{{uuid}}.vscode-cdn.net/{{quality}}/{{commit}}` |
| Extension unpkg CDN | `https://www.vscode-unpkg.net` |
| Publisher-specific CDN | `https://{publisher}.vscode-unpkg.net` |
| Webview content CDN | `https://{{uuid}}.vscode-cdn.net/{{quality}}/{{commit}}/out/vs/workbench/contrib/webview/browser/pre/` |
| Source maps CDN | `https://main.vscode-cdn.net/sourcemaps/` |
| Schema store CDN | `https://schemastore.azurewebsites.net/` |

---

## 9. License Validation

VS Code does not perform traditional license key validation. Instead, licensing is handled through:

1. **Copilot entitlement checks** via `https://api.github.com/copilot_internal/user` and `https://api.github.com/copilot_internal/v2/token`
2. **Marketplace access SKUs** (listed in section 3) determine feature access based on subscription type
3. **Extension signature verification** (`extensions.verifySignature`, `extensionsignature:verification` telemetry events)

License/legal URLs:
| Purpose | URL |
|---------|-----|
| VS Code license | `https://go.microsoft.com/fwlink/?LinkID=533485` |
| Server license | `https://aka.ms/vscode-server-license` |
| Privacy statement | `https://go.microsoft.com/fwlink/?LinkId=521839` |
| Microsoft Privacy Statement | `https://privacy.microsoft.com/en-US/privacystatement` |

---

## 10. Settings Sync

| Purpose | Endpoint |
|---------|----------|
| Sync service (stable) | `https://vscode-sync.trafficmanager.net/` |
| Sync service (insiders) | `https://vscode-sync-insiders.trafficmanager.net/` |
| Edit sessions service | `https://vscode-sync.trafficmanager.net/` |

---

## 11. Settings Search (Bing-powered)

| Purpose | Endpoint |
|---------|----------|
| Settings search API | `https://bingsettingssearch.trafficmanager.net/api/Search` |

---

## 12. Surveys & NPS

| Purpose | Endpoint |
|---------|----------|
| NPS survey | `https://aka.ms/vscode-nps` |
| Newsletter signup | `https://www.research.net/r/vsc-newsletter` |
| C++ survey | `https://www.research.net/r/VBVV6C6` |
| Java survey | `https://www.research.net/r/vscodejava` |
| JavaScript survey | `https://www.research.net/r/vscode-js` |
| TypeScript survey | `https://www.research.net/r/vscode-ts` |
| C# survey | `https://www.research.net/r/8KGJ9V8` |

Survey probability configuration:
- C++: 15% of users after 10 edits
- Java: 30% of users after 10 edits
- JavaScript: 5% of users after 10 edits
- TypeScript: 5% of users after 10 edits
- C#: 10% of users after 10 edits

---

## 13. Schema & Metadata Fetching

VS Code fetches JSON schemas from external sources for IntelliSense:

| Purpose | Endpoint |
|---------|----------|
| JSON Schema Store | `https://json.schemastore.org/` |
| SchemaStore legacy | `https://www.schemastore.org/` |
| Microsoft JSON schemas | `https://developer.microsoft.com/json-schemas/` |
| DevContainer schema | `https://raw.githubusercontent.com/devcontainers/spec/main/schemas/devContainer.schema.json` |
| CSS custom data schema | `https://raw.githubusercontent.com/microsoft/vscode-css-languageservice/master/docs/customData.schema.json` |
| HTML custom data schema | `https://raw.githubusercontent.com/microsoft/vscode-html-languageservice/master/docs/customData.schema.json` |
| TypeScript config schema | `https://www.typescriptlang.org/tsconfig` |
| SWC schema | `https://swc.rs/schema.json` |
| TypeDoc schema | `https://typedoc.org/schema.json` |
| MCP server schema | `https://static.modelcontextprotocol.io/schemas/2025-07-09/server.schema.json` |
| NPM registry | `https://registry.npmjs.org` |
| Bower registry | `https://registry.bower.io` |
| Profile templates | `https://main.vscode-cdn.net/core/profile-templates.json` |

---

## 14. Complete Unique Domain Inventory

All unique domains contacted by VS Code (excluding raw.githubusercontent.com API docs):

### Microsoft-owned
| Domain | Purpose |
|--------|---------|
| `update.code.visualstudio.com` | Update server |
| `code.visualstudio.com` | Download & docs |
| `marketplace.visualstudio.com` | Extension marketplace |
| `main.vscode-cdn.net` | CDN for configs, marketplace control, sourcemaps |
| `*.vscode-cdn.net` | Dynamic CDN |
| `www.vscode-unpkg.net` | Extension package CDN |
| `*.vscode-unpkg.net` | Publisher-specific CDN |
| `*.gallerycdn.vsassets.io` | Gallery CDN assets |
| `mobile.events.data.microsoft.com` | Telemetry submission (1DS) |
| `default.exp-tas.com` | A/B experimentation |
| `login.microsoftonline.com` | Microsoft OAuth |
| `vscode-redirect.azurewebsites.net` | OAuth redirect |
| `vscode-sync.trafficmanager.net` | Settings Sync |
| `vscode-sync-insiders.trafficmanager.net` | Settings Sync (insiders) |
| `bingsettingssearch.trafficmanager.net` | Settings search |
| `vscode-probot.westus.cloudapp.azure.com` | Crash reporting |
| `vscode.download.prss.microsoft.com` | Update download CDN |
| `go.microsoft.com` | URL shortener/redirector |
| `aka.ms` | URL shortener/redirector |
| `privacy.microsoft.com` | Privacy policy |
| `portal.azure.com` | Azure portal |
| `developer.microsoft.com` | JSON schemas |
| `vscode.dev` | Web editor / tunnel UI |
| `*.vscode.dev` | Web editor subdomains |
| `*.github.dev` | GitHub Codespaces web editor |
| `schemastore.azurewebsites.net` | Schema store CDN |
| `www.research.net` | User surveys |

### GitHub-owned
| Domain | Purpose |
|--------|---------|
| `api.github.com` | GitHub API, Copilot entitlements |
| `api.githubcopilot.com` | Copilot MCP service |
| `api.mcp.github.com` | MCP gallery |
| `github.com` | OAuth, repos, issue reporting |
| `*.github.com` | Wildcard GitHub |
| `gh.io` | GitHub short URLs |
| `raw.githubusercontent.com` | Proposed API definitions, schemas |
| `avatars.githubusercontent.com` | User avatars |
| `private-user-images.githubusercontent.com` | Private content images |

### OpenAI
| Domain | Purpose |
|--------|---------|
| `auth.openai.com` | OpenAI authentication (Codex agent) |

### Third-party
| Domain | Purpose |
|--------|---------|
| `json.schemastore.org` | JSON schemas |
| `www.schemastore.org` | JSON schemas |
| `registry.npmjs.org` | NPM package metadata |
| `registry.bower.io` | Bower package metadata |
| `www.typescriptlang.org` | TypeScript schemas |
| `static.modelcontextprotocol.io` | MCP schemas |
| `swc.rs` | SWC schema |
| `typedoc.org` | TypeDoc schema |

---

## 15. Embedded UUIDs and Application IDs

| UUID | Purpose |
|------|---------|
| `aebc6443-996d-45c2-90f0-388ff96faa56` | Microsoft OAuth Client ID |
| `46da2f7e-b5ef-422a-88d4-2a7f9de6a0b2` | Azure AD Tunnel Service App ID |
| `a4e3233c-699c-46ec-b4f4-9c2a77254662` | App Center: win32-x64 |
| `3712d786-7cc8-4f11-8b08-cc12eab6d4f7` | App Center: win32-arm64 |
| `fba07a4d-84bd-4fc8-a125-9640fc8ce171` | App Center: linux-x64 |
| `860d6632-f65b-490b-85a8-3e72944f7774` | App Center: darwin |
| `be71415d-3893-4ae5-b453-e537b9668a10` | App Center: darwin-arm64 |
| `de75e3cc-e22f-4f42-a03f-1409c21d8af8` | App Center: darwin-universal |
| `EBAE60D6-C8A2-4419-92FF-24F8AD5077AB` | macOS Profile UUID |
| `1a736e7c-1324-4338-be46-fc2a58ae4d14` | Built-in extension metadata |
| `4064f6ec-cb38-4ad0-af64-ee6467e63c82` | Built-in extension metadata |
| `4f56482b-5a03-49ba-8356-210d3b0c1c3d` | Built-in extension metadata |

---

## 16. Data Flow Summary

### On startup, VS Code contacts:
1. `update.code.visualstudio.com` - check for updates
2. `mobile.events.data.microsoft.com` - submit startup telemetry
3. `default.exp-tas.com` - fetch A/B experiment assignments
4. `main.vscode-cdn.net/core/stable.json` - check emergency alerts
5. `marketplace.visualstudio.com` - check for extension updates

### During normal operation:
6. `marketplace.visualstudio.com` - extension search, install, recommendations
7. `mobile.events.data.microsoft.com` - periodic telemetry batches
8. `bingsettingssearch.trafficmanager.net` - when searching settings
9. `json.schemastore.org` - JSON schema fetching for IntelliSense
10. `raw.githubusercontent.com` - proposed API definitions for extensions

### When Copilot is active:
11. `api.github.com/copilot_internal/*` - entitlement and token checks
12. `api.githubcopilot.com` - MCP service
13. `api.github.com/copilot/mcp_registry` - MCP registry

### When Settings Sync is enabled:
14. `login.microsoftonline.com` or `github.com/login/oauth` - authentication
15. `vscode-sync.trafficmanager.net` - sync data up/down

### When Remote/Tunnels are used:
16. `vscode.dev` - web editor interface
17. Tunnel service (Azure AD app `46da2f7e-b5ef-422a-88d4-2a7f9de6a0b2`)

### On crash:
18. App Center (via `appcenter://` protocol with platform-specific AIDs)
19. `vscode-probot.westus.cloudapp.azure.com` - crash data
