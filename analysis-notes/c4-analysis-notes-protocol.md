# C4 analysis notes (protocol pass)

## 1) Scope and non-goals

- Scope: This pass models the runtime container architecture of `packages/opencode` and only the external systems it directly integrates with during normal CLI/API/session execution.
- Non-goals: This pass does not decompose component/class internals, and it does not fully model unrelated product surfaces (for example `packages/console` or docs site internals).

## 2) Execution boundaries found

- CLI runtime process: command dispatcher and command handlers (`packages/opencode/src/index.ts:47`, `packages/opencode/src/cli/cmd/serve.ts:6`).
- HTTP server runtime: Hono app + Bun server listener (`packages/opencode/src/server/server.ts:375`, `packages/opencode/src/server/server.ts:789`).
- Session orchestration runtime: session lifecycle + prompt/tool loop (`packages/opencode/src/session/index.ts:30`, `packages/opencode/src/session/prompt.ts:659`).
- Integration runtime: provider adapters, MCP clients, plugin loading/hooks (`packages/opencode/src/provider/provider.ts:45`, `packages/opencode/src/mcp/index.ts:291`, `packages/opencode/src/plugin/index.ts:18`).
- Persistence runtime: SQLite + JSON store (`packages/opencode/src/storage/db.ts:68`, `packages/opencode/src/storage/storage.ts:144`).

## 3) Entry points reviewed

- CLI bootstrap and command registration (`packages/opencode/src/index.ts:47`, `packages/opencode/src/index.ts:122`).
- `serve` command starts headless server (`packages/opencode/src/cli/cmd/serve.ts:7`, `packages/opencode/src/cli/cmd/serve.ts:15`).
- API server route composition and startup (`packages/opencode/src/server/server.ts:375`, `packages/opencode/src/server/server.ts:789`).
- Run command local in-process fetch vs remote attach mode (`packages/opencode/src/cli/cmd/run.ts:605`, `packages/opencode/src/cli/cmd/run.ts:612`).

## 4) Inbound interfaces mapped

- CLI command interface (terminal): yargs command surface (`packages/opencode/src/index.ts:122`).
- HTTP/JSON routes for project/config/session/provider/tool/MCP (`packages/opencode/src/server/server.ts:377`, `packages/opencode/src/server/server.ts:387`).
- SSE event stream interface (`packages/opencode/src/server/server.ts:638`, `packages/opencode/src/server/server.ts:658`).
- Plugin-provided HTTP routes merged into server middleware (`packages/opencode/src/server/server.ts:355`, `packages/opencode/src/plugin/index.ts:148`).

## 5) Outbound dependencies mapped

- Model provider APIs via bundled provider SDKs (`packages/opencode/src/provider/provider.ts:84`).
- MCP remote transports (StreamableHTTP/SSE) and local stdio transport (`packages/opencode/src/mcp/index.ts:328`, `packages/opencode/src/mcp/index.ts:408`).
- Plugin package install/import from npm/file URLs (`packages/opencode/src/plugin/index.ts:69`, `packages/opencode/src/plugin/index.ts:93`).
- Share API synchronization calls (`packages/opencode/src/share/share-next.ts:72`, `packages/opencode/src/share/share-next.ts:148`).
- Project filesystem and shell tooling via built-in tools (`packages/opencode/src/tool/registry.ts:128`, `packages/opencode/src/tool/bash.ts:122`, `packages/opencode/src/tool/read.ts:20`).

## 6) Core flows traced

- Flow A (CLI -> API -> Session): CLI run command builds SDK client and issues session prompt/command calls to local or attached server (`packages/opencode/src/cli/cmd/run.ts:605`, `packages/opencode/src/cli/cmd/run.ts:594`), server routes dispatch (`packages/opencode/src/server/server.ts:381`), session prompt loop executes (`packages/opencode/src/session/prompt.ts:659`).
- Flow B (Session -> Tools -> FS/Shell): session resolves tool registry and executes tool calls (`packages/opencode/src/session/prompt.ts:602`, `packages/opencode/src/session/prompt.ts:783`), including filesystem/shell tools (`packages/opencode/src/tool/read.ts:20`, `packages/opencode/src/tool/bash.ts:122`).
- Flow C (Session -> Provider/MCP/Plugin): session integrates provider, MCP, plugin hooks during tool and model execution (`packages/opencode/src/session/prompt.ts:650`, `packages/opencode/src/session/prompt.ts:830`), with provider and MCP runtimes handling outbound calls (`packages/opencode/src/provider/provider.ts:84`, `packages/opencode/src/mcp/index.ts:347`).
- Flow D (Session -> Persistence + Share sync): session state writes to SQLite/JSON stores (`packages/opencode/src/session/index.ts:13`, `packages/opencode/src/storage/storage.ts:191`), and share sync publishes to remote share endpoints (`packages/opencode/src/share/share-next.ts:23`, `packages/opencode/src/share/share-next.ts:148`).

## 7) Containers selected and rationale

- CLI Runtime: command-and-control boundary for user invocation and local/attach execution mode.
- HTTP API Server: single inbound HTTP/SSE boundary exposing opencode operations.
- Session Orchestrator: primary runtime behavior boundary for agent loop, tool execution, and lifecycle transitions.
- Integration Gateway: grouped provider/MCP/plugin integration responsibilities to keep container abstraction deployable/runtime-focused.
- Config + Auth Loader: centralized config/env/auth merge and policy influence boundary.
- SQLite DB: authoritative persisted relational state.
- JSON State Store: non-relational artifact/state persistence.

## 8) Data boundaries

- SQLite DB owns durable structured records (sessions/messages/parts/permissions/share metadata) (`packages/opencode/src/storage/db.ts:71`, `packages/opencode/src/session/session.sql.ts:11`).
- JSON state store owns filesystem JSON artifacts and migrations (`packages/opencode/src/storage/storage.ts:145`, `packages/opencode/src/storage/storage.ts:191`).
- Session Orchestrator is the primary writer/reader for conversation state and part updates (`packages/opencode/src/session/index.ts:253`, `packages/opencode/src/session/prompt.ts:686`).

## 9) Cross-cutting concerns captured

- Config precedence and override layering materially influence provider/MCP/plugin/runtime behavior (`packages/opencode/src/config/config.ts:71`, `packages/opencode/src/config/config.ts:179`).
- Permission checks are enforced during tool execution (`packages/opencode/src/session/prompt.ts:774`, `packages/opencode/src/tool/read.ts:45`).
- Plugin hooks wrap tool execution and event flow (`packages/opencode/src/session/prompt.ts:794`, `packages/opencode/src/plugin/index.ts:180`).

## 10) Evidence anchors for major containers and relationships

- `CLI Runtime`: `packages/opencode/src/index.ts:47`, `packages/opencode/src/cli/cmd/run.ts:609`.
- `HTTP API Server`: `packages/opencode/src/server/server.ts:375`, `packages/opencode/src/server/server.ts:789`.
- `Session Orchestrator`: `packages/opencode/src/session/index.ts:30`, `packages/opencode/src/session/prompt.ts:659`.
- `Integration Gateway`: `packages/opencode/src/provider/provider.ts:45`, `packages/opencode/src/mcp/index.ts:291`, `packages/opencode/src/plugin/index.ts:18`.
- `Config + Auth Loader`: `packages/opencode/src/config/config.ts:68`, `packages/opencode/src/config/config.ts:205`.
- `SQLite DB`: `packages/opencode/src/storage/db.ts:68`.
- `JSON State Store`: `packages/opencode/src/storage/storage.ts:144`.
- `CLI -> API`: `packages/opencode/src/cli/cmd/run.ts:612`, `packages/opencode/src/cli/cmd/run.ts:605`.
- `API -> Session`: `packages/opencode/src/server/server.ts:381`, `packages/opencode/src/session/prompt.ts:659`.
- `API -> Integration`: `packages/opencode/src/server/server.ts:384`, `packages/opencode/src/server/server.ts:387`.
- `Session -> Integration`: `packages/opencode/src/session/prompt.ts:650`, `packages/opencode/src/session/prompt.ts:830`.
- `Session -> SQLite/JSON`: `packages/opencode/src/session/index.ts:13`, `packages/opencode/src/storage/storage.ts:191`.
- `Session -> Project Filesystem`: `packages/opencode/src/tool/registry.ts:128`, `packages/opencode/src/tool/read.ts:31`, `packages/opencode/src/tool/bash.ts:146`.
- `Integration -> LLM APIs`: `packages/opencode/src/provider/provider.ts:84`.
- `Integration -> MCP Servers`: `packages/opencode/src/mcp/index.ts:328`, `packages/opencode/src/mcp/index.ts:411`.
- `Integration -> Plugin Packages`: `packages/opencode/src/plugin/index.ts:69`, `packages/opencode/src/plugin/index.ts:93`.
- `Session -> Share API`: `packages/opencode/src/share/share-next.ts:72`, `packages/opencode/src/share/share-next.ts:148`.

## 11) Abstraction consistency check

- Diagram intentionally stays at container level; no component/class-level boxes were added.
- Context actors/systems are limited to those needed to explain major runtime interactions.

## 12) Render and validation

- PlantUML source: `docs/architecture/c4-opencode-protocol.puml`.
- Render target: `docs/architecture/c4-opencode-protocol.svg`.
- Validation performed: rendered SVG exists and is non-empty.

## Assumptions and caveats

- Share sync is modeled as an external system interaction from the session runtime because calls are emitted from in-repo `ShareNext` handlers, while cloud deployment topology can vary by environment.
- The container model groups provider/MCP/plugin internals under one integration boundary to preserve readable container-level abstraction.
