# Prompting Techniques That Actually Work

### A case study: five iterations from generic prompt to evidence-backed architecture diagrams

We used AI to produce C4 architecture diagrams for the [OpenCode](https://github.com/sst/opencode) codebase. We started with a one-line prompt and got mediocre results. Over five iterations, we systematically improved our prompts using specific techniques until the output was evidence-backed, reviewer-approved, and caught architectural issues that humans had missed.

This repository contains everything from that process: the prompts, the diagrams from each iteration, the analysis notes, the correctness reports, and the article that tells the full story.

**Read the article:** [Prompting Techniques That Actually Work](article/prompting-techniques-article.md) (also on [dev.to](https://dev.to/uenyioha))

---

## What's here

```
prompts/                        # Reusable prompt templates
  generator-prompt.md           # Final agentic C4 generator prompt
  checker-prompt.md             # Independent correctness checker prompt
  reasoning-protocol.md         # 12-step reasoning protocol (now inlined into generator)

diagrams/                       # C4 diagrams from each iteration
  original/                     # First attempt (project-specific prompt)
  generic/                      # Iteration 1: one-line generic prompt
  protocol/                     # Iteration 2: structured + phased prompt
  agentic/                      # Iteration 3: dual drafts + scoring rubric
  final/                        # Iteration 5: post-checker corrections applied

analysis-notes/                 # Reasoning notes from each iteration
  c4-analysis-notes-generic.md  # 31 lines, 3 sections, no scoring
  c4-analysis-notes-protocol.md # 101 lines, 12 sections, evidence-anchored
  c4-analysis-notes-agentic.md  # 146 lines, 14 sections, Draft A/B scoring
  c4-analysis-notes-final.md    # 145 lines, 17 sections, post-correction

correctness-reports/            # Independent checker findings
  c4-correctness-report-final.md         # Initial check (confidence: 84)
  c4-correctness-report-final-postfix.md # Post-correction check (confidence: 90)

article/                        # The article itself
  prompting-techniques-article.md    # Full article (markdown)
  prompting-techniques-article.html  # HTML version with styling
  prompting-techniques-article.docx  # Word version
  prompting-techniques-devto.md      # dev.to version with hosted images
  article-generation-prompt.md       # The prompt used to generate the article
  earlier-draft.md                   # First draft (for comparison)
```

## The progression

| Iteration | Techniques added | Local containers | Notes | Key change |
|---|---|---:|---|---|
| Original | Project-specific prompt | 10 | Unstructured | Too granular, no boundaries |
| Generic | None (one-liner) | 6 | 31 lines, 3 sections | Merged "Integration Gateway" |
| Protocol | Instruction-first, structured output, phases, evidence anchors | 7 | 101 lines, 12 sections | Better notes, still merged |
| Agentic | Dual drafts, scoring rubric, lossiness checks | 8 | 146 lines, 14 sections | Provider/MCP/Plugin split |
| Final | Independent checker, correction loops, feedback | 8 + corrections | 145 lines, 17 sections | Share gateway, inferred markers |

## The ten techniques

1. **Instruction-first prompting** -- Lead with the task, not the context
2. **Structured output requirements** -- Fixed sections, no omissions, "N/A" for empty
3. **Prompt chaining** -- Sequential phases, each producing artifacts for the next
4. **Dual drafts + scoring rubric** -- Draft A/B with numeric scoring prevents first-answer bias
5. **Evidence-grounded claims** -- File:line anchors for every major claim
6. **Lossiness checks** -- For every merge, state what signal is lost
7. **Independent checker** -- Separate agent audits claims as VERIFIED/PARTIAL/UNVERIFIED/INCORRECT
8. **Correction loops** -- Offer before apply, no scope expansion
9. **Feedback loops** -- Checker findings become generator requirements
10. **Self-critique** -- One pass before the checker, not after

## Using the prompts

The prompts in `prompts/` are designed to be reusable. They reference a generic "target codebase" rather than OpenCode specifically. To use them:

1. Point the **generator prompt** at your codebase and run it
2. Review the dual drafts and scoring table
3. Run the **checker prompt** against the generator's output
4. Apply corrections surgically
5. Feed checker findings back into the generator prompt

## License

The prompts, article, and analysis artifacts in this repository are released under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). The C4 diagrams depict the architecture of [OpenCode](https://github.com/sst/opencode), which is licensed under its own terms.
