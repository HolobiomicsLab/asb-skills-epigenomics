---
name: bit-rank-table-parsing
description: Use when you have BIT rank tables from cancer-type-specific accessible
  regions and need to determine which top-ranked TRs are functionally essential via
  DepMap CRISPR Chronos knockout scores.
when_to_use_negative:
- When the query region set has not yet been scored by BIT — run the BIT Bayesian
  hierarchical model first to obtain TR, BIT_score, and Rank columns
- When assessing TR essentiality by gene expression or survival (e.g., Kaplan-Meier
  plotter correlation) rather than DepMap CRISPR/Cas9 knockout viability
- When working with scRNA-seq or perturbation-seq data where functional essentiality
  is not defined by Chronos gene effect scores
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0203
tools:
- name: BIT
  role: Generates the ranked TR tables (TR, BIT_score, Rank) from cancer-type-specific
    accessible regions that this skill parses and intersects with DepMap data
  repo: https://github.com/ZeyuL01/BIT
- name: DepMap
  role: Provides the CRISPR/Cas9 Chronos gene effect scores used to define functional
    essentiality of top-ranked TRs per cancer lineage
- name: R
  role: Implementation language; the BIT R package and downstream parsing/validation
    scripts are written in R
  repo: https://github.com/ZeyuL01/BIT
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-60269-4
    title: 'BIT: Bayesian Identification of Transcriptional regulators from epigenomics-based
      query region sets'
  source_content_hash: sha256:6cba793189c15c3a241190a10b31aabc5db1fc8421200e808fb1d2acd1287493
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ZeyuL01/BIT
  repo_url: https://github.com/ZeyuL01/BIT
  tools:
  - BIT
  - DepMap
  - R
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# bit-rank-table-parsing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parses per-cancer-type BIT output rank tables (TR, BIT_score, Rank columns) and intersects top-ranked transcriptional regulators with DepMap Chronos gene-effect scores to classify functionally essential TRs using the Chronos < -0.4 threshold.

## When to use

Use when you have BIT rank tables from cancer-type-specific accessible regions and need to determine which top-ranked TRs are functionally essential via DepMap CRISPR Chronos knockout scores.

## When NOT to use

- When the query region set has not yet been scored by BIT — run the BIT Bayesian hierarchical model first to obtain TR, BIT_score, and Rank columns
- When assessing TR essentiality by gene expression or survival (e.g., Kaplan-Meier plotter correlation) rather than DepMap CRISPR/Cas9 knockout viability
- When working with scRNA-seq or perturbation-seq data where functional essentiality is not defined by Chronos gene effect scores

## Inputs

- Per-cancer-type BIT rank tables with TR, BIT_score, and Rank columns (derived from TCGA cancer-type-specific accessible regions)
- DepMap Chronos gene effect scores (CRISPR/Cas9 knockout viability data) per cancer lineage

## Outputs

- Per-cancer-type essential-TR count summary CSV (one row per cancer type with an essential-TR count field)
- Lineage-level Chronos score assignments per TR with essential (Chronos < -0.4) classification

## How to apply

Load the per-cancer-type BIT rank tables containing TR, BIT_score, and Rank columns generated from TCGA cancer-type-specific accessible regions. Load DepMap Chronos gene effect scores and, for each cancer lineage, compute the lineage-level Chronos score for each TR, defining a TR as functionally essential within a lineage when its Chronos score falls below the -0.4 threshold. For each cancer type, intersect the top-ranked BIT TRs (e.g., top-10 and top-50) with the essential set and count essential TRs, then write per-cancer-type essential TR counts to a summary CSV table. Validate that the output CSV has one row per cancer type with an essential-TR count field matching a recomputation of the intersection.

## Related tools

- **BIT** (Generates the ranked TR tables (TR, BIT_score, Rank) from cancer-type-specific accessible regions that this skill parses and intersects with DepMap data) — https://github.com/ZeyuL01/BIT
- **DepMap** (Provides the CRISPR/Cas9 Chronos gene effect scores used to define functional essentiality of top-ranked TRs per cancer lineage)
- **R** (Implementation language; the BIT R package and downstream parsing/validation scripts are written in R) — https://github.com/ZeyuL01/BIT

## Examples

```
Rscript count_essential_TRs.R --bit-ranks bit_rank_BRCA.csv --chronos depmap_chronos.csv --threshold -0.4 --top 10 --out essential_TR_counts.csv
```

## Evaluation signals

- Output CSV exists and contains exactly one row per cancer type (nine cancer types in the reference analysis) with an essential-TR count field
- Each essential-TR count matches an independent recomputation of the intersection between top-ranked BIT TRs and the Chronos < -0.4 essential set for that lineage
- Reproduces the paper's benchmark counts: 52 of 90 top-10 TRs and 232 of 450 top-50 TRs with minimum Chronos scores less than -0.4 across the nine cancer types
- All TR names in the BIT rank table resolve to entries in the DepMap Chronos gene effect matrix (no unmatched gene identifiers)

## Limitations

- The -0.4 Chronos threshold is a specific operational definition of functional essentiality; other cutoffs will change which TRs are counted as essential
- BIT's performance can be reduced by biological complexities of TR regulation, including dynamic TR binding across cell types and time, chromatin structure and epigenetic modifications affecting binding affinity/specificity, and cooperative binding among TRs
- Multiple types of sequencing biases, such as GC content bias or PCR amplification bias, can reduce the reliability of the input epigenomic region set and binding profiles from which the rank tables derive

## Evidence

- [methods] task workflow: [evidence span withheld — anchor status: not_found]
- [results] essentiality threshold: [evidence span withheld — anchor status: not_found]
- [other] essentiality definition: [evidence span withheld — anchor status: not_found]
