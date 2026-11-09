---
name: depmap-chronos-essentiality-classification
description: Use when you have a ranked list of candidate TRs (e.g., BIT scores from
  TCGA-derived accessible regions) and need to quantify what fraction are functionally
  essential using DepMap CRISPR knockout viability data.
when_to_use_negative:
- When the candidate TR list comes from motif-only prediction without epigenomic region
  support — Chronos essentiality validation is designed to follow BIT-style region-based
  rankings
- When the goal is to predict essentiality de novo rather than validate already-ranked
  TRs against knockout viability data
- When Chronos gene effect scores are unavailable for the target lineage, since the
  -0.4 threshold requires lineage-level scores
tools:
- name: BIT
  role: Produces the top-ranked TR lists from cancer-type-specific accessible regions
    that are then tested for essentiality
  repo: https://github.com/ZeyuL01/BIT
- name: R
  role: Environment for computing lineage-level Chronos scores, applying the -0.4
    threshold, and writing the summary CSV
- name: DepMap
  role: Source of CRISPR/Cas9 screening data evaluating cancer cell viability after
    gene knockout
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-60269-4
    title: 'BIT: Bayesian Identification of Transcriptional regulators from epigenomics-based
      query region sets'
  source_content_hash: sha256:12ac1ccd174a93b734ca7bbf68f0e7b7345f996d63dc62c9c6736029915d5be8
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# DepMap Chronos essentiality classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classifies top-ranked transcriptional regulators (TRs) — e.g., BIT-ranked TRs from cancer-type-specific accessible regions — as functionally essential by intersecting them with DepMap CRISPR/Cas9 Chronos gene-effect scores using a -0.4 essentiality threshold per cancer lineage.

## When to use

Use when you have a ranked list of candidate TRs (e.g., BIT scores from TCGA-derived accessible regions) and need to quantify what fraction are functionally essential using DepMap CRISPR knockout viability data.

## When NOT to use

- When the candidate TR list comes from motif-only prediction without epigenomic region support — Chronos essentiality validation is designed to follow BIT-style region-based rankings
- When the goal is to predict essentiality de novo rather than validate already-ranked TRs against knockout viability data
- When Chronos gene effect scores are unavailable for the target lineage, since the -0.4 threshold requires lineage-level scores

## Inputs

- Per-cancer-type BIT rank tables (CSV with TR, BIT_score, Rank columns) from TCGA cancer-type-specific accessible regions
- DepMap CRISPR/Cas9 Chronos gene effect score matrix (gene x cell line/lineage)
- Cancer lineage assignment for DepMap cell lines (nine cancer types)
- Essentiality threshold (-0.4) and top-k cutoff (top-10, top-50)

## Outputs

- Per-cancer-type essential-TR counts summary CSV (one row per cancer type with an essential-TR count field)
- Binary essential/non-essential classification per TR per lineage

## How to apply

Load the per-cancer-type TR rank tables (TR, BIT_score, Rank columns) generated from TCGA cancer-type-specific accessible regions. Load the DepMap Chronos gene effect scores and, for each of the nine cancer lineages, compute the lineage-level Chronos score for each TR. Classify a TR as functionally essential within a lineage when its Chronos score falls below the -0.4 threshold, using the minimum Chronos score as the summary statistic. For each cancer type, count how many top-ranked BIT TRs (e.g., top-10 and top-50 lists) fall in the essential set, and write per-cancer-type essential-TR counts to a summary CSV. In the BIT paper this showed that roughly half of identified TRs were essential (52 of 90 top-10 and 232 of 450 top-50 TRs across nine cancer types, Fig. 4d), demonstrating that BIT rankings enrich for cancer dependencies.

## Related tools

- **BIT** (Produces the top-ranked TR lists from cancer-type-specific accessible regions that are then tested for essentiality) — https://github.com/ZeyuL01/BIT
- **R** (Environment for computing lineage-level Chronos scores, applying the -0.4 threshold, and writing the summary CSV)
- **DepMap** (Source of CRISPR/Cas9 screening data evaluating cancer cell viability after gene knockout)

## Examples

```
essential <- top_trs[top_trs$TR %in% names(which(apply(chronos[lineage_rows, ], 1, min, na.rm = TRUE) < -0.4)), ]
```

## Evaluation signals

- Output CSV exists and contains one row per cancer type with an essential-TR count field
- Each essential-TR count matches an independent recomputation of the intersection between top-ranked BIT TRs and the Chronos < -0.4 essential set for that lineage
- Counts are bounded: essential counts must not exceed the top-k list size (e.g., <=10 for top-10, <=50 for top-50) and total counts should be comparable to the published 52/90 top-10 and 232/450 top-50 across the nine cancer types
- Every classified TR has a defined minimum Chronos score for the corresponding lineage (no silent NA handling inflating or deflating counts)
- The same fixed threshold of -0.4 is applied uniformly across all nine cancer types

## Limitations

- The -0.4 Chronos threshold is a heuristic cutoff; essentiality assignments depend on this choice and on lineage-level score aggregation
- DepMap Chronos scores cover only the cell lines screened, so TRs not expressed or not screened in a lineage may be misclassified as non-essential
- BIT's own limitations (dynamic TR binding, chromatin context, sequencing bias) propagate into the TR lists being validated, affecting essentiality fractions

## Evidence

- [results] DepMap essentiality threshold and counts: [evidence span withheld — anchor status: not_found]
- [methods] Classification rule: [evidence span withheld — anchor status: not_found]
