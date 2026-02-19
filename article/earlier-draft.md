---
title: Prompting patterns
description: Practical prompt habits that produce better architecture outputs
---

# Prompting patterns

Good prompts turn architecture work from "looks about right" into "reviewable and traceable."
When you're doing C4-style analysis, the way you prompt directly affects how clear boundaries are, how solid the evidence is, and how fast reviews go.

This guide walks through patterns we've found reliable — each one with a definition, a concrete example, and a note on why it helps.

---

## Start with instruction-first prompts

Put the task up front, then your constraints, then any background context.
When the model sees the action first, it stays on track instead of wandering into tangential detail.

- **What this means:** Lead with what you want done, follow with hard limits, and attach references last.
- **How we used it:** In our C4 passes, we asked for container-level boundaries before mentioning individual files.
- **Why it helped:** The model stuck to architecture-level thinking instead of diving into low-level component noise.

```txt
Task: Produce a container-level architecture summary.
Constraints: No component-level decomposition. Include only runtime boundaries.
Context: Use provided file paths as evidence anchors.
Output: 8-12 containers, each with a one-line responsibility.
```

---

## Require structured output

If you want reviews to be consistent, ask for a fixed structure.
Freeform prose can read well, but it's much harder to verify or compare across drafts.

- **What this means:** Spell out exactly which sections, tables, or bullet formats you expect.
- **How we used it:** We required scope, boundaries, flows, a scoring table, and a final selection — every time.
- **Why it helped:** Reviewers could compare drafts side by side and spot gaps immediately.

```txt
Return exactly these sections:
1) Scope and non-goals
2) Runtime boundaries
3) Key flows
4) Draft scoring table
5) Final selection with rationale
```

---

## Chain prompts deliberately

Complex outputs get better when you break them into stages.
Each stage should produce a concrete artifact that the next stage consumes — not just "more context."

- **What this means:** Split one hard task into sequential prompts with clear handoffs between them.
- **How we used it:** We moved through discovery → draft model → comparison → final model, one step at a time.
- **Why it helped:** Mistakes showed up early, before they could cascade into the final diagram.

```txt
Stage 1: Extract candidate boundaries and cite file evidence.
Stage 2: Build Draft A and Draft B from Stage 1 output only.
Stage 3: Score drafts against fixed criteria and choose one.
```

---

## Score self-consistency with Draft A/B

Ask for two valid drafts, then score both against a fixed rubric.
This keeps you from locking onto the first plausible answer, which is a common trap when working with LLMs.

- **What this means:** Generate alternatives, apply a scoring rubric, and make the selection explicit.
- **How we used it:** We compared merged vs. split integration boundaries in our C4 drafts.
- **Why it helped:** Tradeoffs became visible — especially the tension between fidelity and readability.

```txt
Create Draft A and Draft B.
Score each 1-5 for fidelity, explanatory power, readability, and boundary quality.
Select the winner and justify each score in one sentence.
```

---

## Ground claims in evidence

Architecture statements should point to something concrete.
Every major claim needs at least one anchor — a file path, a line number, a config entry.

- **What this means:** Require file-path evidence for boundaries, flows, and responsibilities.
- **How we used it:** Our C4 notes tied each claim to a runtime entry point or integration module.
- **Why it helped:** Review shifted from "I think this is right" to "here's the proof."

```txt
For every architectural claim, attach at least one evidence anchor.
Use format: <claim> -> <file path:line>.
Mark claims with no anchor as "unverified".
```

---

## Add anti-hallucination checks

Tell the model what to do when evidence is missing.
An honest "I don't know" is far more useful than confident-sounding fabrication.

- **What this means:** Force uncertainty labels and missing-evidence flags into the output format.
- **How we used it:** We required "inferred" labels for actors that weren't directly implemented in code.
- **Why it helped:** It stopped the model from inventing internals and made assumptions visible to reviewers.

```txt
Do not infer implementation details without evidence.
If evidence is missing, output "unknown" and list what data is needed.
Tag inferred elements as "inferred" with a confidence level.
```

---

## Run an independent correctness pass

Use a second prompt as a checker — not a rewriter.
The checker's job is to validate claims against evidence and report what doesn't hold up.

- **What this means:** Separate the generation step from the verification step to reduce confirmation bias.
- **How we used it:** After picking a winning draft, we ran a dedicated correctness-checker pass.
- **Why it helped:** It caught unsupported claims and places where boundaries leaked between containers.

```txt
Act as an independent checker.
Verify each claim against the provided evidence list.
Output: confirmed, contradicted, or unsupported, with brief notes.
```

---

## Use a correction loop before edits

When the checker finds issues, don't jump straight to rewriting.
Ask for a proposed fix first. Apply edits only after the proposals are reviewed and approved.

- **What this means:** Follow a "diagnose → propose → confirm → apply" cycle instead of a direct rewrite.
- **How we used it:** We reviewed proposed corrections and rationale before touching the final C4 notes.
- **Why it helped:** It prevented accidental drift — small "fixes" that quietly shift accepted boundaries.

```txt
Identify issues and propose minimal edits.
Do not rewrite the document yet.
Wait for confirmation, then apply only approved changes.
```

---

## Reuse a practical workflow

Here's a sequence you can follow when producing architecture docs with AI.
It's fast enough for daily work and structured enough to survive review.

1. Define the objective, scope, and non-goals in 3–6 lines.
2. Run a discovery prompt to extract boundaries and evidence anchors.
3. Generate Draft A and Draft B with a fixed scoring rubric.
4. Select a winner and require explicit tradeoff rationale.
5. Run an independent correctness checker on the selected draft.
6. Open a correction loop and approve only minimal edits.
7. Publish the final doc with claim-to-evidence traceability.

---

## Fix common failure modes

Most bad outputs fail in predictable ways.
Treat these as prompt design bugs — not model quirks.

- **Scope creep:** The model dives into component details → tighten scope to container level and explicitly forbid lower levels.
- **Pretty but vague:** Output reads well but has no verifiable anchors → require at least one anchor per major claim.
- **Single-draft lock-in:** First-answer bias takes hold → force Draft A/B with numeric scoring.
- **Hallucinated certainty:** Missing data gets papered over → require `unknown` tags and confidence levels.
- **Checker echo:** The verifier just restates the draft → run the checker with a strict "validate, do not rewrite" instruction.
- **Over-editing:** Minor findings trigger a full rewrite → use a correction loop with a minimal-patch rule.

---

## Use a review-ready checklist

Run through this before opening a PR.
If any item is false, go back and fix your prompts or evidence first.

- [ ] Scope and non-goals are explicit and testable.
- [ ] Output structure is fixed and fully populated.
- [ ] Draft A/B exists with rubric scores and rationale.
- [ ] Every major claim has at least one evidence anchor.
- [ ] Inferred items are labeled with a confidence level.
- [ ] Independent checker report is attached.
- [ ] Corrections were proposed before final edits were applied.
- [ ] Final document reflects only approved changes.

---

## Adapt these snippets to your work

These snippets are intentionally generic.
We used them in our C4 work by swapping in repo-specific file paths and evidence — you can do the same.

- **Scope keeps drifting?** Start with the instruction-first snippet.
- **Reviews feel inconsistent?** Add the structured-output snippet.
- **Tradeoffs are non-trivial?** Use chaining and Draft A/B together.
- **Finalizing a doc?** Layer on the checker and correction-loop snippets.

Keep these prompts in a shared team playbook and tune only the constraints for each task.
That gives you repeatable architecture outputs without overfitting to one codebase.
