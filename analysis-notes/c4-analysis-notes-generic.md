# C4 analysis notes (generic pass)

## Scope
- Generated a fresh container-level C4 diagram for this repository using the generic template, including local runtime, client surfaces, and cloud worker services.

## Evidence mapping (key anchors)
- CLI entrypoint and command surface: `packages/opencode/src/index.ts:47`
- `serve` command starts headless server: `packages/opencode/src/cli/cmd/serve.ts:15`
- Core HTTP API server/routes/auth/CORS/SSE/fallback proxy: `packages/opencode/src/server/server.ts:118`
- Session orchestration and tool/event flow via run command: `packages/opencode/src/cli/cmd/run.ts:407`
- Provider gateway and multi-provider adapters: `packages/opencode/src/provider/provider.ts:84`
- MCP local/remote transports and OAuth flow: `packages/opencode/src/mcp/index.ts:304`
- Plugin runtime and route hook loading: `packages/opencode/src/plugin/index.ts:32`
- Config/env/auth precedence and loading: `packages/opencode/src/config/config.ts:68`
- Local SQLite persistence and migrations: `packages/opencode/src/storage/db.ts:68`
- Local JSON state storage/migrations: `packages/opencode/src/storage/storage.ts:144`
- Web app SDK client usage to server URL: `packages/app/src/context/global-sdk.tsx:41`
- Desktop app starts/attaches sidecar and health-checks local server: `packages/desktop/src-tauri/src/lib.rs:728`
- Desktop sidecar spawn command (`serve --hostname --port`): `packages/desktop/src-tauri/src/cli.rs:431`
- Cloud API worker (share, token exchange, webhooks): `packages/function/src/api.ts:129`
- Cloud infra wiring for Worker + Durable Object + R2: `infra/app.ts:13`

## Modeling choices
- Kept **container-level abstraction** (not components) to stay within requested scope.
- Grouped provider, MCP, and plugin modules into a single "Integration Gateway" container to keep labels concise while preserving evidence-backed responsibilities.
- Included cloud worker resources because sharing and token-exchange behavior are implemented in-repo and referenced by local runtime (`packages/opencode/src/share/share-next.ts:72`).

## Assumptions and caveats
- The diagram shows runtime flows represented directly in source and infra files; deployment topology can vary by local vs hosted environments.
- `packages/console/*` and billing/auth console internals are out of primary OpenCode local runtime scope and were not decomposed further in this pass.
- "External SaaS APIs" is intentionally grouped (GitHub/Discord/Feishu) to keep relationship labels concise while remaining evidence-based from the worker handlers.
