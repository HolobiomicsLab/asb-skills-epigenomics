---
name: chip-seq-peak-calling-for-loop-anchors
description: Use when processing ChIA-PET data where chromatin loops must be called
  between ChIP-Seq-like peak anchors, e.g. after bwa alignment and removeDup, before
  bedpe2Interaction/MICC loop calling.
when_to_use_negative:
- Input is raw paired-end FASTQ — linker trimming (trimLinker), bwa alignment and
  removeDup must precede tagAlign/peak calling
- Data are not from a peak-anchored assay (e.g. Hi-C contact maps) — ChIA-PET2 loop
  calling assumes anchors sit at ChIP-Seq-like peaks
- The bedpe already contains interactions restricted to known peak anchors and no
  new peak calling is needed
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: ChIA-PET2
  role: Provides buildTagAlign to convert the deduplicated bedpe to tagAlign and bedpe2Interaction
    (pairToBed -type both) to detect interactions between peak anchors
  repo: https://github.com/GuipengLi/ChIA-PET2
- name: MACS2
  role: Calls anchor peaks from the tagAlign file with a q-value cutoff of 0.05
  repo: https://github.com/macs3-project/MACS
- name: bedtools
  role: Underlies the pairToBed -type both intersection used by bedpe2Interaction
    to keep PETs whose both ends fall in MACS2 peaks
  repo: https://github.com/arq5x/bedtools2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkw809
    title: 'ChIA-PET2: a versatile and flexible pipeline for ChIA-PET data analysis'
  source_content_hash: sha256:356c1afeabae9c4f87714a981c3f806ad7c06a3ebff97422a4ee5111fdf64175
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
  - R
  - bedtools
  - bwa
  - samtools
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# chip-seq-peak-calling-for-loop-anchors

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts deduplicated ChIA-PET bedpe files to tagAlign format with buildTagAlign and calls anchor peaks with MACS2 (q-value 0.05) so that chromatin interactions can be detected between peak-anchored PET pairs, as implemented in the ChIA-PET2 pipeline.

## When to use

Use when processing ChIA-PET data where chromatin loops must be called between ChIP-Seq-like peak anchors, e.g. after bwa alignment and removeDup, before bedpe2Interaction/MICC loop calling.

## When NOT to use

- Input is raw paired-end FASTQ — linker trimming (trimLinker), bwa alignment and removeDup must precede tagAlign/peak calling
- Data are not from a peak-anchored assay (e.g. Hi-C contact maps) — ChIA-PET2 loop calling assumes anchors sit at ChIP-Seq-like peaks
- The bedpe already contains interactions restricted to known peak anchors and no new peak calling is needed

## Inputs

- Deduplicated bedpe file of PETs (output of buildBedpe + removeDup)
- Reference genome-aligned paired-end tags (from bwa SAM files)
- Optional ChIP-seq tracks (CTCF/RAD21/SMC3) for anchor validation

## Outputs

- tagAlign file generated from the bedpe (MACS2 input format)
- MACS2 anchor peak calls at q-value 0.05
- Peak-anchored chromatin interactions from bedpe2Interaction (pairToBed -type both)

## How to apply

After PCR duplicate removal, convert the deduplicated bedpe to a tagAlign file using ChIA-PET2's buildTagAlpha (buildTagAlign), which is explicitly designed as MACS2 input. Run MACS2 callpeak on the tagAlign with a q-value cutoff of 0.05 to define anchor peaks; ChIA-PET2's loop-calling strategy assumes loop anchors coincide with ChIP-Seq-like peaks. Intersect the deduplicated bedpe PETs with the called peaks using bedpe2Interaction (bedtools pairToBed -type both), requiring both PET ends to fall in peaks. For the GM12878 CTCF dataset this anchors the subsequent MICC2.R significance estimation (PET count cutoff 2), enabling detection of short-range (<8 kb) loops including the ~1.6 kb MYC loop.

## Related tools

- **ChIA-PET2** (Provides buildTagAlign to convert the deduplicated bedpe to tagAlign and bedpe2Interaction (pairToBed -type both) to detect interactions between peak anchors) — https://github.com/GuipengLi/ChIA-PET2
- **MACS2** (Calls anchor peaks from the tagAlign file with a q-value cutoff of 0.05) — https://github.com/macs3-project/MACS
- **bedtools** (Underlies the pairToBed -type both intersection used by bedpe2Interaction to keep PETs whose both ends fall in MACS2 peaks) — https://github.com/arq5x/bedtools2

## Examples

```
macs2 callpeak -t CTCF.tagAlign -f BED -n CTCF_anchors -q 0.05
```

## Evaluation signals

- buildTagAlign output exists and is in BED-like tagAlign format accepted by MACS2
- MACS2 produces peak calls restricted to q-value 0.05 and the number of anchors is consistent with the library's PET counts
- bedpe2Interaction output contains only PETs whose both ends overlap called peaks (pairToBed -type both)
- Anchor peaks overlap CTCF/RAD21/SMC3 occupancy at validated loci, e.g. anchors of the ~1.6 kb MYC loop supported by 30 PETs
- Downstream MICC2.R runs successfully on the peak-anchored interactions and its output retains the expected 13-column format

## Limitations

- Loop calling depends on ChIP-Seq-like peak calling sensitivity — loops whose anchors are not called as peaks will be missed
- The strategy assumes anchors of most chromatin loops are located at ChIP-Seq-like peaks; deviations from this assumption bias anchor definition
- Users must choose thresholds (e.g. MACS2 q-value, PET cutoffs) to balance sensitivity and specificity of downstream loop calling

## Evidence
