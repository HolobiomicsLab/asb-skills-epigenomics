---
name: chromatin-loop-transitivity-and-redundancy-analysis
description: Use when a Hi-C contact map implies a transitive or redundant chromatin
  loop between loci and a ChIA-PET2 CTCF bedpe/contact matrix is available to confirm
  or refute the direct interaction.
when_to_use_negative:
- When starting from raw ChIA-PET FASTQ — this skill operates downstream of ChIA-PET2's
  trimLinker, buildBedpe, removeDup and loop-calling steps on deduplicated bedpe output
- When the goal is only significant loop calling (MICC/ChiaSig/Mango comparison) without
  cross-referencing an external Hi-C contact map
- The paper provides only a few illustrative examples of transitivity/redundancy/robustness,
  so this is not a validated genome-wide transitivity classifier
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0602
tools:
- name: ChIA-PET2
  role: provides bedpe2Matrix to convert the deduplicated CTCF bedpe into a Hi-C style
    10 kb contact matrix and bedpe2Interaction/MICC2.R for significant loop calling
  repo: https://github.com/GuipengLi/ChIA-PET2
- name: MICC
  role: Bayesian mixture model that filters the CTCF contact matrix to significant
    loops via -log10(1-PostProbability) and FDR
- name: bedtools
  role: bed/bedpe file manipulation used to prepare and restrict loop intervals to
    the loci of interest
  repo: https://github.com/arq5x/bedtools2
- name: HiCPlotter
  role: renders the side-by-side overlay of the triplet sparse CTCF contact matrix
    onto the matched Hi-C contact map
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkw809
    title: 'ChIA-PET2: a versatile and flexible pipeline for ChIA-PET data analysis'
  source_content_hash: sha256:80835ee5b976946eac2612bb27b340611ea4d8551eb980bd8ad6febd643fbc28
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
  - HiCPlotter
  - MICC
  - R
  - bedtools
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# chromatin-loop-transitivity-and-redundancy-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Cross-checks ChIA-PET2-derived CTCF chromatin loops against a matched Hi-C contact map to test whether Hi-C-implied transitive loops (e.g., L1–L3a and L1–L4a implying L3a–L4a) exist as direct ChIA-PET loops, and to tabulate redundancy cases where one Hi-C loop resolves into multiple CTCF-mediated loops.

## When to use

Use when a Hi-C contact map implies a transitive or redundant chromatin loop between loci and a ChIA-PET2 CTCF bedpe/contact matrix is available to confirm or refute the direct interaction.

## When NOT to use

- When starting from raw ChIA-PET FASTQ — this skill operates downstream of ChIA-PET2's trimLinker, buildBedpe, removeDup and loop-calling steps on deduplicated bedpe output
- When the goal is only significant loop calling (MICC/ChiaSig/Mango comparison) without cross-referencing an external Hi-C contact map
- The paper provides only a few illustrative examples of transitivity/redundancy/robustness, so this is not a validated genome-wide transitivity classifier

## Inputs

- deduplicated CTCF ChIA-PET bedpe file (in.rmDup.bedpe) produced by ChIA-PET2 removeDup
- chromosome sizes file (chrom_hg19.sizes) in hg19
- matching GM12878 Hi-C contact map at 10 kb resolution
- MICC2.R significant-loop output with -log10(1-PostProbability) and FDR per loop

## Outputs

- Hi-C style CTCF contact matrix in triplet sparse format (HiCPlotter-compatible)
- side-by-side HiCPlotter overlay figures of CTCF contact matrix vs Hi-C contact map at test loci
- table of transitivity verdicts (transitive loop absent/present) and redundancy cases (one Hi-C loop = multiple CTCF loops)

## How to apply

Convert the deduplicated CTCF ChIA-PET bedpe (in.rmDup.bedpe) into a Hi-C style contact matrix using ChIA-PET2's bedpe2Matrix with --binsize 10000 and the chromosome sizes file (chrom_hg19.sizes), producing a triplet sparse format matrix. Import the matching GM12878 Hi-C contact map at the same 10 kb resolution for the regions covered by the CTCF loops. Restrict the CTCF contact matrix to significant loops using MICC2.R output, which estimates -log10(1-PostProbability) and FDR per loop via a Bayesian mixture model. Overlay the CTCF contact matrix with the Hi-C contact map using HiCPlotter (the triplet sparse matrix is HiCPlotter-compatible) at loci where Hi-C implies a transitive contact but the CTCF loop is absent. Finally, tabulate redundancy cases in which a single Hi-C loop corresponds to multiple CTCF-mediated loops and render the side-by-side comparison figure; interpret absence of a loop pair (e.g., L3a–L4a) as evidence the transitive loop does not exist, resolving the structure instead as an alternative loop (e.g., L3b–L4a).

## Related tools

- **ChIA-PET2** (provides bedpe2Matrix to convert the deduplicated CTCF bedpe into a Hi-C style 10 kb contact matrix and bedpe2Interaction/MICC2.R for significant loop calling) — https://github.com/GuipengLi/ChIA-PET2
- **MICC** (Bayesian mixture model that filters the CTCF contact matrix to significant loops via -log10(1-PostProbability) and FDR)
- **bedtools** (bed/bedpe file manipulation used to prepare and restrict loop intervals to the loci of interest) — https://github.com/arq5x/bedtools2
- **HiCPlotter** (renders the side-by-side overlay of the triplet sparse CTCF contact matrix onto the matched Hi-C contact map)

## Examples

```
bedpe2Matrix --bedpe in.rmDup.bedpe --binsize 10000 --genome chrom_hg19.sizes --output ctcf_matrix
```

## Evaluation signals

- bedpe2Matrix with --binsize 10000 and chrom_hg19.sizes yields a triplet sparse format matrix that loads directly into HiCPlotter without reformatting
- MICC2.R output carries per-loop -log10(1-PostProbability) and FDR values from the Bayesian mixture model
- At the test locus, the CTCF contact matrix shows no significant loop between L3a and L4a (transitive loop absent) while an alternative pair (e.g., L3b–L4a) shows a loop
- Redundancy table lists Hi-C loops that map to more than one CTCF-mediated loop in the overlay figure
- Both contact maps compared at identical 10 kb resolution over the same genomic regions

## Limitations

- ChIA-PET2 calls chromatin loops based on ChIP-Seq-like peaks, so loop detection depends on ChIP-Seq-like peak calling sensitivity at anchors
- The paper states that only a few examples of transitivity, redundancy and robustness of chromatin loops were provided, so the logic is demonstrated, not benchmarked genome-wide
- Users must choose different thresholds in the ChIA-PET2 pipeline to balance sensitivity and specificity of chromatin loop calling; conclusions can shift with the FDR cutoff

## Evidence
