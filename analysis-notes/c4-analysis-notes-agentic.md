# C4 analysis notes (agentic pass)

## Scope and non-goals

- Scope: This pass models container-level runtime architecture for OpenCode across primary execution paths in `packages/opencode`, first-party clients (`packages/app`, `packages/desktop`), and share cloud runtime (`packages/function`).
- Non-goals: This pass does not decompose components/classes, and it does not model non-runtime repository areas (for example docs site internals or infra provisioning internals).

## Execution boundaries found

- CLI runtime process and command dispatcher (`packages/opencode/src/index.ts:47`, `packages/opencode/src/index.ts:133`, `packages/opencode/src/cli/cmd/run.ts:609`).
- HTTP API runtime with Bun listener and Hono routes (`packages/opencode/src/server/server.ts:118`, `packages/opencode/src/server/server.ts:375`, `packages/opencode/src/server/server.ts:789`).
- Session orchestration runtime for prompt/tool loop (`packages/opencode/src/session/prompt.ts:659`, `packages/opencode/src/session/prompt.ts:783`).
- Integration runtimes for providers, MCP, and plugins (`packages/opencode/src/provider/provider.ts:84`, `packages/opencode/src/mcp/index.ts:291`, `packages/opencode/src/plugin/index.ts:18`).
- Policy runtime for config/auth precedence and auth material (`packages/opencode/src/config/config.ts:68`, `packages/opencode/src/config/config.ts:71`, `packages/opencode/src/auth/index.ts:37`).
- Desktop sidecar runtime boundary (`packages/desktop/src-tauri/src/server.rs:104`, `packages/desktop/src-tauri/src/cli.rs:416`).
- App UI runtime boundary using SDK client/event stream (`packages/app/src/context/global-sdk.tsx:41`, `packages/app/src/context/global-sdk.tsx:129`, `packages/app/src/context/global-sdk.tsx:203`).
- Cloud share worker + Durable Object runtime (`packages/function/src/api.ts:29`, `packages/function/src/api.ts:129`, `packages/function/src/api.ts:163`).

## Entry points reviewed

- CLI bootstrap and command registration (`packages/opencode/src/index.ts:47`, `packages/opencode/src/index.ts:158`).
- `serve` command starts headless server (`packages/opencode/src/cli/cmd/serve.ts:6`, `packages/opencode/src/cli/cmd/serve.ts:15`).
- `run` command local in-process call path and attach path (`packages/opencode/src/cli/cmd/run.ts:604`, `packages/opencode/src/cli/cmd/run.ts:612`).
- Server app composition and startup (`packages/opencode/src/server/server.ts:375`, `packages/opencode/src/server/server.ts:789`).
- Desktop sidecar spawn entrypoint (`packages/desktop/src-tauri/src/cli.rs:416`, `packages/desktop/src-tauri/src/cli.rs:433`).
- Cloud share API routes (`packages/function/src/api.ts:131`, `packages/function/src/api.ts:163`).

## Inbound interfaces and auth expectations

- CLI terminal command interface through yargs (`packages/opencode/src/index.ts:47`, `packages/opencode/src/index.ts:122`).
- HTTP/JSON route surface for config/session/provider/tool/mcp/project (`packages/opencode/src/server/server.ts:377`, `packages/opencode/src/server/server.ts:387`).
- SSE event stream (`packages/opencode/src/server/server.ts:638`, `packages/opencode/src/server/server.ts:658`).
- API auth policy: Basic auth and plugin/API-key gates (`packages/opencode/src/server/server.ts:156`, `packages/opencode/src/server/server.ts:182`, `packages/opencode/src/server/server.ts:748`).
- Desktop bridge commands via Tauri invoke bindings (`packages/desktop/src/bindings.ts:7`, `packages/desktop/src/bindings.ts:10`).
- Cloud worker ingress via share routes (`packages/function/src/api.ts:131`, `packages/function/src/api.ts:177`).

## Outbound dependencies mapped

- Provider runtime to external LLM APIs via bundled provider SDKs (`packages/opencode/src/provider/provider.ts:84`, `packages/opencode/src/provider/provider.ts:91`).
- MCP runtime to remote and local MCP transports (`packages/opencode/src/mcp/index.ts:328`, `packages/opencode/src/mcp/index.ts:338`, `packages/opencode/src/mcp/index.ts:411`).
- Plugin runtime to npm/file plugin sources (`packages/opencode/src/plugin/index.ts:69`, `packages/opencode/src/plugin/index.ts:74`, `packages/opencode/src/plugin/index.ts:93`).
- Session runtime to project filesystem/shell tools (`packages/opencode/src/session/prompt.ts:783`, `packages/opencode/src/session/prompt.ts:805`).
- Share sync from local runtime to cloud API (`packages/opencode/src/share/share-next.ts:72`, `packages/opencode/src/share/share-next.ts:148`).
- Cloud share worker to external SaaS APIs and R2 (`packages/function/src/api.ts:250`, `packages/function/src/api.ts:284`, `packages/function/src/api.ts:68`).

## Core flows traced

- Flow A (CLI/app/desktop -> API -> session): CLI local fetch path (`packages/opencode/src/cli/cmd/run.ts:610`), app SDK API/event calls (`packages/app/src/context/global-sdk.tsx:41`, `packages/app/src/context/global-sdk.tsx:129`), API dispatch (`packages/opencode/src/server/server.ts:381`), session prompt execution (`packages/opencode/src/session/prompt.ts:659`).
- Flow B (session -> tools -> filesystem): tool resolution/execution and permission checks (`packages/opencode/src/session/prompt.ts:783`, `packages/opencode/src/session/prompt.ts:774`, `packages/opencode/src/session/prompt.ts:805`) to project files/shell.
- Flow C (session -> provider/mcp/plugin): provider model retrieval (`packages/opencode/src/session/prompt.ts:732`), MCP tool bridge (`packages/opencode/src/session/prompt.ts:830`), plugin hooks before/after execution (`packages/opencode/src/session/prompt.ts:794`, `packages/opencode/src/session/prompt.ts:816`).
- Flow D (session -> persistence + cloud share): SQL + JSON storage (`packages/opencode/src/session/index.ts:13`, `packages/opencode/src/storage/db.ts:68`, `packages/opencode/src/storage/storage.ts:191`) and share sync (`packages/opencode/src/share/share-next.ts:148`).
- Flow E (desktop sidecar startup): desktop spawns `opencode serve`, polls `/global/health`, and exposes URL/password to UI (`packages/desktop/src-tauri/src/cli.rs:433`, `packages/desktop/src-tauri/src/server.rs:120`, `packages/desktop/src/index.tsx:486`).

## Draft A container model + rationale

- Model A containers:
  - Clients: `CLI Runtime`, `Web/App UI`, `Desktop Shell`.
  - Local runtime: `HTTP API Server`, `Session + Integrations` (merged), `Config + Auth Policy`, `SQLite DB`, `JSON State Store`.
  - Cloud share: `Share API Worker`, `Share Sync Durable Object`, `Share Bucket`.
- Rationale: Lower cognitive load and fewer edges; quick read for new contributors.
- Lossiness check for merged container (`Session + Integrations`):
  - Lost signal: provider vs MCP vs plugin failure domains are blurred.
  - Lost signal: ownership/change-coupling between model adapters and plugin route hooks is hidden.
  - Lost signal: debugging path for transport failures (MCP) vs provider auth/config failures is less explicit.
  - Loss level: high for debugging and impact analysis.

## Draft B container model + rationale

- Model B containers:
  - Clients: `CLI Runtime`, `Web/App UI`, `Desktop Shell`.
  - Local runtime: `HTTP API Server`, `Session Engine`, `Provider Runtime`, `MCP Gateway`, `Plugin Runtime`, `Config + Auth Policy`, `SQLite DB`, `JSON State Store`.
  - Cloud share: `Share API Worker`, `Share Sync Durable Object`, `Share Bucket`.
- Rationale: Preserves materially different responsibilities and failure domains while staying container-level.
- Lossiness check:
  - `Provider Runtime` split keeps model/auth/provider drift observable.
  - `MCP Gateway` split keeps transport/auth/connectivity concerns explicit.
  - `Plugin Runtime` split keeps extension-route/tool-hook behavior explicit.
  - Residual merge accepted: all session orchestration remains one container to avoid component-level fragmentation.

## Draft scoring table + selected model

Scoring rubric (1-5 each):
1. Fidelity to evidence in code/docs.
2. Explanatory power for engineers (debugging and change impact analysis).
3. Readability (layout, label clarity, edge complexity).
4. Boundary quality (materially different responsibilities separated).

| Draft | Fidelity | Explanatory power | Readability | Boundary quality | Total |
|---|---:|---:|---:|---:|---:|
| A (merged integrations) | 4 | 3 | 4 | 2 | 13 |
| B (split provider/mcp/plugin) | 5 | 5 | 4 | 5 | 19 |

Selection rationale:
- Selected Draft B. It scores highest on explanatory power and boundary quality, and still keeps diagram readability acceptable at container abstraction.
- This directly satisfies the requirement to favor useful boundaries over conservative minimal grouping.

## Container list and final rationale

- `CLI Runtime`: user command/control and local-vs-attach execution mode.
- `Web/App UI`: interactive frontend consuming API + SSE.
- `Desktop Shell`: sidecar process lifecycle and desktop bridging.
- `HTTP API Server`: single inbound API/SSE boundary.
- `Session Engine`: central orchestration for prompt/tool/lifecycle.
- `Provider Runtime`: provider SDK/auth/model invocation boundary.
- `MCP Gateway`: MCP transport/connectivity/tool-resource boundary.
- `Plugin Runtime`: extension loading and hook/route integration boundary.
- `Config + Auth Policy`: config precedence, auth materials, and route/tool policy gates.
- `SQLite DB` + `JSON State Store`: distinct persistence ownership boundaries.
- `Share API Worker` + `Share Sync Durable Object` + `Share Bucket`: cloud share runtime and persistence boundaries.

## Key evidence anchors (major relationships)

- CLI -> API (`packages/opencode/src/cli/cmd/run.ts:612`, `packages/opencode/src/cli/cmd/run.ts:614`).
- API -> Session (`packages/opencode/src/server/server.ts:381`, `packages/opencode/src/session/prompt.ts:659`).
- API -> Plugin routes (`packages/opencode/src/server/server.ts:355`, `packages/opencode/src/plugin/index.ts:148`).
- Session -> Provider/MCP/Plugin (`packages/opencode/src/session/prompt.ts:732`, `packages/opencode/src/session/prompt.ts:830`, `packages/opencode/src/session/prompt.ts:794`).
- Session -> Persistence (`packages/opencode/src/session/index.ts:13`, `packages/opencode/src/storage/db.ts:68`, `packages/opencode/src/storage/storage.ts:191`).
- Provider -> LLM APIs (`packages/opencode/src/provider/provider.ts:84`, `packages/opencode/src/provider/provider.ts:91`).
- MCP -> MCP servers (`packages/opencode/src/mcp/index.ts:328`, `packages/opencode/src/mcp/index.ts:411`).
- Plugin -> Plugin sources (`packages/opencode/src/plugin/index.ts:69`, `packages/opencode/src/plugin/index.ts:93`).
- Policy -> auth/config loading (`packages/opencode/src/config/config.ts:71`, `packages/opencode/src/auth/index.ts:37`).
- App UI -> API and SSE (`packages/app/src/context/global-sdk.tsx:41`, `packages/app/src/context/global-sdk.tsx:129`, `packages/app/src/context/global-sdk.tsx:203`).
- Desktop -> sidecar CLI -> API (`packages/desktop/src-tauri/src/cli.rs:433`, `packages/desktop/src-tauri/src/server.rs:120`, `packages/desktop/src/index.tsx:471`).
- Session -> Share API (`packages/opencode/src/share/share-next.ts:72`, `packages/opencode/src/share/share-next.ts:148`).
- Share API -> Durable Object -> R2 (`packages/function/src/api.ts:135`, `packages/function/src/api.ts:174`, `packages/function/src/api.ts:68`).

## Assumptions and caveats

- Cloud share endpoints are modeled as first-party external runtime to local `packages/opencode`; deployment topology may vary by environment.
- App UI and desktop are included because they directly shape runtime ingress and sidecar behavior for the same API surface.
- This pass keeps container-level abstraction and intentionally avoids component-level decomposition inside session/provider/mcp/plugin internals.

## Self-critique and one refinement

- Self-critique:
  - Initial Draft B still risked edge density from policy and cloud links.
  - Some relationships could over-specify transport details without improving architecture understanding.
- One refinement applied:
  - Kept the split integration boundaries, but tightened edge text and removed non-essential cross-links so the final diagram remains readable while preserving key failure-domain boundaries.

## Render and validation

- PlantUML source: `docs/architecture/c4-opencode-agentic.puml`.
- SVG target: `docs/architecture/c4-opencode-agentic.svg`.
- Render command used: `plantuml -tsvg docs/architecture/c4-opencode-agentic.puml`.
- Validation: output SVG exists and is non-empty.
