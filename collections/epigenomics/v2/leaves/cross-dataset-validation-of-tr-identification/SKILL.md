---
name: cross-dataset-validation-of-tr-identification
description: Use when a ranked TR list (e.g., BIT scores/ranks from TCGA cancer-type
  accessible regions) must be independently validated by asking whether top-ranked
  TRs are essential upon CRISPR/Cas9 knockout in the same cancer lineages (DepMap
  Chronos).
when_to_use_negative:
- The TR list is unranked or has no BIT scores/ranks — the validation depends on intersecting
  top-ranked TRs with the essential set
- The goal is de novo TR identification from motifs or ChIP-seq alone (e.g., WhichTF/HOMER,
  BART) rather than post hoc functional validation of an existing ranked list
- The knockout data perturb only coding-gene viability but the query regions target
  non-coding regulatory elements without a corresponding gene-level essentiality readout
edam_topics:
- http://edamontology.org/topic_0204
tools:
- name: BIT
  role: Supplies the top-ranked TR lists (BIT_score, Rank) per cancer type that are
    validated against DepMap Chronos essentiality
  repo: https://github.com/ZeyuL01/BIT
- name: DepMap
  role: Source of CRISPR/Cas9 knockout Chronos gene-effect scores used as the independent
    functional-essentiality benchmark
- name: R
  role: Implementation language for joining BIT rank tables with Chronos scores, applying
    the −0.4 threshold, and writing the summary CSV
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-60269-4
    title: 'BIT: Bayesian Identification of Transcriptional regulators from epigenomics-based
      query region sets'
  source_content_hash: sha256:b21c7d5e39f58af5a46c2b8b27350e00c21bb6bb31c10c5e2b833c71f35c0de4
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
  edam_topics:
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# Cross-dataset validation of TR identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates computationally ranked transcriptional regulators (e.g., BIT-ranked TRs from cancer-type-specific accessible regions) against an independent functional dataset — DepMap CRISPR/Cas9 Chronos gene-effect scores — to determine what fraction of top-ranked TRs are functionally essential genes.

## When to use

Use when a ranked TR list (e.g., BIT scores/ranks from TCGA cancer-type accessible regions) must be independently validated by asking whether top-ranked TRs are essential upon CRISPR/Cas9 knockout in the same cancer lineages (DepMap Chronos).

## When NOT to use

- The TR list is unranked or has no BIT scores/ranks — the validation depends on intersecting top-ranked TRs with the essential set
- The goal is de novo TR identification from motifs or ChIP-seq alone (e.g., WhichTF/HOMER, BART) rather than post hoc functional validation of an existing ranked list
- The knockout data perturb only coding-gene viability but the query regions target non-coding regulatory elements without a corresponding gene-level essentiality readout

## Inputs

- Per-cancer-type BIT rank tables (TR, BIT_score, Rank) derived from TCGA cancer-type-specific accessible regions
- DepMap Chronos gene effect scores (per-cell-line/per-gene CRISPR/Cas9 knockout viability, aggregated per cancer lineage)
- Lineage mapping between BIT cancer types and the nine DepMap cancer lineages

## Outputs

- Summary CSV with one row per cancer type and an essential-TR count field (top-10 and/or top-50 BIT TRs classified as Chronos < −0.4)
- Per-lineage essential TR set (TRs with lineage-level Chronos score < −0.4)

## How to apply

Load the per-cancer-type BIT rank tables (columns TR, BIT_score, Rank) generated from TCGA cancer-type-specific accessible regions, then load DepMap Chronos gene-effect scores and compute the lineage-level Chronos score for each TR within its matching cancer lineage. Classify a TR as functionally essential in a lineage when its Chronos score falls below the −0.4 threshold. For each cancer type, count how many of the top-ranked BIT TRs (e.g., top-10 and top-50 per the BIT ranking) are essential under this threshold, and write the per-cancer-type essential-TR counts to a summary CSV. Validation follows the paper's approach of intersecting the top-ranked BIT TR set with the Chronos < −0.4 essential set per lineage and comparing the counts.

## Related tools

- **BIT** (Supplies the top-ranked TR lists (BIT_score, Rank) per cancer type that are validated against DepMap Chronos essentiality) — https://github.com/ZeyuL01/BIT
- **DepMap** (Source of CRISPR/Cas9 knockout Chronos gene-effect scores used as the independent functional-essentiality benchmark)
- **R** (Implementation language for joining BIT rank tables with Chronos scores, applying the −0.4 threshold, and writing the summary CSV)

## Evaluation signals

- Output CSV exists and contains exactly one row per cancer type (nine cancer types expected) with an essential-TR count field
- Each count equals a recomputed intersection of the top-ranked BIT TR set with the Chronos < −0.4 essential set for that lineage
- Aggregate result is consistent with the published benchmark: 52 of 90 top-10 TRs and 232 of 450 top-50 TRs had minimum Chronos scores less than −0.4 across the nine cancer types
- Lineage-level Chronos aggregation matches the cancer lineage of each BIT rank table (no cross-lineage mixing of scores)

## Limitations

- The −0.4 Chronos threshold is a fixed essentiality cutoff; TRs near the cutoff are sensitive to lineage aggregation and cell-line composition within the lineage
- CRISPR knockout essentiality reflects only viability effects in cell lines and may miss TRs whose function is context-dependent or not captured by the screened lineages

## Evidence

- [results] Approximately half of the BIT-identified cancer-type TRs were functionally essential: 52 of 90 top-10 TRs and 232 of 450 top-50 TRs had minimum Chronos scores less than −0.4 across the nine cancer types (Fig. 4d).: [evidence span withheld — anchor status: not_found]
