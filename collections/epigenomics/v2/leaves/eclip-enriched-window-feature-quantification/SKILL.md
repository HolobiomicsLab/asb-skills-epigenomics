---
name: eclip-enriched-window-feature-quantification
description: Use when Skipper has produced reproducible_enriched_windows tables for
  two or more eCLIP conditions (e.g., wild-type vs missense-mutant RBP constructs)
  and the goal is to compare the fraction of binding sites per transcript region or
  per annotated locus class such as tRNA.
when_to_use_negative:
- CLIP datasets lack matched input controls — Skipper requires a matched input sample
  to construct the beta-binomial model, so enriched-window tables cannot be generated
  in the first place
- The question is mechanism-level (complex formation vs localization vs binding preference)
  — eCLIP window proportions alone cannot resolve this
- The input is raw CLIP reads rather than Skipper output — the end-to-end Skipper
  pipeline (preprocessing, alignment, GFF partitioning, enriched window identification)
  must run first
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_0102
tools:
- name: Skipper
  role: Produces the reproducible_enriched_windows tables and feature_type_top annotations
    that are aggregated by transcript region and tested for WT/mutant shifts
  repo: https://github.com/YeoLab/skipper
- name: Snakemake 9.12.0
  role: Workflow engine used to run the Skipper pipeline that generates the enriched-window
    tables
  repo: https://github.com/YeoLab/skipper
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.xgen.2023.100317
    title: Skipper analysis of eCLIP datasets enables sensitive detection of constrained
      translation factor binding sites
  source_content_hash: sha256:8544fa5b4841d2b2504fe27614846076182ee46d5f6e6664a1686c2e33458c9c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/YeoLab/skipper
  repo_url: https://github.com/YeoLab/skipper
  tools:
  - Skipper
  - Snakemake 9.12.0
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# eCLIP enriched-window feature quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies the distribution of Skipper's reproducible enriched windows across transcript feature classes (CDS, UTRs, deep introns, tRNA loci) and tests WT-vs-mutant shifts, as done for RPS19 eCLIP (R62W, R101H) where CDS binding rose 33%→45% and deep intronic binding fell 20%→7% (p < 2.2e−16).

## When to use

Use when Skipper has produced reproducible_enriched_windows tables for two or more eCLIP conditions (e.g., wild-type vs missense-mutant RBP constructs) and the goal is to compare the fraction of binding sites per transcript region or per annotated locus class such as tRNA.

## When NOT to use

- CLIP datasets lack matched input controls — Skipper requires a matched input sample to construct the beta-binomial model, so enriched-window tables cannot be generated in the first place
- The question is mechanism-level (complex formation vs localization vs binding preference) — eCLIP window proportions alone cannot resolve this
- The input is raw CLIP reads rather than Skipper output — the end-to-end Skipper pipeline (preprocessing, alignment, GFF partitioning, enriched window identification) must run first

## Inputs

- Skipper reproducible_enriched_windows tables for each eCLIP replicate (WT, R62W, R101H)
- Skipper GFF-derived feature annotations (feature_type_top per window) from a filtered GENCODE GFF3
- tRNA locus annotations for subsetting windows to annotated tRNA loci

## Outputs

- Per-sample feature-type proportion table (fraction of reproducible enriched windows per transcript region)
- WT-to-mutant shift statistics (e.g., CDS 33%→45%, deep intronic 20%→7%, p < 2.2e−16)
- Per-replicate tRNA enrichment values with paired t-test p-values
- Figure7e_feature_proportions.pdf and figure7f_trna_enrichment.pdf

## How to apply

Load the reproducible_enriched_windows tables emitted by Skipper for each sample (WT, R62W, R101H replicates), which Skipper generates by tiling windows over annotated transcripts from a filtered GENCODE GFF and testing read-start enrichment against a matched input via its beta-binomial model. Aggregate windows by the feature_type_top annotation within each sample to compute the fraction assigned to CDS, deep introns, UTRs, etc., then quantify WT-to-mutant shifts in these proportions (expected: CDS 33%→45%, deep intronic 20%→7%). For locus-class analyses such as tRNA binding, subset windows mapping to annotated tRNA loci, compute per-replicate enrichment, and apply paired t-tests across matched WT/mutant replicates. Render the region-proportion comparison and the tRNA enrichment comparison (with paired-test significance annotations) as figures (e.g., figure7e_feature_proportions.pdf and figure7f_trna_enrichment.pdf). Accept the analysis only if computed proportions reproduce the reported shifts and paired t-tests yield significant tRNA enrichment p-values for both mutants relative to WT.

## Related tools

- **Skipper** (Produces the reproducible_enriched_windows tables and feature_type_top annotations that are aggregated by transcript region and tested for WT/mutant shifts) — https://github.com/YeoLab/skipper
- **Snakemake 9.12.0** (Workflow engine used to run the Skipper pipeline that generates the enriched-window tables) — https://github.com/YeoLab/skipper

## Examples

```
snakemake -s Skipper.py --configfile example/Example_config.yaml --profile profiles/example_slurm
```

## Evaluation signals

- Aggregated feature_type_top proportions reproduce the reported shifts: CDS rises from 33% (WT) to 45% (mutants) and deep intronic binding falls from 20% to 7%
- Overall WT-vs-mutant distribution difference is significant (p < 2.2e−16)
- Paired t-tests across matched WT/mutant replicates yield significant tRNA enrichment p-values for both R62W and R101H relative to WT
- Reproducible enriched windows respect Skipper's constraint of not crossing intron/exon boundaries (windows were tiled on the filtered, one-best-transcript-per-gene GFF)
- Output figures (figure7e_feature_proportions.pdf, figure7f_trna_enrichment.pdf) render with significance annotations matching the paired-test results

## Limitations

- eCLIP data alone do not answer whether observed binding changes arise from differences in complex formation, localization, binding preference, or another mechanism
- The RPS19 follow-up expressed variant ORFs in HEK293T cells, which may not reflect the true disease state
- Skipper requires a matched input sample; datasets lacking these controls cannot be analyzed, so no window tables exist to quantify

## Evidence
