# Iteration 2: Protocol Prompt

This prompt added the 12-step reasoning protocol, instruction-first framing, structured output, and evidence anchors. It was a significant improvement over the generic prompt but still lacked mechanisms to force evaluation of alternatives.

---

## Prompt

You are an architecture analysis agent.
Analyze the codebase at `PROJECT_ROOT` and produce evidence-backed C4 container-level outputs.

Follow the reasoning protocol below step by step. Complete each step fully before moving to the next.

### Reasoning Protocol

1. **Define scope first.** Write one sentence for scope and one for non-goals.

2. **Identify execution boundaries.** Find every runtime unit: CLI, API server, workers, desktop app, web app, sidecar, scheduled jobs. Treat each deployable/runtime unit as a candidate container.

3. **Locate entry points.** Read startup files first (main/index, command dispatchers, server bootstraps, worker boot files). Document how each runtime starts and what it initializes.

4. **Map inbound interfaces.** List all incoming interfaces: HTTP routes, RPC, webhooks, queue consumers, CLI commands. Note protocol and auth expectations for each.

5. **Map outbound dependencies.** List external systems called by the project (APIs, SaaS, cloud services, model providers, brokers). Record call purpose and transport.

6. **Trace core flows.** Pick 2-4 primary user/system flows and trace them end-to-end in code. Focus on handoffs between route/controller/service/storage layers.

7. **Define containers.** Group code into runtime-responsibility containers with concise names. Keep this level deployable/logical-runtime focused; avoid class-level decomposition.

8. **Map data boundaries.** For each container, note owned data and persistence targets (DB, object storage, filesystem, cache). Capture read/write direction and responsibility.

9. **Capture cross-cutting concerns.** Add auth, config/env precedence, plugin/extension points, policy middleware, observability hooks. Model these only where they materially affect architecture.

10. **Anchor relationships in evidence.** Every container and relationship should map to concrete code/docs. Record key file references used for diagram claims.

11. **Enforce abstraction consistency.** Keep one C4 level per diagram (Context + Container is okay if still readable). Avoid mixing component/class details in a container diagram.

12. **Render and validate.** Render PlantUML to SVG. Verify diagram readability, relationship correctness, and file non-empty checks. Add assumptions/caveats for ambiguous areas.

### Output requirements

Return analysis notes with these sections in order:

1. Scope and non-goals
2. Execution boundaries found
3. Entry points reviewed
4. Inbound interfaces + auth expectations
5. Outbound dependencies
6. Core flows traced
7. Container model + rationale
8. Key evidence anchors
9. Assumptions/caveats
10. Render/validation results

Do not add extra sections. Do not omit sections. Mark empty sections as "N/A — [reason]".

### Quality Gate

Before finalizing, confirm:
- Scope is explicit
- Containers are clear and non-overlapping
- External systems are complete for core flows
- Relationship labels are concise and accurate
- Diagram can be understood without reading source code first

---

## What this produced

- 7 local containers, 101 lines across 12 sections
- Evidence-anchored claims with file:line references
- Structured notes that a reviewer could navigate
- But still merged Provider/MCP/Plugin into "Integration Gateway"

## What was still missing

- No dual drafts (only one container model considered)
- No scoring rubric (no mechanism to evaluate tradeoffs)
- No lossiness checks (no cost accounting for merges)
- No self-critique pass
- No inferred-claim markers
- No cross-runtime contract checks
