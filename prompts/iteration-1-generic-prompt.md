# Iteration 1: Generic Prompt

This is the actual prompt used for the first (generic) pass. It was deliberately minimal — a one-liner to establish a baseline.

---

## Prompt

Produce a C4 container diagram for this repository. Include local runtime containers, client surfaces, and cloud services. Generate a PlantUML `.puml` file and render it to SVG. Write brief analysis notes documenting your modeling choices.

---

## What this produced

- 6 local containers, including a merged "Integration Gateway"
- 31 lines of freeform analysis notes
- No scoring, no alternatives, no evidence anchors for most claims
- Valid diagram, but not useful for real engineering decisions

## What was missing

- No instruction-first framing (role, constraints, output format)
- No phased workflow
- No structured output requirements
- No dual drafts or scoring rubric
- No lossiness checks
- No evidence anchor requirements
- No inferred-claim markers
