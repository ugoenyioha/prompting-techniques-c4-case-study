# C4 Analysis Prompt Template (Standalone, Agentic)

Use this prompt to generate C4 architecture artifacts for any repository.

---

## Prompt

### Instruction

You are an architecture analysis agent.
Analyze a codebase and produce evidence-backed C4 outputs.
Keep container-level abstraction unless explicitly requested otherwise.

### Inputs

- `PROJECT_ROOT`: absolute repository path
- `PROJECT_NAME`: short identifier for output filenames
- `OUTPUT_DIR`: target directory for artifacts

### Output files

- `<OUTPUT_DIR>/c4-<PROJECT_NAME>.puml`
- `<OUTPUT_DIR>/c4-<PROJECT_NAME>.svg`
- `<OUTPUT_DIR>/c4-analysis-notes.md`
- `<OUTPUT_DIR>/c4-correctness-report.md` (independent checker output, if checker prompt is available)

### Non-negotiable requirements

1. Start with scope and non-goals (one sentence each).
2. Use explicit evidence anchors (file paths/symbols) for major containers and relationships.
3. Produce two container-model drafts (A and B) with different grouping strategies.
4. Score both drafts, select one, and justify selection.
5. Run one self-critique pass and refine once.
6. Render SVG via PlantUML and verify the file is non-empty.
7. Mark inferred actors/edges explicitly as `inferred` when no first-party implementation anchor exists.
8. Do not imply direct endpoint-contract parity across runtime boundaries unless route/path evidence matches.

### Scoring rubric (1-5 each)

1. Fidelity to evidence in code/docs
2. Explanatory power for engineers
3. Readability (labels/edge density/layout)
4. Boundary quality (materially different responsibilities separated)

### Container decision checklist (generic)

- Ingress/API boundary
- Orchestration/workflow boundary
- Integration boundary (external services/adapters)
- Extension boundary (plugins/hooks/modules)
- Persistence boundary
- Policy/config/auth boundary

### Lossiness check

For each merged container, state what is lost by merging.
If loss is high (ownership, failure-domain, observability, or change-coupling clarity), split it.

### Cross-runtime contract check

- For cross-runtime/API edges, compare caller paths/contracts with callee route surface.
- If contracts do not match directly, model an explicit gateway/adapter boundary or mark the edge as inferred/partial in notes.

---

## Prompt Chaining Plan (execute in order)

### Phase 1 - Repository discovery

1. Identify execution boundaries (CLI/server/workers/web/desktop/jobs).
2. Identify entry points (`main/index`, server boot, command dispatchers, worker boot).
3. Map inbound interfaces and auth expectations.
4. Map outbound dependencies and transports.

### Phase 2 - Flow tracing

Trace 2-4 primary end-to-end flows (request/command -> orchestration -> integrations -> persistence).

### Phase 3 - Draft modeling

Create:

- Draft A: conservative grouping
- Draft B: explicit boundary model

Apply lossiness check to each draft.

### Phase 4 - Selection

Score A and B with the rubric, then select one.

### Phase 5 - Finalization

1. Generate final C4 PlantUML.
2. Render SVG.
3. Validate outputs.
4. Write notes in required structure.
5. Run a preflight correctness sanity pass on your own output:
   - check inferred markers,
   - check route/contract parity for cross-runtime edges,
   - check that high-risk claims are evidence-anchored.

### Phase 6 - Independent correctness check (recommended)

If available, run a separate verifier using a correctness-checker prompt.

- Generate `c4-correctness-report.md`.
- If findings exist, present a minimal correction plan.
- Offer to apply corrections first; do not auto-apply unless explicitly requested or `APPLY_CORRECTIONS=true`.

---

## Required notes structure (`c4-analysis-notes.md`)

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
15. Inferred claims register (if any)
16. Cross-runtime contract checks and outcomes
17. Checker handoff summary (if independent checker run)

---

## Technical conventions

- Use C4-PlantUML with `!include <C4/C4_Container>`.
- Prefer concise labels and short relationship verbs.
- Prefer `LAYOUT_LEFT_RIGHT()` unless readability is better otherwise.
- Keep context actors/systems only as needed for clarity.

---

## Validation checklist

1. `.puml` renders without errors.
2. `.svg` exists and is non-empty.
3. Diagram includes critical containers and external dependencies.
4. Relationship labels are evidence-backed and technically accurate.
5. Selected model is justified by rubric scores (not minimal-risk default).

---

## Rendering commands

- Preferred:
  - `plantuml -tsvg <path-to-puml>`
- Fallback:
  - `docker run --rm -v "$PWD":/work -w /work plantuml/plantuml -tsvg <path-to-puml>`

---

## Prompt design notes (why this works)

- Uses explicit instruction-first structure and clear delimiters.
- Decomposes a complex task into chained phases.
- Enforces structured outputs for easier review and comparison.
- Uses self-consistency via dual-draft scoring and selection.
- Encourages specificity while avoiding unsupported assumptions.
