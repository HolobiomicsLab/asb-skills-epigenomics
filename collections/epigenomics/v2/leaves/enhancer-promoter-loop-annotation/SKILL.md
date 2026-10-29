---
name: enhancer-promoter-loop-annotation
description: Use when RNAPII ChIA-PET (or HiChIP/PLAC-seq) loops have already been
  called with PET count >= 3 and peak support, and each loop must be labeled E-P,
  P-P, or neither using enhancer/promoter BED annotations.
when_to_use_negative:
- The underlying loops have not yet been called by ChIA-PIPE (no peak-supported, PET
  count >= 3 loop set with unique loop IDs exists).
- The experiment is CTCF ChIA-PET rather than RNAPII, so enhancer/promoter loop semantics
  do not apply to the anchor biology.
- The goal is CCD calling or allele-specific loop resolution rather than functional
  (E-P/P-P) categorization of loops.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: ChIA-PIPE
  role: Pipeline that calls the loops (PET count >= 3, peak support, unique loop IDs)
    and whose custom Python script performs the E-P / P-P / neither classification,
    producing subset loop files for per-category visualization
  repo: https://github.com/TheJacksonLaboratory/ChIA-PIPE
- name: bedtools
  role: Intersects each loop anchor BED file against enhancer and promoter BED files,
    requiring coordinate overlap on the same chromosome, to flag anchor annotations
- name: CPU (ChIA-PET Utilities)
  role: Modularized executables integrated by ChIA-PIPE for processing ChIA-PET data
    from raw reads to interactions, including the dedup step that yields unique PETs
    feeding loop calling
  repo: https://github.com/cheehongsg/CPU
- name: BASIC Browser
  role: Loop browser in which the per-category (E-P, P-P, neither) subset loop files
    are displayed as separate tracks
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1126/sciadv.aay2078
    title: 'ChIA-PIPE: A fully automated pipeline for comprehensive ChIA-PET data
      analysis and visualization'
  source_content_hash: sha256:ef6646e5bcd750c79b9496b2cd48fc600906caa1a16d36f18f971eabb7866ac3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TheJacksonLaboratory/ChIA-PIPE
  tools:
  - BASIC Browser
  - CPU (ChIA-PET Utilities)
  - ChIA-PIPE
  - bedtools
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# enhancer-promoter-loop-annotation

## Summary

Classify called RNAPII ChIA-PET loops (PET count >= 3 with peak support, each with a unique loop ID) into enhancer-promoter (E-P), promoter-promoter (P-P), or neither categories by intersecting anchor coordinates with enhancer and promoter BED files, and emit per-category subset loop files for separate BASIC Browser tracks.

## When to use

Use when RNAPII ChIA-PET (or HiChIP/PLAC-seq) loops have already been called with PET count >= 3 and peak support, and each loop must be labeled E-P, P-P, or neither using enhancer/promoter BED annotations.

## When NOT to use

- The underlying loops have not yet been called by ChIA-PIPE (no peak-supported, PET count >= 3 loop set with unique loop IDs exists).
- The experiment is CTCF ChIA-PET rather than RNAPII, so enhancer/promoter loop semantics do not apply to the anchor biology.
- The goal is CCD calling or allele-specific loop resolution rather than functional (E-P/P-P) categorization of loops.

## Inputs

- ChIA-PIPE loop calling output: left and right loop anchor BED files (loops with PET count >= 3 and peak support, unique loop IDs)
- Enhancer BED file (genomic coordinates of enhancers)
- Promoter BED file (genomic coordinates of promoters)

## Outputs

- ep_loops.bed — E-P loops, original loop BED columns plus a class label column
- pp_loops.bed — P-P loops, original loop BED columns plus a class label column
- neither_loops.bed — loops classified as neither E-P nor P-P, with class label column

## How to apply

Load the loop left/right anchor BED files produced by ChIA-PIPE loop calling, where each loop carries a unique loop ID. Run bedtools intersect between each anchor BED and the enhancer BED, and between each anchor BED and the promoter BED, requiring coordinate overlap on the same chromosome to flag which anchors overlap an enhancer or a promoter. For each loop ID, classify it as E-P if exactly one anchor overlaps an enhancer and the other overlaps a promoter, P-P if both anchors overlap promoters, and neither otherwise (ChIA-PIPE uses a custom Python script for this determination). Write the classified loops into three per-category subset files (e.g., ep_loops.bed, pp_loops.bed, neither_loops.bed) preserving the original loop BED columns plus a class label column, so each loop category can be displayed as a separate BASIC Browser track.

## Related tools

- **ChIA-PIPE** (Pipeline that calls the loops (PET count >= 3, peak support, unique loop IDs) and whose custom Python script performs the E-P / P-P / neither classification, producing subset loop files for per-category visualization) — https://github.com/TheJacksonLaboratory/ChIA-PIPE
- **bedtools** (Intersects each loop anchor BED file against enhancer and promoter BED files, requiring coordinate overlap on the same chromosome, to flag anchor annotations)
- **CPU (ChIA-PET Utilities)** (Modularized executables integrated by ChIA-PIPE for processing ChIA-PET data from raw reads to interactions, including the dedup step that yields unique PETs feeding loop calling) — https://github.com/cheehongsg/CPU
- **BASIC Browser** (Loop browser in which the per-category (E-P, P-P, neither) subset loop files are displayed as separate tracks)

## Examples

```
bash chia_pipe-master/0.chia_pipe_shell.sh -c chia_pipe-master/example_config_file.sh
```

## Evaluation signals

- The union of the three per-category files (ep_loops.bed, pp_loops.bed, neither_loops.bed) covers every input loop ID exactly once — no loops lost or duplicated.
- Each emitted subset file contains a non-empty class label column as a QC check on the annotation output.
- Every loop labeled E-P has exactly one anchor overlapping the enhancer BED and the other overlapping the promoter BED on the same chromosome (verifiable by re-running bedtools intersect).
- Every loop labeled P-P has both anchors overlapping promoter BED intervals.
- Input loop count equals the sum of rows across the three subset files, and all input loops had PET count >= 3 with peak support before annotation.

## Limitations

- Loops that do not match either the E-P or P-P criteria (e.g., enhancer-enhancer or unannotated anchors) fall into the 'neither' category and receive no functional label.
- The custom Python script used for E-P/P-P loop classification is not named, versioned, or hosted at a stated location, complicating reproducibility of exactly this annotation step.
- Classification quality depends entirely on the completeness of the supplied enhancer and promoter BED references; anchors lacking overlaps are simply labeled 'neither'.

## Evidence

- [discussion] The unique alignments [MAPQ (mapping quality) ≥ 30] and nonredundant PETs are retained by calling the CPU "dedup" module.: [evidence span withheld — anchor status: not_found]
