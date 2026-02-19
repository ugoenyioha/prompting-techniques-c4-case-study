# C4 Reasoning Protocol

Use this protocol when analyzing any codebase to produce a C4 diagram.

1. Define scope first

- Decide whether the target is full-system architecture or a specific runtime boundary.
- Write one sentence for scope and one sentence for non-goals.

2. Identify execution boundaries

- Find every runtime unit: CLI, API server, workers, desktop app, web app, sidecar, scheduled jobs.
- Treat each deployable/runtime unit as a candidate container.

3. Locate entry points

- Read startup files first (`main/index`, command dispatchers, server bootstraps, worker boot files).
- Document how each runtime starts and what it initializes.

4. Map inbound interfaces

- List all incoming interfaces: HTTP routes, RPC, webhooks, queue consumers, CLI commands.
- Note protocol and auth expectations for each interface.

5. Map outbound dependencies

- List external systems called by the project (APIs, SaaS, cloud services, model providers, brokers).
- Record call purpose and transport (HTTP, SSE, stdio, DB protocol, etc.).

6. Trace core flows

- Pick 2-4 primary user/system flows and trace them end-to-end in code.
- Focus on handoffs between route/controller/service/storage layers.

7. Define containers

- Group code into runtime-responsibility containers with concise names.
- Keep this level deployable/logical-runtime focused; avoid class-level decomposition.

8. Map data boundaries

- For each container, note owned data and persistence targets (DB, object storage, filesystem, cache).
- Capture read/write direction and responsibility.

9. Capture cross-cutting concerns

- Add auth, config/env precedence, plugin/extension points, policy middleware, observability hooks.
- Model these only where they materially affect architecture.

10. Anchor relationships in evidence

- Every container and relationship should map to concrete code/docs.
- Record key file references used for diagram claims.

11. Enforce abstraction consistency

- Keep one C4 level per diagram (Context + Container is okay if still readable).
- Avoid mixing component/class details in a container diagram.

12. Render and validate

- Render PlantUML to SVG.
- Verify diagram readability, relationship correctness, and file non-empty checks.
- Add assumptions/caveats for ambiguous areas.

## Quality Gate

Before finalizing, confirm:

- Scope is explicit.
- Containers are clear and non-overlapping.
- External systems are complete for core flows.
- Relationship labels are concise and accurate.
- Diagram can be understood without reading source code first.
