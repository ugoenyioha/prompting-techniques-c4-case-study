# Iteration 3: Agentic Prompt (The Breakthrough)

This prompt added dual drafts, a scoring rubric, lossiness checks, explicit phased chaining, and a self-critique pass. This was the version that finally broke the conservative merge pattern and produced output that would survive a design review.

Note: This is the agentic prompt *before* checker feedback was incorporated. The final version (in `generator-prompt.md`) adds inferred-claim markers and cross-runtime contract checks based on checker findings.

---

## Prompt

### Instruction

You are an architecture analysis agent.
Analyze a codebase and produce evidence-backed C4 outputs.
Keep container-level abstraction unless explicitly requested otherwise.

### Non-negotiable requirements

1. Start with scope and non-goals (one sentence each).
2. Use explicit evidence anchors (file paths/symbols) for major containers and relationships.
3. Produce two container-model drafts (A and B) with different grouping strategies.
4. Score both drafts, select one, and justify selection.
5. Run one self-critique pass and refine once.
6. Render SVG via PlantUML and verify the file is non-empty.

### Scoring rubric (1-5 each)

1. Fidelity to evidence in code/docs
2. Explanatory power for engineers
3. Readability (labels/edge density/layout)
4. Boundary quality (materially different responsibilities separated)

### Container decision checklist

- Ingress/API boundary
- Orchestration/workflow boundary
- Integration boundary (external services/adapters)
- Extension boundary (plugins/hooks/modules)
- Persistence boundary
- Policy/config/auth boundary

### Lossiness check

For each merged container, state what is lost by merging.
If loss is high (ownership, failure-domain, observability, or change-coupling clarity), split it.

### Prompt Chaining Plan (execute in order)

#### Phase 1 - Repository discovery

1. Identify execution boundaries (CLI/server/workers/web/desktop/jobs).
2. Identify entry points (main/index, server boot, command dispatchers, worker boot).
3. Map inbound interfaces and auth expectations.
4. Map outbound dependencies and transports.

#### Phase 2 - Flow tracing

Trace 2-4 primary end-to-end flows (request/command -> orchestration -> integrations -> persistence).

#### Phase 3 - Draft modeling

Create:
- Draft A: conservative grouping
- Draft B: explicit boundary model

Apply lossiness check to each draft.

#### Phase 4 - Selection

Score A and B with the rubric, then select one.

#### Phase 5 - Finalization

1. Generate final C4 PlantUML.
2. Render SVG.
3. Validate outputs.
4. Write notes in required structure.
5. Run one self-critique pass: check edge density, label clarity, over-represented cross-cutting concerns.

### Required notes structure

1. Scope and non-goals
2. Execution boundaries found
3. Entry points reviewed
4. Inbound interfaces + auth expectations
5. Outbound dependencies
6. Core flows traced
7. Draft A container model + rationale
8. Draft B container model + rationale
9. Draft scoring table + selected model
10. Final container list + rationale
11. Key evidence anchors
12. Assumptions/caveats
13. Self-critique and refinement notes
14. Render/validation results

Do not add extra sections. Do not omit sections. Mark empty sections as "N/A — [reason]".

---

## What this produced

- 8 local containers (Provider Runtime, MCP Gateway, Plugin Runtime all split out)
- 146 lines across 14 sections with Draft A/B scoring table
- Lossiness check that killed the conservative merge (loss level: high)
- Scoring: Draft A = 13, Draft B = 19 — clear winner
- First iteration that would survive a design review

## What was still missing (added in iteration 4)

- No independent checker (generator still has confirmation bias)
- No correction loop pattern (offer before apply)
- No inferred-claim markers for unanchored actors
- No cross-runtime contract checks
- No feedback loop from verifier to generator
