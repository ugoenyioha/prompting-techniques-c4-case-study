### A) Summary verdict

- Verdict: PASS_WITH_WARNINGS
- Confidence (0-100): 84
- One-paragraph rationale: The final C4 artifacts are largely evidence-backed at container level for local runtime boundaries (CLI, API server, session engine, provider/MCP/plugin runtimes, config/auth policy, SQLite, JSON store) and for the cloud share Durable Object + R2 path. Most major runtime relationships are directly anchored in code. Two correctness risks remain: (1) a contract mismatch between local share client paths (`/api/share/*`) and `packages/function` worker routes (`/share_*`), and (2) an external-actor edge (`Automation Client -> API`) that is structurally plausible but not directly anchored by a concrete first-party caller implementation. These are warnings, not blockers for container-level fidelity.

### B) Claim audit table

| Claim ID | Claim | Status | Evidence | Notes |
| --- | --- | --- | --- | --- |
| C01 | `CLI Runtime` is a real runtime container and command entrypoint | VERIFIED | `packages/opencode/src/index.ts:47`, `packages/opencode/src/index.ts:122`, `packages/opencode/src/index.ts:158` | yargs bootstrap and command wiring are concrete runtime anchors. |
| C02 | `CLI Runtime -> HTTP API Server` via in-process fetch or attached HTTP server | VERIFIED | `packages/opencode/src/cli/cmd/run.ts:604`, `packages/opencode/src/cli/cmd/run.ts:612`, `packages/opencode/src/cli/cmd/run.ts:614` | Both attach mode and in-process `Server.App().fetch` are explicit. |
| C03 | `Web/App UI` uses SDK client + SSE to API | VERIFIED | `packages/app/src/context/global-sdk.tsx:41`, `packages/app/src/context/global-sdk.tsx:129`, `packages/app/src/context/global-sdk.tsx:203` | Separate event and request SDK clients are created against server URL. |
| C04 | `Desktop Shell` spawns/manages sidecar server lifecycle | VERIFIED | `packages/desktop/src-tauri/src/cli.rs:416`, `packages/desktop/src-tauri/src/cli.rs:433`, `packages/desktop/src-tauri/src/server.rs:104` | Rust desktop runtime invokes `serve` sidecar and health-checks it. |
| C05 | `HTTP API Server` is Bun + Hono ingress with route composition and SSE | VERIFIED | `packages/opencode/src/server/server.ts:118`, `packages/opencode/src/server/server.ts:375`, `packages/opencode/src/server/server.ts:638`, `packages/opencode/src/server/server.ts:789` | Hono app composition, route registrations, SSE endpoint, and Bun listener are present. |
| C06 | `API -> Session Engine` dispatches session actions | VERIFIED | `packages/opencode/src/server/server.ts:381`, `packages/opencode/src/server/routes/session.ts:739`, `packages/opencode/src/server/routes/session.ts:808` | Session routes call `SessionPrompt.prompt/command`. |
| C07 | `API -> Plugin Runtime` mounts plugin-provided routes | VERIFIED | `packages/opencode/src/server/server.ts:84`, `packages/opencode/src/server/server.ts:355`, `packages/opencode/src/plugin/index.ts:148` | Dynamic plugin route app is built and fetched before base routes. |
| C08 | `API -> Config/Auth Policy` enforces auth/CORS/tool-endpoint policy | VERIFIED | `packages/opencode/src/server/server.ts:155`, `packages/opencode/src/server/server.ts:212`, `packages/opencode/src/server/server.ts:728`, `packages/opencode/src/server/server.ts:748` | Basic/API-key auth, CORS checks, and tool endpoint validation are explicit. |
| C09 | `Session Engine` orchestrates prompt/tool lifecycle + permissions/events | VERIFIED | `packages/opencode/src/session/processor.ts:45`, `packages/opencode/src/session/processor.ts:53`, `packages/opencode/src/session/prompt.ts:774` | Session processor drives LLM stream and permission flow. |
| C10 | `Session -> Provider Runtime` requests model execution | VERIFIED | `packages/opencode/src/session/prompt.ts:336`, `packages/opencode/src/session/llm.ts:59`, `packages/opencode/src/session/llm.ts:176` | Session resolves model, then `LLM.stream` executes provider-backed `streamText`. |
| C11 | `Provider Runtime` resolves config/auth/env and calls provider endpoints | VERIFIED | `packages/opencode/src/provider/provider.ts:132`, `packages/opencode/src/provider/provider.ts:133`, `packages/opencode/src/provider/provider.ts:397`, `packages/opencode/src/provider/provider.ts:407` | Provider runtime merges auth/config/env and performs outbound provider fetches. |
| C12 | `Session -> MCP Gateway` executes MCP tools/resources | VERIFIED | `packages/opencode/src/session/prompt.ts:830`, `packages/opencode/src/session/prompt.ts:1002`, `packages/opencode/src/mcp/index.ts:135`, `packages/opencode/src/mcp/index.ts:690` | Tool execution and resource reads are both wired end-to-end. |
| C13 | `MCP Gateway` supports stdio + HTTP/SSE transport negotiation from config | VERIFIED | `packages/opencode/src/mcp/index.ts:165`, `packages/opencode/src/mcp/index.ts:328`, `packages/opencode/src/mcp/index.ts:337`, `packages/opencode/src/mcp/index.ts:411` | Config-driven remote/local MCP connectivity is explicit. |
| C14 | `Session -> Plugin Runtime` triggers tool/chat hooks | VERIFIED | `packages/opencode/src/session/prompt.ts:794`, `packages/opencode/src/session/prompt.ts:815`, `packages/opencode/src/session/llm.ts:84`, `packages/opencode/src/session/llm.ts:118` | Before/after tool hooks and chat mutation hooks are called. |
| C15 | `Plugin Runtime` loads internal/external plugins from npm/file sources | VERIFIED | `packages/opencode/src/plugin/index.ts:24`, `packages/opencode/src/plugin/index.ts:69`, `packages/opencode/src/plugin/index.ts:74`, `packages/opencode/src/plugin/index.ts:93` | Built-in + dynamic plugin install/import paths are concrete. |
| C16 | `Config + Auth Policy -> Project Filesystem` loads project/global config/auth inputs | VERIFIED | `packages/opencode/src/config/config.ts:71`, `packages/opencode/src/config/config.ts:112`, `packages/opencode/src/config/config.ts:130`, `packages/opencode/src/auth/index.ts:37`, `packages/opencode/src/auth/index.ts:45` | Config precedence and auth file loading are implemented. |
| C17 | `Session -> SQLite DB` for durable session/message state | VERIFIED | `packages/opencode/src/session/index.ts:13`, `packages/opencode/src/session/index.ts:334`, `packages/opencode/src/storage/db.ts:68`, `packages/opencode/src/storage/db.ts:71` | Session module uses DB queries; DB opens `opencode.db`. |
| C18 | `Session -> JSON State Store` for auxiliary JSON artifacts | VERIFIED | `packages/opencode/src/session/summary.ts:93`, `packages/opencode/src/session/revert.ts:64`, `packages/opencode/src/session/index.ts:504`, `packages/opencode/src/storage/storage.ts:145` | Session diff artifacts are persisted/read in filesystem JSON store. |
| C19 | `Session -> Project Filesystem + Git` runs shell/file operations | VERIFIED | `packages/opencode/src/session/prompt.ts:1530`, `packages/opencode/src/session/prompt.ts:1622`, `packages/opencode/src/session/prompt.ts:983`, `packages/opencode/src/session/prompt.ts:1084` | Session uses worktree paths and executes shell/read operations against filesystem context. |
| C20 | Cloud share runtime exists as `Share API Worker`, `Share Sync Durable Object`, `Share Bucket` | VERIFIED | `packages/function/src/api.ts:29`, `packages/function/src/api.ts:129`, `packages/function/src/api.ts:68` | Worker class, Hono API, and R2 writes are explicit. |
| C21 | `Share API Worker -> Sync DO -> R2` real-time sync/storage chain | VERIFIED | `packages/function/src/api.ts:136`, `packages/function/src/api.ts:174`, `packages/function/src/api.ts:68`, `packages/function/src/api.ts:74` | API routes obtain DO stub; DO persists payloads to R2 and fanouts websockets. |
| C22 | `Share API Worker -> External SaaS APIs` | VERIFIED | `packages/function/src/api.ts:16`, `packages/function/src/api.ts:250`, `packages/function/src/api.ts:284` | Feishu auth API, Discord API, and GitHub JWKS/OIDC token verification are outbound integrations. |
| C23 | `Automation Client -> HTTP API Server` relationship is concretely evidenced in repo | PARTIAL | `packages/opencode/src/server/server.ts:375`, `packages/opencode/src/server/server.ts:638` | API and SSE ingress exist, but no first-party automation client implementation was found; actor remains inferred/external. |
| C24 | `Session -> Share API Worker (packages/function)` endpoint contract is directly evidenced | PARTIAL | `packages/opencode/src/share/share-next.ts:72`, `packages/opencode/src/share/share-next.ts:148`, `packages/function/src/api.ts:131`, `packages/function/src/api.ts:163` | Local client uses `/api/share*` paths, while worker exposes `/share_*`; mapping may exist outside visible code, so direct contract is unproven. |

### C) Hallucination report

- medium - Potential endpoint-contract over-claim in cloud share path
  - why unsupported: `Session` share client uses `/api/share`, `/api/share/:id/sync`, `/api/share/:id`, but `packages/function/src/api.ts` defines `/share_create`, `/share_sync`, `/share_delete`.
  - exact fix: relabel the edge to avoid asserting direct route parity (e.g., "Syncs via share gateway API"), or add an evidence-backed adapter/gateway container between local runtime and `packages/function` worker.
- low - External automation actor is not directly represented by first-party implementation
  - why unsupported: API ingress supports programmatic callers, but no dedicated in-repo "Automation Client" runtime was anchored.
  - exact fix: keep actor but mark as external/inferred in notes, or cite an actual automation client package if one exists.

### D) Omission report

- medium - Missing explicit gateway boundary that explains `/api/share/*` to worker route translation
  - evidence for why it should exist: local runtime targets `/api/share*` (`packages/opencode/src/share/share-next.ts:72`, `packages/opencode/src/share/share-next.ts:148`), while worker route surface is `/share_*` (`packages/function/src/api.ts:131`, `packages/function/src/api.ts:163`).
  - exact fix: add a small external container (e.g., "Share Gateway/Edge Router") and split edges: `Session -> Share Gateway` and `Share Gateway -> Share API Worker`.
- low - API fallback proxy to hosted web app not modeled
  - evidence for why it should exist: server catch-all proxies unknown routes to `https://app.opencode.ai` (`packages/opencode/src/server/server.ts:695`, `packages/opencode/src/server/server.ts:698`).
  - exact fix: optionally add `HTTP API Server -> app.opencode.ai` external edge, or explicitly state this is intentionally out of scope in notes.

### E) Minimal correction plan

1. Update `docs/architecture/c4-analysis-notes-final.md` to explicitly mark cloud share path as "PARTIAL: endpoint mapping inferred" unless a concrete `/api/share*` adapter anchor is provided.
2. In `docs/architecture/c4-opencode-final.puml`, relabel `Rel(session, share_api, ...)` to remove direct route-contract implication (e.g., "Syncs via share gateway API when enabled").
3. Add either (a) a gateway external system/container in `c4-opencode-final.puml`, or (b) a notes caveat stating route translation occurs outside `packages/function` and is not modeled.
4. In notes, annotate `Automation Client` as an external inferred actor (supported by API surface, not by in-repo client implementation).
5. (Optional, low-priority) Add API fallback proxy edge or explicit non-goal note for `app.opencode.ai` catch-all proxy behavior.
