---
name: cancer-lineage-specific-essentiality-thresholding
description: Use when a ranked list of candidate TRs (e.g., BIT scores from TCGA cancer-type-specific
  accessible regions) must be checked for knockout-validated essentiality using DepMap
  CRISPR/Cas9 Chronos scores per cancer lineage.
when_to_use_negative:
- The query TRs come from a lineage or tissue not represented in the DepMap Chronos
  screening data — no lineage-level score exists to threshold.
- The input is a gene set rather than a ranked TR list; use enrichment-style tools
  (e.g., BART geneset mode) instead of dependency-score intersection.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0622
tools:
- name: BIT
  role: Produces the per-cancer-type TR rank tables (TR, BIT_score, Rank) that are
    intersected with the essential set
  repo: https://github.com/ZeyuL01/BIT
- name: DepMap
  role: Source of CRISPR/Cas9 Chronos gene effect scores used to define functional
    essentiality per cancer lineage
- name: R
  role: Implementation language for thresholding, intersection, and CSV summary generation
  repo: https://github.com/ZeyuL01/BIT
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-60269-4
    title: 'BIT: Bayesian Identification of Transcriptional regulators from epigenomics-based
      query region sets'
  source_content_hash: sha256:b14f8f2d4dff6cbc95ea74f131deb75688bb232ab8cad3f22d9c6845d0575c7e
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
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# cancer-lineage-specific-essentiality-thresholding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates computationally ranked transcriptional regulators (TRs) against functional dependency data by intersecting top-ranked BIT TRs with lineage-level DepMap Chronos gene effect scores and classifying a TR as functionally essential when its Chronos score falls below -0.4 in a given cancer lineage.

## When to use

Use when a ranked list of candidate TRs (e.g., BIT scores from TCGA cancer-type-specific accessible regions) must be checked for knockout-validated essentiality using DepMap CRISPR/Cas9 Chronos scores per cancer lineage.

## When NOT to use

- The query TRs come from a lineage or tissue not represented in the DepMap Chronos screening data — no lineage-level score exists to threshold.
- The input is a gene set rather than a ranked TR list; use enrichment-style tools (e.g., BART geneset mode) instead of dependency-score intersection.

## Inputs

- Per-cancer-type BIT rank tables with columns TR, BIT_score, Rank (from TCGA cancer-type-specific accessible regions)
- DepMap CRISPR/Cas9 Chronos gene effect score matrix (gene x cell line/lineage)
- Cancer lineage assignment for each DepMap cell line (nine cancer types in the study)

## Outputs

- Per-cancer-type summary CSV with essential-TR counts among top-ranked BIT TRs
- Lineage-level Chronos score per TR with essential/non-essential classification (threshold -0.4)

## How to apply

Load the per-cancer-type BIT rank tables (TR, BIT_score, Rank columns) derived from TCGA cancer-type-specific accessible regions, and load DepMap Chronos gene effect scores. For each cancer lineage, compute the lineage-level Chronos score for each TR and apply the -0.4 threshold: a TR with minimum Chronos score < -0.4 across the lineage is classified as functionally essential. Count, per cancer type, how many of the top-ranked BIT TRs (top-10 and top-50) fall in the essential set, and write per-cancer-type essential-TR counts to a summary CSV. The rationale is that DepMap CRISPR/Cas9 screens evaluate cancer cell viability after gene knockout, so a strongly negative Chronos score indicates the TR's loss compromises viability, i.e., functional essentiality.

## Related tools

- **BIT** (Produces the per-cancer-type TR rank tables (TR, BIT_score, Rank) that are intersected with the essential set) — https://github.com/ZeyuL01/BIT
- **DepMap** (Source of CRISPR/Cas9 Chronos gene effect scores used to define functional essentiality per cancer lineage)
- **R** (Implementation language for thresholding, intersection, and CSV summary generation) — https://github.com/ZeyuL01/BIT

## Examples

```
essential <- tr %>% group_by(lineage) %>% summarise(essential_top10 = sum(top10_tr %in% names(which(chronos_min < -0.4))))
```

## Evaluation signals

- Output CSV exists and contains one row per cancer type with an essential-TR count field.
- Each count matches an independent recomputation of the intersection between top-ranked BIT TRs and the Chronos < -0.4 essential set for that lineage.
- Reported proportions fall in the expected range from the paper: 52 of 90 top-10 TRs and 232 of 450 top-50 TRs essential across the nine cancer types.
- All TRs in the BIT rank tables are matched to a gene identifier present in the Chronos matrix (no silent dropouts).
- Threshold check is applied on minimum lineage-level Chronos score, not mean, so borderline TRs with one strongly negative score are classified consistently.

## Limitations

- The -0.4 Chronos threshold is an arbitrary operational cutoff for essentiality; results are sensitive to its choice.
- DepMap coverage is cell-line-based, so lineage-level scores may not represent all tumor contexts within a cancer type.
- BIT's identified TRs may not be essential even when genuinely regulating a process — binding activity does not guarantee knockout-induced viability loss.

## Evidence

- [results] Approximately half of the BIT-identified cancer-type TRs were functionally essential: 52 of 90 top-10 TRs and 232 of 450 top-50 TRs had minimum Chronos scores less than −0.4 across the nine cancer types: [evidence span withheld — anchor status: not_found]
