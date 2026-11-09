---
name: macs2-chip-seq-peak-calling
description: Use when processing deduplicated ChIA-PET bedpe/PET data through the
  ChIA-PET2 pipeline and ChIP-seq-like anchor peaks must be defined before interaction
  (loop) calling.
when_to_use_negative:
- Input is still raw ChIA-PET FASTQ or unfiltered alignments — linker trimming, read
  alignment and PCR duplicate removal (buildBedpe/removeDup, -Q 30) must precede peak
  calling.
- The analysis goal is not to define anchors for ChIA-PET loop calling but to call
  peaks from an ordinary ChIP-seq library; while MACS2 itself applies, the ChIA-PET2
  tagAlign-anchoring context does not.
- Loop calling is desired on data whose anchors are not expected at ChIP-seq-like
  peaks, violating ChIA-PET2's stated peak-based loop-calling assumption.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: MACS2
  role: Calls ChIP-seq-like anchor peaks from the tagAlign file built from deduplicated
    ChIA-PET bedpe, with ChIA-PET2 defaults -q 0.05
  repo: https://github.com/macs3-project/MACS
- name: ChIA-PET2
  role: Pipeline wrapper whose buildTagAlign module converts the deduplicated bedpe
    into the MACS2 input tagAlign file
  repo: https://github.com/GuipengLi/ChIA-PET2
- name: bedtools
  role: Used downstream to intersect loop-anchor coordinates with independent CTCF,
    RAD21 and SMC3 ChIP-seq peak tracks for anchor-occupancy validation
  repo: https://github.com/arq5x/bedtools2
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkw809
    title: 'ChIA-PET2: a versatile and flexible pipeline for ChIA-PET data analysis'
  source_content_hash: sha256:0eac7359ceba978014fa9bc22818e6269100c132b8ddcd11b39747d493dac79b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/GuipengLi/ChIA-PET2
  repo_url: https://github.com/macs3-project/MACS
  tools:
  - ChIA-PET2
  - MACS2
  - R
  - bedtools
  - bwa
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# macs2-chip-seq-peak-calling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Call ChIP-seq-like anchor peaks with MACS2 from ChIA-PET tagAlign data as the anchor-detection step of the ChIA-PET2 chromatin loop calling pipeline, using MACS2 default parameters (-q 0.05).

## When to use

Use when processing deduplicated ChIA-PET bedpe/PET data through the ChIA-PET2 pipeline and ChIP-seq-like anchor peaks must be defined before interaction (loop) calling.

## When NOT to use

- Input is still raw ChIA-PET FASTQ or unfiltered alignments — linker trimming, read alignment and PCR duplicate removal (buildBedpe/removeDup, -Q 30) must precede peak calling.
- The analysis goal is not to define anchors for ChIA-PET loop calling but to call peaks from an ordinary ChIP-seq library; while MACS2 itself applies, the ChIA-PET2 tagAlign-anchoring context does not.
- Loop calling is desired on data whose anchors are not expected at ChIP-seq-like peaks, violating ChIA-PET2's stated peak-based loop-calling assumption.

## Inputs

- Deduplicated ChIA-PET bedpe file (output of buildBedpe/removeDup from aligned reads with MAPQ >= 30)
- Reference genome (for MACS2 effective genome size setting)

## Outputs

- MACS2 peak calls (narrowPeak BED) representing ChIP-seq-like loop anchors
- Peak-interval file consumed by bedpe2Interaction to pair PETs with anchors

## How to apply

Within the ChIA-PET2 workflow, first convert the deduplicated bedpe file to a tagAlign file with ChIA-PET2's buildTagAlign, which is the designated MACS2 input format. Then run MACS2 callpeak on this tagAlign file using ChIA-PET2's default parameters (-M "-q 0.05") to obtain ChIA-Seq-like anchor peak intervals. The rationale is that ChIA-PET2 calls chromatin loops based on ChIP-Seq-like peaks, assuming loop anchors coincide with such peaks; candidate interactions are later formed by assigning PETs to peak pairs (bedpe2Interaction) with a 100 bp slop (-S) and 500 bp anchor extension (-E). Validation can be done by overlapping loop anchors with independent CTCF/RAD21/SMC3 ChIP-seq peaks using bedtools.

## Related tools

- **MACS2** (Calls ChIP-seq-like anchor peaks from the tagAlign file built from deduplicated ChIA-PET bedpe, with ChIA-PET2 defaults -q 0.05) — https://github.com/macs3-project/MACS
- **ChIA-PET2** (Pipeline wrapper whose buildTagAlign module converts the deduplicated bedpe into the MACS2 input tagAlign file) — https://github.com/GuipengLi/ChIA-PET2
- **bedtools** (Used downstream to intersect loop-anchor coordinates with independent CTCF, RAD21 and SMC3 ChIP-seq peak tracks for anchor-occupancy validation) — https://github.com/arq5x/bedtools2

## Examples

```
buildTagAlign dedup.bedpe anchors.tagAlign && macs2 callpeak -t anchors.tagAlign -q 0.05
```

## Evaluation signals

- MACS2 runs with ChIA-PET2 default significance threshold (-q 0.05) and emits narrowPeak-format anchor intervals.
- Peak input is the tagAlign produced by buildTagAlign from the deduplicated bedpe (PETs built at default MAPQ 30), not raw alignments.
- Downstream bedpe2Interaction accepts the peak file and applies 100 bp slop (-S) and 500 bp anchor extension (-E) without coordinate errors.
- Loop-anchor overlap with independent ENCODE CTCF/RAD21/SMC3 ChIP-seq peaks reproduces the reported occupancy levels (CTCF 99.9%, RAD21 98.8%, SMC3 99.7%) within a reasonable margin.
- FDR <= 0.05 intra-chromosomal loop counts (e.g., ~25,155 on GM12878 CTCF with PET cutoff 2) are reproducible from the resulting peaks.

## Limitations

- ChIA-PET2 loop calling depends on ChIP-Seq-like peak calling sensitivity — loops are only called at anchors overlapping detected peaks.
- The peak-based strategy rests on the assumption that anchors of most chromatin loops lie at ChIP-seq-like peaks; deviations from this assumption will bias loop detection.
- The article states no software versions for required dependencies (bwa, samtools, bedtools, MACS2, g++), so reproducibility of exact peak sets may vary across versions.

## Evidence

- [readme] MACS purpose as stated in its repository README.: [evidence span withheld — anchor status: external_doc]
