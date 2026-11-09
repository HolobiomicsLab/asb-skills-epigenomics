---
name: fdr-thresholded-loop-count-reproducibility-checking
description: Use when a ChIA-PET loop-calling result (MICC FDR-filtered interactions)
  must be reproduced against a published interaction count and validated by anchor
  ChIP-seq co-occupancy.
when_to_use_negative:
- The loop calls come from a different pipeline whose statistical model differs (CPT,
  ChiaSig, Mango), so counts at the same nominal FDR are not comparable.
- The dataset is not a ChIA-PET library processed with a PET count cut-off filter,
  so MICC's default CUTOFFPET=2 significance estimation does not apply.
- The goal is differential loop comparison between conditions rather than reproduction
  of a single reported interaction count.
edam_topics:
- http://edamontology.org/topic_0654
tools:
- name: ChIA-PET2
  role: end-to-end pipeline producing bedpe candidate interactions and the MICC-filtered
    loop set being counted
  repo: https://github.com/GuipengLi/ChIA-PET2
- name: MICC
  role: Bayesian mixture model assigning -log10(1-PostProbability) and FDR to each
    loop; invoked via MICC2.R with default CUTOFFPET=2
- name: MACS2
  role: calls ChIP-seq-like anchor peaks from buildTagAlign output with -q 0.05, defining
    the peaks pairs PETs are assigned to
  repo: https://github.com/macs3-project/MACS
- name: bedtools
  role: overlaps loop-anchor coordinates with ENCODE CTCF/RAD21/SMC3 peak tracks and
    manipulates bed/bedpe files (v2.24.0+)
  repo: https://github.com/arq5x/bedtools2
- name: R
  role: runtime for MICC2.R loop significance estimation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkw809
    title: 'ChIA-PET2: a versatile and flexible pipeline for ChIA-PET data analysis'
  source_content_hash: sha256:ab452b51b46a25b44aa05409112f8dd9e744a6bbadf28a77600c876ef6433dea
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/GuipengLi/ChIA-PET2
  repo_url: https://github.com/GuipengLi/ChIA-PET2
  tools:
  - ChIA-PET2
  - MACS2
  - MICC
  - R
  - bedtools
  - bwa
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# fdr-thresholded-loop-count-reproducibility-checking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Verifies that a ChIA-PET2 loop-calling run reproduces a published count of high-confidence intra-chromosomal chromatin loops (e.g. ~25,155 at FDR ≤ 0.05 with PET count cut-off 2 on the GM12878 CTCF dataset), and cross-checks loop anchors against independent CTCF/RAD21/SMC3 ChIP-seq peak tracks.

## When to use

Use when a ChIA-PET loop-calling result (MICC FDR-filtered interactions) must be reproduced against a published interaction count and validated by anchor ChIP-seq co-occupancy.

## When NOT to use

- The loop calls come from a different pipeline whose statistical model differs (CPT, ChiaSig, Mango), so counts at the same nominal FDR are not comparable.
- The dataset is not a ChIA-PET library processed with a PET count cut-off filter, so MICC's default CUTOFFPET=2 significance estimation does not apply.
- The goal is differential loop comparison between conditions rather than reproduction of a single reported interaction count.

## Inputs

- Duplicate-removed, aligned ChIA-PET paired-end tag data (SAM/BAM) from the GM12878 CTCF library (GSM970213 / GSE72816)
- bedpe files of candidate interactions (interactions.intra.bedpe, interactions.inter.bedpe) with PET counts
- out.interactions.MICC file with -log10(1-PostProbability) and FDR per loop
- Independent ENCODE CTCF, RAD21 and SMC3 ChIP-seq peak tracks (BED)

## Outputs

- Count of intra-chromosomal loops with FDR <= 0.05 (expected ~25,155)
- Fraction of loop anchors overlapping CTCF ChIP-seq peaks (expected 99.9%)
- Fraction of anchors co-occupied by RAD21 (expected 98.8%) and SMC3 (expected 99.7%)
- MICC per-loop significance table (FDR and posterior probability)

## How to apply

Run the ChIA-PET2 pipeline through MICC: build PET records from duplicate-removed alignments with buildBedpe/removeDup at MAPQ 30 (-Q 30), call ChIP-seq-like anchor peaks with MACS2 at -q 0.05, assign PETs to peak pairs with bedpe2Interaction using slop 100 bp (-S) and 500 bp anchor extension (-E), then apply the PET count cut-off of 2 (-C) and run MICC via MICC2.R to estimate -log10(1-PostProbability) and FDR for each loop under the Bayesian mixture model. From out.interactions.MICC, count intra-chromosomal loops with FDR <= 0.05 and compare to the reported 25,155 high-confidence interactions. Independently overlap loop-anchor coordinates with ENCODE CTCF, RAD21 and SMC3 ChIP-seq peak tracks using bedtools to compute the fraction of CTCF-bound anchors and RAD21/SMC3 co-occupied anchors. The check is accepted when both the loop count and the anchor occupancy levels (CTCF 99.9%, RAD21 98.8%, SMC3 99.7%) fall within a reasonable margin of the reported values.

## Related tools

- **ChIA-PET2** (end-to-end pipeline producing bedpe candidate interactions and the MICC-filtered loop set being counted) — https://github.com/GuipengLi/ChIA-PET2
- **MICC** (Bayesian mixture model assigning -log10(1-PostProbability) and FDR to each loop; invoked via MICC2.R with default CUTOFFPET=2)
- **MACS2** (calls ChIP-seq-like anchor peaks from buildTagAlign output with -q 0.05, defining the peaks pairs PETs are assigned to) — https://github.com/macs3-project/MACS
- **bedtools** (overlaps loop-anchor coordinates with ENCODE CTCF/RAD21/SMC3 peak tracks and manipulates bed/bedpe files (v2.24.0+)) — https://github.com/arq5x/bedtools2
- **R** (runtime for MICC2.R loop significance estimation)

## Examples

```
Rscript MICC2.R interactions.intra.bedpe interactions.inter.bedpe miccOUT CUTOFFPET
```

## Evaluation signals

- Intra-chromosomal loop count at FDR <= 0.05 in out.interactions.MICC reproduces the reported 25,155 high-confidence interactions within a reasonable margin.
- Anchor occupancy matches reported CTCF 99.9%, RAD21 98.8% and SMC3 99.7% levels.
- PET count cut-off of 2 (-C) was applied before MICC significance estimation, matching the default CUTOFFPET=2.
- Every reported loop carries both an FDR and a -log10(1-PostProbability) value from the Bayesian mixture model.
- The candidate-interaction step used the default slop of 100 bp (-S) and 500 bp anchor extension (-E), consistent with the paper's parameters.

## Limitations

- ChIA-PET2 calls chromatin loops based on ChIP-Seq-like peaks, so loop calling depends on the sensitivity of the MACS2 peak-calling step.
- No software version of ChIA-PET2 or its dependencies (bwa, samtools, bedtools, MACS2, g++) is stated, complicating exact reproduction.
- Users choose different thresholds to balance sensitivity and specificity of loop calling, so FDR-thresholded counts shift with parameter choices.

## Evidence
