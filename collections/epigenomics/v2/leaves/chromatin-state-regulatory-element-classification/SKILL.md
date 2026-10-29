---
name: chromatin-state-regulatory-element-classification
description: Use when RNAPII ChIA-PET loop calls (PET count ≥3 with peak support)
  need to be annotated as enhancer-promoter or promoter-promoter interactions and
  exported as separate per-category loop files for visualization in BASIC Browser
  tracks.
when_to_use_negative:
- Input loops are not yet called or lack peak support / PET count ≥3 filtering — classification
  anchors are the ChIA-PIPE loop-calling outputs, not raw PETs.
- The assay is not RNAPII (or analogous) ChIA-PET data where enhancer/promoter assignment
  of anchors is biologically meaningful — e.g., CTCF ChIA-PET loops should be interpreted
  differently.
- Input is already a classified/annotated loop file — re-running intersection would
  duplicate annotation without new information.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0605
tools:
- name: ChIA-PIPE
  role: Upstream pipeline producing the loop calls (PET count ≥3, peak support, unique
    loop IDs) that are annotated; its enhancer-promoter loop annotation step this
    skill implements, using a custom Python script for per-loop classification
  repo: https://github.com/TheJacksonLaboratory/ChIA-PIPE
- name: bedtools
  role: Performs coordinate intersection of each loop anchor BED against enhancer
    and promoter BED files on the same chromosome to flag anchor overlap
- name: CPU (ChIA-PET Utilities)
  role: Modularized executables integrated by ChIA-PIPE for ChIA-PET processing up
    to interaction calls that feed this annotation
  repo: https://github.com/cheehongsg/CPU
- name: BASIC Browser
  role: 'Downstream visualization target: per-category subset loop files enable each
    loop class (E-P, P-P, neither) to be displayed in a separate track'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1126/sciadv.aay2078
    title: 'ChIA-PIPE: A fully automated pipeline for comprehensive ChIA-PET data
      analysis and visualization'
  source_content_hash: sha256:c723bf7db514f11680c68d7e543ff9e7c4791403f8396182d43f54017a190862
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TheJacksonLaboratory/ChIA-PIPE
  tools:
  - BASIC Browser
  - CPU (ChIA-PET Utilities)
  - ChIA-PIPE
  - bedtools
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# chromatin-state-regulatory-element-classification

## Summary

Classifies called ChIA-PET loops (PET count ≥3, peak-supported, each with a unique loop ID) into enhancer-promoter (E-P), promoter-promoter (P-P), or neither categories by intersecting loop anchor coordinates against enhancer and promoter BED annotations, producing per-category subset loop files for separate browser track display.

## When to use

Use when RNAPII ChIA-PET loop calls (PET count ≥3 with peak support) need to be annotated as enhancer-promoter or promoter-promoter interactions and exported as separate per-category loop files for visualization in BASIC Browser tracks.

## When NOT to use

- Input loops are not yet called or lack peak support / PET count ≥3 filtering — classification anchors are the ChIA-PIPE loop-calling outputs, not raw PETs.
- The assay is not RNAPII (or analogous) ChIA-PET data where enhancer/promoter assignment of anchors is biologically meaningful — e.g., CTCF ChIA-PET loops should be interpreted differently.
- Input is already a classified/annotated loop file — re-running intersection would duplicate annotation without new information.

## Inputs

- ChIA-PIPE loop calling output: left and right loop anchor BED files (loops with PET count ≥3 and peak support, unique loop ID per loop)
- Enhancer coordinate BED file
- Promoter coordinate BED file

## Outputs

- ep_loops.bed — loops classified enhancer-promoter, original loop BED columns plus class label column
- pp_loops.bed — loops classified promoter-promoter, original loop BED columns plus class label column
- neither_loops.bed — loops with neither classification, original loop BED columns plus class label column

## How to apply

Load the loop left/right anchor BED files emitted by ChIA-PIPE loop calling, where loops have PET count ≥3 and peak support and each loop carries a unique loop ID. Use bedtools intersect between each anchor BED and the enhancer BED, and between each anchor BED and the promoter BED, requiring coordinate overlap on the same chromosome to flag which anchors overlap an enhancer or a promoter. For each loop ID, classify it as E-P if exactly one anchor overlaps an enhancer and the other overlaps a promoter, P-P if both anchors overlap promoters, and neither otherwise (in the article this is done by a custom Python script). Write classified loops into three per-category subset files (ep_loops.bed, pp_loops.bed, neither_loops.bed) preserving original loop BED columns plus a class label column, so each loop category can be rendered in a separate BASIC Browser track. This realizes ChIA-PIPE's enhancer-promoter loop annotation step, one of its downstream analyses after linker filtering, read mapping, peak calling, and loop calling.

## Related tools

- **ChIA-PIPE** (Upstream pipeline producing the loop calls (PET count ≥3, peak support, unique loop IDs) that are annotated; its enhancer-promoter loop annotation step this skill implements, using a custom Python script for per-loop classification) — https://github.com/TheJacksonLaboratory/ChIA-PIPE
- **bedtools** (Performs coordinate intersection of each loop anchor BED against enhancer and promoter BED files on the same chromosome to flag anchor overlap)
- **CPU (ChIA-PET Utilities)** (Modularized executables integrated by ChIA-PIPE for ChIA-PET processing up to interaction calls that feed this annotation) — https://github.com/cheehongsg/CPU
- **BASIC Browser** (Downstream visualization target: per-category subset loop files enable each loop class (E-P, P-P, neither) to be displayed in a separate track)

## Examples

```
bedtools intersect -a loop_anchor_R1.bed -b enhancers.bed > r1_enh.txt; bedtools intersect -a loop_anchor_R2.bed -b promoters.bed > r2_prom.txt
```

## Evaluation signals

- Set invariant: the union of ep_loops.bed, pp_loops.bed, and neither_loops.bed covers every input loop ID exactly once — no loops lost or duplicated.
- Each emitted subset file contains a non-empty class label column appended to the original loop BED columns (QC check on the annotation output).
- Sanity check that classification logic holds on spot-checked loops: exactly one enhancer-overlapping + one promoter-overlapping anchor ⇒ E-P; two promoter-overlapping anchors ⇒ P-P.
- bedtools intersect results agree with chromosome-constrained coordinate overlap expectations (no cross-chromosomal anchor flagging).
- Each subset file is loadable as a separate BASIC Browser track and loop counts across the three files sum to the input loop count.

## Limitations

- The custom Python script used for E-P/P-P loop classification is not named, versioned, or hosted at a stated location in the article — reproducibility of the exact classification code is not guaranteed.
- Loops whose anchors overlap neither an enhancer nor a promoter, or with ambiguous anchor combinations (e.g., both anchors enhancer-overlapping), fall into the 'neither' bucket and receive no regulatory pairing.
- Reference genome version used for alignment is not specified in this section, so enhancer/promoter BED annotations must be built against a genome build consistent with the loop coordinates.

## Evidence
