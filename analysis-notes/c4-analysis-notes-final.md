# C4 analysis notes (final comparison pass)

## 1. Scope and non-goals

- Scope: This pass produces a final container-level C4 model for OpenCode runtime boundaries across local runtime (`packages/opencode`), first-party clients (`packages/app`, `packages/desktop`), and cloud share runtime (`packages/function`).
- Non-goals: This pass does not decompose components/classes and does not model unrelated repository areas in detail (for example docs-site internals or full console product internals).

## 2. Execution boundaries found

- CLI runtime process and command dispatch (`packages/opencode/src/index.ts:47`, `packages/opencode/src/cli/cmd/run.ts:609`).
- HTTP API runtime with Bun listener and Hono route composition (`packages/opencode/src/server/server.ts:375`, `packages/opencode/src/server/server.ts:789`).
- Session orchestration runtime for prompt/tool lifecycle (`packages/opencode/src/session/prompt.ts:659`, `packages/opencode/src/session/prompt.ts:783`).
- Integration runtimes for provider, MCP, and plugin responsibilities (`packages/opencode/src/provider/provider.ts:84`, `packages/opencode/src/mcp/index.ts:291`, `packages/opencode/src/plugin/index.ts:18`).
- Policy/config/auth runtime boundary (`packages/opencode/src/config/config.ts:68`, `packages/opencode/src/auth/index.ts:37`).
- Desktop sidecar runtime boundary (`packages/desktop/src-tauri/src/server.rs:104`, `packages/desktop/src-tauri/src/cli.rs:416`).
- App UI runtime boundary consuming API/SSE through SDK (`packages/app/src/context/global-sdk.tsx:41`, `packages/app/src/context/global-sdk.tsx:129`).
- Cloud worker and Durable Object share runtime (`packages/function/src/api.ts:129`, `packages/function/src/api.ts:163`).

## 3. Entry points reviewed

- CLI bootstrap and command registration (`packages/opencode/src/index.ts:47`, `packages/opencode/src/index.ts:158`).
- `serve` command startup (`packages/opencode/src/cli/cmd/serve.ts:6`, `packages/opencode/src/cli/cmd/serve.ts:15`).
- `run` command local in-process and attach execution path (`packages/opencode/src/cli/cmd/run.ts:604`, `packages/opencode/src/cli/cmd/run.ts:612`).
- Server app startup and route registration (`packages/opencode/src/server/server.ts:375`, `packages/opencode/src/server/server.ts:789`).
- Desktop sidecar spawn command path (`packages/desktop/src-tauri/src/cli.rs:433`).
- Cloud share ingress routes (`packages/function/src/api.ts:131`, `packages/function/src/api.ts:177`).

## 4. Inbound interfaces + auth expectations

- CLI terminal command interface via yargs (`packages/opencode/src/index.ts:122`).
- HTTP/JSON route ingress for config/session/provider/tool/mcp/project (`packages/opencode/src/server/server.ts:377`, `packages/opencode/src/server/server.ts:387`).
- SSE stream interface (`packages/opencode/src/server/server.ts:638`, `packages/opencode/src/server/server.ts:658`).
- API auth policy with Basic auth and plugin/API-key gates (`packages/opencode/src/server/server.ts:156`, `packages/opencode/src/server/server.ts:182`, `packages/opencode/src/server/server.ts:748`).
- Desktop invoke bridge to sidecar runtime (`packages/desktop/src/bindings.ts:7`, `packages/desktop/src/bindings.ts:10`).
- `Automation Client` is modeled as an inferred external actor from exposed HTTP/SSE ingress, not as a first-party runtime in this repository.

## 5. Outbound dependencies

- Provider runtime to external model APIs (`packages/opencode/src/provider/provider.ts:84`, `packages/opencode/src/provider/provider.ts:91`).
- MCP runtime to local and remote MCP transports (`packages/opencode/src/mcp/index.ts:328`, `packages/opencode/src/mcp/index.ts:411`).
- Plugin runtime to npm/file plugin sources (`packages/opencode/src/plugin/index.ts:69`, `packages/opencode/src/plugin/index.ts:93`).
- Session runtime to filesystem/shell tools (`packages/opencode/src/session/prompt.ts:783`, `packages/opencode/src/session/prompt.ts:805`).
- Session share sync to share gateway API (`packages/opencode/src/share/share-next.ts:72`, `packages/opencode/src/share/share-next.ts:148`).
- Cloud share worker to external SaaS APIs and R2 (`packages/function/src/api.ts:250`, `packages/function/src/api.ts:284`, `packages/function/src/api.ts:68`).
- API fallback proxy target for unmatched paths is `app.opencode.ai` (`packages/opencode/src/server/server.ts:695`, `packages/opencode/src/server/server.ts:698`).

## 6. Core flows traced

- Flow A: CLI/app/desktop -> API -> session orchestration (`packages/opencode/src/cli/cmd/run.ts:610`, `packages/app/src/context/global-sdk.tsx:41`, `packages/opencode/src/server/server.ts:381`, `packages/opencode/src/session/prompt.ts:659`).
- Flow B: session -> tools -> filesystem/shell with permissions (`packages/opencode/src/session/prompt.ts:774`, `packages/opencode/src/session/prompt.ts:783`, `packages/opencode/src/session/prompt.ts:805`).
- Flow C: session -> provider/MCP/plugin (`packages/opencode/src/session/prompt.ts:732`, `packages/opencode/src/session/prompt.ts:830`, `packages/opencode/src/session/prompt.ts:794`).
- Flow D: session -> persistence + share sync (`packages/opencode/src/session/index.ts:13`, `packages/opencode/src/storage/db.ts:68`, `packages/opencode/src/storage/storage.ts:191`, `packages/opencode/src/share/share-next.ts:148`).
- Flow E: API catch-all -> hosted web app proxy for unmatched routes (`packages/opencode/src/server/server.ts:695`, `packages/opencode/src/server/server.ts:698`).

## 7. Draft A container model + rationale

- Clients: `CLI Runtime`, `Web/App UI`, `Desktop Shell`.
- Local runtime: `HTTP API Server`, `Session + Integrations` (merged), `Config + Auth Policy`, `SQLite DB`, `JSON State Store`.
- Cloud runtime: `Share API Worker`, `Share Sync Durable Object`, `Share Bucket`.
- Rationale: lower edge density and fast orientation for new contributors.
- Lossiness check on merged `Session + Integrations`:
  - Ownership and failure-domain boundaries between provider/MCP/plugin are obscured.
  - Transport-debugging paths for MCP issues vs provider auth/config issues become less clear.
  - Change-coupling analysis for extensions (plugin hooks/routes) is diluted.
  - Loss level: high, so Draft A is not preferred.

## 8. Draft B container model + rationale

- Clients: `CLI Runtime`, `Web/App UI`, `Desktop Shell`.
- Local runtime: `HTTP API Server`, `Session Engine`, `Provider Runtime`, `MCP Gateway`, `Plugin Runtime`, `Config + Auth Policy`, `SQLite DB`, `JSON State Store`.
- Cloud runtime: `Share API Worker`, `Share Sync Durable Object`, `Share Bucket`.
- Rationale: keeps materially different responsibilities split while remaining container-level.
- Lossiness check:
  - Provider split preserves model/auth drift and provider outage failure-domain clarity.
  - MCP split preserves transport/connectivity/tool-resource failure diagnostics.
  - Plugin split preserves extension ownership and hook-impact boundaries.
  - Residual merge accepted: session internals remain one container to avoid component-level fragmentation.

## 9. Draft scoring table + selected model

| Draft                         | Fidelity to evidence | Explanatory power | Readability | Boundary quality | Total |
| ----------------------------- | -------------------: | ----------------: | ----------: | ---------------: | ----: |
| A (merged integrations)       |                    4 |                 3 |           4 |                2 |    13 |
| B (split provider/mcp/plugin) |                    5 |                 5 |           4 |                5 |    19 |

Selected model: Draft B.

Selected model rationale:

- Draft B wins on fidelity and boundary quality without dropping readability below acceptable container-level density.
- The split model maps directly to how incidents and ownership are handled in practice (provider vs MCP transport vs plugin extension behavior).

## 10. Final container list + rationale

- `CLI Runtime`: command/control and local-vs-attach execution handling.
- `Web/App UI`: frontend runtime consuming API and SSE streams.
- `Desktop Shell`: sidecar lifecycle and desktop bridge.
- `HTTP API Server`: single inbound API/SSE dispatch boundary.
- `Session Engine`: orchestration for prompt/tool loop and lifecycle updates.
- `Provider Runtime`: external model adapter boundary.
- `MCP Gateway`: MCP transport and tool/resource execution boundary.
- `Plugin Runtime`: extension loading and hook/route boundary.
- `Config + Auth Policy`: config precedence, auth material loading, policy gates.
- `SQLite DB` and `JSON State Store`: distinct persistence boundaries.
- `Share API Worker`, `Share Sync Durable Object`, `Share Bucket`: cloud share runtime and storage boundaries.

## 11. Key evidence anchors

- CLI -> API: `packages/opencode/src/cli/cmd/run.ts:612`.
- API -> session: `packages/opencode/src/server/server.ts:381`.
- API -> plugin routes: `packages/opencode/src/server/server.ts:355`, `packages/opencode/src/plugin/index.ts:148`.
- Session -> provider/mcp/plugin: `packages/opencode/src/session/prompt.ts:732`, `packages/opencode/src/session/prompt.ts:830`, `packages/opencode/src/session/prompt.ts:794`.
- Session -> persistence: `packages/opencode/src/session/index.ts:13`, `packages/opencode/src/storage/db.ts:68`, `packages/opencode/src/storage/storage.ts:191`.
- Provider -> model APIs: `packages/opencode/src/provider/provider.ts:84`.
- MCP -> MCP servers: `packages/opencode/src/mcp/index.ts:328`, `packages/opencode/src/mcp/index.ts:411`.
- Plugin -> plugin sources: `packages/opencode/src/plugin/index.ts:69`, `packages/opencode/src/plugin/index.ts:93`.
- Policy inputs -> filesystem config/auth files: `packages/opencode/src/config/config.ts:71`, `packages/opencode/src/auth/index.ts:37`.
- App UI -> API/SSE: `packages/app/src/context/global-sdk.tsx:41`, `packages/app/src/context/global-sdk.tsx:129`, `packages/app/src/context/global-sdk.tsx:203`.
- Desktop -> sidecar/API: `packages/desktop/src-tauri/src/cli.rs:433`, `packages/desktop/src-tauri/src/server.rs:120`.
- Session -> share gateway -> share API -> durable object -> R2: `packages/opencode/src/share/share-next.ts:148`, `packages/function/src/api.ts:135`, `packages/function/src/api.ts:174`, `packages/function/src/api.ts:68`.
- Session -> share gateway path contracts: `packages/opencode/src/share/share-next.ts:72`, `packages/opencode/src/share/share-next.ts:148`.
- Share worker route surface: `packages/function/src/api.ts:131`, `packages/function/src/api.ts:163`.
- API fallback proxy edge: `packages/opencode/src/server/server.ts:695`, `packages/opencode/src/server/server.ts:698`.

## 12. Assumptions/caveats

- Cloud share is modeled as a first-party external runtime from local `packages/opencode`; deployment topology can vary by environment.
- The `Share Gateway API` container is explicit to avoid over-claiming direct route parity between local `/api/share*` and worker `/share_*` surfaces.
- Gateway translation implementation is outside this repository, so this contract boundary is intentionally modeled as partial/inferred rather than verified in-repo.
- `Automation Client` remains an inferred external actor; no first-party automation-client package is asserted in this model.
- This pass intentionally excludes detailed decomposition of `packages/console` and docs internals.
- Relationship labels are intentionally short to keep edge density readable while preserving evidence-backed accuracy.

## 13. Self-critique and refinement notes

- Self-critique: first draft overemphasized minor edges from policy and cloud links, which reduced readability.
- Refinement applied: retained the explicit provider/MCP/plugin split but trimmed low-signal cross-links and simplified relationship verbs.
- Outcome: final model keeps failure-domain clarity while staying readable at container-level abstraction.

## 14. Render/validation results

- PlantUML source: `docs/architecture/c4-opencode-final.puml`.
- Render target: `docs/architecture/c4-opencode-final.svg`.
- Render command: `plantuml -tsvg docs/architecture/c4-opencode-final.puml`.
- Validation checks: render completed without error, SVG exists, and SVG is non-empty.
