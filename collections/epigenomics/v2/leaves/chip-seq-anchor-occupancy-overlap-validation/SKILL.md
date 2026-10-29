---
name: chip-seq-anchor-occupancy-overlap-validation
description: Use when a set of chromatin loop calls (e.g., bedpe from MICC/ChIA-PET2)
  needs orthogonal validation against independent CTCF/cohesin ChIP-seq peaks in the
  same cell line (e.g., GM12878).
when_to_use_negative:
- No loop calls exist yet — anchors can only be overlapped after ChIA-PET2 (buildBedpe,
  bedpe2Interaction, MICC2.R) has produced out.interactions.MICC
- No matching-factor ChIP-seq peak tracks exist for the same or a closely related
  cell type — occupancy fractions would not be interpretable
- The loop caller is not peak/anchor-based, so the expectation that loop anchors fall
  at ChIP-seq peaks does not hold
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: bedtools
  role: 'genome arithmetic: intersect loop-anchor coordinates with CTCF/RAD21/SMC3
    ChIP-seq peak tracks to compute occupancy fractions'
  repo: https://github.com/arq5x/bedtools2
- name: ChIA-PET2
  role: upstream pipeline that generates the loop bedpe (out.interactions.MICC) whose
    anchors are validated
  repo: https://github.com/GuipengLi/ChIA-PET2
- name: MICC
  role: Bayesian mixture model producing FDR per loop; FDR <= 0.05 loops feed the
    overlap validation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkw809
    title: 'ChIA-PET2: a versatile and flexible pipeline for ChIA-PET data analysis'
  source_content_hash: sha256:2dae291e72b1f55bbc59798c3a1c10072ca58ccfe7239a479c370aa77b191784
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/GuipengLi/ChIA-PET2
  repo_url: https://github.com/arq5x/bedtools2
  tools:
  - ChIA-PET2
  - MICC
  - R
  - bedtools
  - bwa
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# chip-seq-anchor-occupancy-overlap-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates ChIA-PET loop calls by overlapping loop-anchor coordinates with independent ENCODE ChIP-seq peak tracks (CTCF, RAD21, SMC3) using bedtools, computing the fraction of protein-bound anchors and co-occupied anchors as a biological plausibility check.

## When to use

Use when a set of chromatin loop calls (e.g., bedpe from MICC/ChIA-PET2) needs orthogonal validation against independent CTCF/cohesin ChIP-seq peaks in the same cell line (e.g., GM12878).

## When NOT to use

- No loop calls exist yet — anchors can only be overlapped after ChIA-PET2 (buildBedpe, bedpe2Interaction, MICC2.R) has produced out.interactions.MICC
- No matching-factor ChIP-seq peak tracks exist for the same or a closely related cell type — occupancy fractions would not be interpretable
- The loop caller is not peak/anchor-based, so the expectation that loop anchors fall at ChIP-seq peaks does not hold

## Inputs

- FDR-filtered intra-chromosomal loop bedpe (out.interactions.MICC from ChIA-PET2/MICC2.R)
- ENCODE CTCF ChIP-seq peak track (BED) for GM12878
- ENCODE RAD21 ChIP-seq peak track (BED) for GM12878
- ENCODE SMC3 ChIP-seq peak track (BED) for GM12878

## Outputs

- Fraction (percentage) of loop anchors overlapping CTCF peaks
- Fraction of anchors overlapping RAD21 peaks
- Fraction of anchors overlapping SMC3 peaks
- Fraction of anchor pairs co-occupied by RAD21/SMC3

## How to apply

Take the FDR-filtered intra-chromosomal loops (FDR <= 0.05, PET count cut-off 2) from the out.interactions.MICC file produced by ChIA-PET2/MICC and extract the two anchor coordinate intervals per loop. Download independent ENCODE ChIP-seq peak tracks for CTCF, RAD21 and SMC3 in the matching cell type (GM12878; the paper retrieved ChIP-Seq data from the ENCODE data repository). Use bedtools intersect to compute the fraction of anchors overlapping each factor's peaks and the fraction of anchor pairs co-occupied by RAD21 and SMC3. Accept the loop set when anchor occupancy reproduces the reported enrichment levels — CTCF 99.9%, RAD21 98.8% and SMC3 99.7% — within a reasonable margin. The rationale is that ChIA-PET2 calls loops based on ChIP-Seq-like peaks, so anchors should be highly enriched for the immunoprecipitated factor and cohesin occupancy.

## Related tools

- **bedtools** (genome arithmetic: intersect loop-anchor coordinates with CTCF/RAD21/SMC3 ChIP-seq peak tracks to compute occupancy fractions) — https://github.com/arq5x/bedtools2
- **ChIA-PET2** (upstream pipeline that generates the loop bedpe (out.interactions.MICC) whose anchors are validated) — https://github.com/GuipengLi/ChIA-PET2
- **MICC** (Bayesian mixture model producing FDR per loop; FDR <= 0.05 loops feed the overlap validation)

## Examples

```
bedtools intersect -a loop_anchors.bed -b ENCODE_GM12878_CTCF_peaks.bed -u | wc -l
```

## Evaluation signals

- Fraction of CTCF-bound anchors matches the reported 99.9% within a reasonable margin
- Fraction of RAD21-occupied anchors matches the reported 98.8%
- Fraction of SMC3-occupied anchors matches the reported 99.7%
- Number of overlapped anchors per factor never exceeds the total number of loop anchors extracted from the bedpe
- Loop count feeding the validation reproduces the ~25,155 FDR <= 0.05 intra-chromosomal interactions on the GM12878 CTCF dataset

## Limitations

- ChIA-PET2 loop calling depends on ChIP-Seq-like peak calling sensitivity, so anchors are by construction biased toward ChIP-seq peak regions
- No versions stated for required dependencies (bwa, samtools, bedtools, MACS2), which may affect reproducibility of the overlap step
- Only a few examples of transitivity, redundancy and robustness of chromatin loops were provided in the paper

## Evidence

- [other] Overlap loop-anchor coordinates with independent ENCODE CTCF, RAD21 and SMC3 ChIP-seq peak tracks using bedtools: [evidence span withheld — anchor status: not_found]
- [readme] Collectively, the bedtools utilities are a swiss-army knife of tools: [evidence span withheld — anchor status: external_doc]
