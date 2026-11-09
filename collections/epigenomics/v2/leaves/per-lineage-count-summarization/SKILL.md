---
name: per-lineage-count-summarization
description: Use when you have per-cancer-type BIT rank tables and DepMap CRISPR/Cas9
  Chronos gene effect data and need per-lineage counts of functionally essential TRs.
when_to_use_negative:
- When the goal is to rank or identify TRs from an epigenomic query region set itself
  — that is BIT's job; this skill only post-processes BIT ranks against dependency
  data.
- When the input is per-cell-line rather than per-lineage Chronos data and lineage-level
  scores cannot be computed for the nine cancer types.
- When validating TR essentiality by a criterion other than knockout viability (e.g.,
  survival correlation via Kaplan-Meier plotter), since the -0.4 Chronos cutoff applies
  specifically to CRISPR/Cas9 gene effect scores.
tools:
- name: BIT
  role: Produces the per-cancer-type TR rank tables whose top-10 and top-50 TRs are
    tested for essentiality
  repo: https://github.com/ZeyuL01/BIT
- name: R
  role: Language used to load rank tables, compute lineage-level Chronos scores, apply
    the -0.4 threshold, and write the summary CSV
- name: DepMap
  role: Source of CRISPR/Cas9 Chronos gene effect scores defining the essential gene
    sets per cancer lineage
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-60269-4
    title: 'BIT: Bayesian Identification of Transcriptional regulators from epigenomics-based
      query region sets'
  source_content_hash: sha256:b199a947812b6750da71567c600beac0eda05d64231fac1a5524cbbbb7470ccb
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
yes_rate: 0.7
n_claims: 10
---

# per-lineage-count-summarization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Summarizes how many BIT-ranked transcriptional regulators (TRs) are functionally essential in each cancer lineage by intersecting BIT rank tables with lineage-level DepMap Chronos gene-effect scores under a strict < -0.4 threshold, and writing one row per cancer type to a summary CSV.

## When to use

Use when you have per-cancer-type BIT rank tables and DepMap CRISPR/Cas9 Chronos gene effect data and need per-lineage counts of functionally essential TRs.

## When NOT to use

- When the goal is to rank or identify TRs from an epigenomic query region set itself — that is BIT's job; this skill only post-processes BIT ranks against dependency data.
- When the input is per-cell-line rather than per-lineage Chronos data and lineage-level scores cannot be computed for the nine cancer types.
- When validating TR essentiality by a criterion other than knockout viability (e.g., survival correlation via Kaplan-Meier plotter), since the -0.4 Chronos cutoff applies specifically to CRISPR/Cas9 gene effect scores.

## Inputs

- Per-cancer-type BIT rank tables with columns TR, BIT_score, Rank (from TCGA cancer-type-specific accessible regions)
- DepMap CRISPR/Cas9 Chronos gene effect scores mapped to cancer lineages

## Outputs

- Summary CSV table with one row per cancer type (nine cancer types) and a functionally-essential TR count field

## How to apply

Load the per-cancer-type BIT rank tables (columns: TR, BIT_score, Rank) derived from TCGA cancer-type-specific accessible regions. Load the DepMap Chronos gene effect scores and, for each of the nine cancer lineages, compute the lineage-level Chronos score for each TR; a TR is classified as functionally essential when its Chronos score is below the -0.4 threshold (the cutoff the paper used to define essentiality from knockout viability effects). For each cancer type, count how many of the top-ranked BIT TRs (top-10 and top-50 lists) fall in the essential set, then write a summary CSV with one row per cancer type and an essential-TR count field. The rationale is that BIT ranks TRs from cancer-type-specific accessible regions, and Chronos < -0.4 indicates that knockout measurably reduces cell viability, so the intersection measures how often BIT's top predictions are dependencies.

## Related tools

- **BIT** (Produces the per-cancer-type TR rank tables whose top-10 and top-50 TRs are tested for essentiality) — https://github.com/ZeyuL01/BIT
- **R** (Language used to load rank tables, compute lineage-level Chronos scores, apply the -0.4 threshold, and write the summary CSV)
- **DepMap** (Source of CRISPR/Cas9 Chronos gene effect scores defining the essential gene sets per cancer lineage)

## Examples

```
Rscript count_essential_TRs.R --ranks bit_ranks.csv --chronos chronos_scores.csv --threshold -0.4 --out essential_TR_counts.csv
```

## Evaluation signals

- Output CSV exists and contains exactly one row per cancer type (nine rows) with a well-formed essential-TR count field.
- Each per-lineage count equals a recomputation of the intersection between the top-ranked BIT TRs and the Chronos < -0.4 essential set for that lineage.
- Across all nine cancer types, top-10 counts sum to roughly 52 of 90 and top-50 counts to roughly 232 of 450, matching the reported finding (Fig. 4d).
- Threshold is applied strictly as minimum Chronos score < -0.4 (not <=), matching the paper's definition.
- Counts are monotonically non-decreasing when moving from top-10 to top-50 TR lists for every lineage.

## Limitations

- The -0.4 Chronos cutoff is a single fixed threshold; it does not account for lineage-specific screening quality or gene-effect score uncertainty.
- Essentiality is defined by knockout viability in DepMap cell lines, which may not transfer to primary tumor contexts.
- Counts depend on BIT's ranking quality, which the paper notes can suffer from biological complexities of TR regulation such as dynamic binding across cell types and cooperative binding.

## Evidence

- [results] Half of BIT TRs essential under Chronos < -0.4: [evidence span withheld — anchor status: not_found]
