---
name: genomic-interval-intersection-with-bedtools
description: Use when ChIA-PIPE has produced peak-supported loops (PET count >= 3)
  with unique loop IDs from RNAPII ChIA-PET data and you must annotate each loop as
  E-P, P-P, or neither using enhancer/promoter BED annotations.
when_to_use_negative:
- Input loops lack peak support or fall below the PET count >= 3 threshold from ChIA-PIPE
  loop calling, since E-P annotation is applied only after such loops are called.
- No enhancer or promoter BED annotation exists for the organism/genome build, making
  the intersection-based E-P/P-P flags impossible to compute.
- The input is raw ChIA-PET data (FASTQ) rather than called loops — loop calling,
  not annotation, is required first.
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ChIA-PIPE
  role: Produces the peak-supported loop anchor BED files with unique loop IDs that
    are the classification input
  repo: https://github.com/TheJacksonLaboratory/ChIA-PIPE
- name: bedtools
  role: Intersects loop anchor BED files with enhancer and promoter BED files on the
    same chromosome to flag anchor overlaps
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1126/sciadv.aay2078
    title: 'ChIA-PIPE: A fully automated pipeline for comprehensive ChIA-PET data
      analysis and visualization'
  source_content_hash: sha256:42fb2b95e68bb8bd2673f2c157b61178213e6a448cd286bd4d1473c544c57841
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TheJacksonLaboratory/ChIA-PIPE
  tools:
  - AmpliconArchitect
  - BedTools
  - ChIA-PIPE
  - MACS2
  - R
  - bedtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- genomic-interval-intersection
- genomic-interval-merging-with-bedtools
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Genomic interval intersection with bedtools for ChIA-PIPE E-P/P-P loop classification

## Summary

Classifies RNAPII ChIA-PET loops from ChIA-PIPE into enhancer-promoter (E-P), promoter-promoter (P-P), or neither by running bedtools intersect of loop anchor BED files against enhancer and promoter BED files, then writing per-category subset loop files for separate BASIC Browser tracks.

## When to use

Use when ChIA-PIPE has produced peak-supported loops (PET count >= 3) with unique loop IDs from RNAPII ChIA-PET data and you must annotate each loop as E-P, P-P, or neither using enhancer/promoter BED annotations.

## When NOT to use

- Input loops lack peak support or fall below the PET count >= 3 threshold from ChIA-PIPE loop calling, since E-P annotation is applied only after such loops are called.
- No enhancer or promoter BED annotation exists for the organism/genome build, making the intersection-based E-P/P-P flags impossible to compute.
- The input is raw ChIA-PET data (FASTQ) rather than called loops — loop calling, not annotation, is required first.

## Inputs

- ChIA-PIPE loop left anchor BED file (loops with PET count >= 3 and peak support, unique loop IDs)
- ChIA-PIPE loop right anchor BED file
- Enhancer BED file
- Promoter BED file

## Outputs

- ep_loops.bed (E-P loops, original loop BED columns plus class label column)
- pp_loops.bed (P-P loops, original loop BED columns plus class label column)
- neither_loops.bed (unclassified loops, original loop BED columns plus class label column)

## How to apply

Load the loop left/right anchor BED files produced by ChIA-PIPE loop calling, where each loop carries a unique loop ID. Run bedtools intersect between each anchor BED file and the enhancer BED file, and between each anchor BED and the promoter BED file, requiring coordinate overlap on the same chromosome to flag which anchors overlap an enhancer or promoter. For each loop ID, classify it as E-P if exactly one anchor overlaps an enhancer and the other overlaps a promoter, P-P if both anchors overlap promoters, and neither otherwise; the article notes a custom Python script performs this per-loop-ID determination. Write the classified loops into three per-category subset files (ep_loops.bed, pp_loops.bed, neither_loops.bed), preserving the original loop BED columns plus a class label column. The rationale is that ChIA-PIPE annotates E-P loops so each category can be rendered in a separate BASIC Browser track.

## Related tools

- **ChIA-PIPE** (Produces the peak-supported loop anchor BED files with unique loop IDs that are the classification input) — https://github.com/TheJacksonLaboratory/ChIA-PIPE
- **bedtools** (Intersects loop anchor BED files with enhancer and promoter BED files on the same chromosome to flag anchor overlaps)

## Examples

```
bedtools intersect -a loop_left_anchor.bed -b enhancer.bed; bedtools intersect -a loop_right_anchor.bed -b promoter.bed
```

## Evaluation signals

- The union of ep_loops.bed, pp_loops.bed, and neither_loops.bed covers every input loop ID exactly once (no loops lost or duplicated).
- Each emitted per-category file contains a non-empty class label column.
- E-P class appears only when exactly one anchor overlaps an enhancer and the other overlaps a promoter.
- P-P class appears only when both anchors overlap promoters.
- Subset files preserve the original loop BED columns so they can be displayed in separate BASIC Browser tracks.

## Limitations

- The custom Python script used for E-P/P-P loop classification is not named, versioned, or hosted at a stated location, so the exact overlap logic may not be reproducible from the paper alone.
- The reference genome version used for alignment is not specified in the article section, complicating enhancer/promoter BED coordinate matching.
- Loops that are neither E-P nor P-P (e.g., enhancer-enhancer or promoter-enhancer ambiguous cases) are assigned to a 'neither' class rather than resolved further.

## Evidence

- [discussion] ChIA-PIPE automatically annotates E-P loops and promoter-promoter (P-P) loops from RNAPII ChIA-PET datasets: [evidence span withheld — anchor status: not_found]
