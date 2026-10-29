---
name: bed-file-subsetting-and-partitioning
description: Use when you have peak-supported ChIA-PET/HiChIP loop BED files (e.g.,
  RNAPII ChIA-PET loops with PET count >= 3 and unique loop IDs) and need to split
  them into category-specific tracks (E-P vs P-P vs neither) for separate browser
  visualization.
when_to_use_negative:
- Loop anchors come from a factor whose binding is not enhancer/promoter-informative
  (e.g., CTCF ChIA-PET), where E-P/P-P classification from RNAPII ChIA-PET data is
  not biologically meaningful.
- Input loops have not passed ChIA-PIPE loop calling thresholds (PET count >= 3 with
  peak support), so anchor coordinates and unique loop IDs required for classification
  are absent.
- The goal is 2D contact-map visualization (.hic/.cool for Juicebox/HiGlass) rather
  than per-category loop annotation tracks.
tools:
- name: ChIA-PIPE
  role: Produces the peak-supported, uniquely-IDed loop BED files that serve as the
    classification input; its E-P/P-P annotation step assigns each loop ID a class
  repo: https://github.com/TheJacksonLaboratory/ChIA-PIPE
- name: bedtools
  role: Performs coordinate intersection between each loop anchor BED file and the
    enhancer/promoter BED references on the same chromosome
- name: CPU (ChIA-PET Utilities)
  role: Modularized executables integrated by ChIA-PIPE for processing ChIA-PET data
    from raw reads to interactions, upstream of loop annotation
  repo: https://github.com/cheehongsg/CPU
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1126/sciadv.aay2078
    title: 'ChIA-PIPE: A fully automated pipeline for comprehensive ChIA-PET data
      analysis and visualization'
  source_content_hash: sha256:6a8073cbeb80e5ce2161ce9df1290376888c89f7bff048b907df45430ff34fab
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TheJacksonLaboratory/ChIA-PIPE
  tools:
  - CPU (ChIA-PET Utilities)
  - ChIA-PIPE
  - bedtools
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# BED file subsetting and partitioning

## Summary

Partitions ChIA-PIPE-called chromatin interaction loops (PET count >= 3, peak-supported, each with a unique loop ID) into enhancer-promoter (E-P), promoter-promoter (P-P), and 'neither' BED subsets by testing coordinate overlap of each loop's left/right anchor BED files against enhancer and promoter BED references using bedtools intersect.

## When to use

Use when you have peak-supported ChIA-PET/HiChIP loop BED files (e.g., RNAPII ChIA-PET loops with PET count >= 3 and unique loop IDs) and need to split them into category-specific tracks (E-P vs P-P vs neither) for separate browser visualization.

## When NOT to use

- Loop anchors come from a factor whose binding is not enhancer/promoter-informative (e.g., CTCF ChIA-PET), where E-P/P-P classification from RNAPII ChIA-PET data is not biologically meaningful.
- Input loops have not passed ChIA-PIPE loop calling thresholds (PET count >= 3 with peak support), so anchor coordinates and unique loop IDs required for classification are absent.
- The goal is 2D contact-map visualization (.hic/.cool for Juicebox/HiGlass) rather than per-category loop annotation tracks.

## Inputs

- Loop left anchor BED file from ChIA-PIPE loop calling (PET count >= 3, peak support, unique loop ID per row)
- Loop right anchor BED file from ChIA-PIPE loop calling
- Enhancer BED file (reference annotation)
- Promoter BED file (reference annotation)

## Outputs

- ep_loops.bed — loops classified enhancer-promoter, original loop BED columns plus class label column
- pp_loops.bed — loops classified promoter-promoter, original loop BED columns plus class label column
- neither_loops.bed — loops qualifying as neither E-P nor P-P, original loop BED columns plus class label column

## How to apply

Load the loop left/right anchor BED files produced by ChIA-PIPE loop calling, where every loop carries a unique loop ID. Run bedtools intersect between each anchor BED file and the enhancer BED file, and between each anchor BED file and the promoter BED file, requiring coordinate overlap on the same chromosome to flag which anchors overlap an enhancer or promoter. For each loop ID, classify it as E-P if exactly one anchor overlaps an enhancer and the other overlaps a promoter, P-P if both anchors overlap promoters, and 'neither' otherwise (in ChIA-PIPE this classification is done by a custom Python script over the loop IDs). Write classified loops into three per-category subset files (ep_loops.bed, pp_loops.bed, neither_loops.bed) preserving the original loop BED columns plus a class label column. The rationale is that RNAPII ChIA-PET loops anchored between a regulatory enhancer and a promoter versus two promoters have distinct biological interpretations, so per-category BASIC Browser tracks can display them separately.

## Related tools

- **ChIA-PIPE** (Produces the peak-supported, uniquely-IDed loop BED files that serve as the classification input; its E-P/P-P annotation step assigns each loop ID a class) — https://github.com/TheJacksonLaboratory/ChIA-PIPE
- **bedtools** (Performs coordinate intersection between each loop anchor BED file and the enhancer/promoter BED references on the same chromosome)
- **CPU (ChIA-PET Utilities)** (Modularized executables integrated by ChIA-PIPE for processing ChIA-PET data from raw reads to interactions, upstream of loop annotation) — https://github.com/cheehongsg/CPU

## Examples

```
bedtools intersect -a loop_left_anchors.bed -b enhancers.bed -wa -wb > left_anchor_enhancer_overlaps.tsv
```

## Evaluation signals

- Union of ep_loops.bed, pp_loops.bed, and neither_loops.bed covers every input loop ID exactly once — no loops lost or duplicated (partition invariant).
- Each emitted subset file contains a non-empty class label column as a QC check on the annotation output.
- Every E-P loop has exactly one anchor intersecting the enhancer BED and the other anchor intersecting the promoter BED on the same chromosome.
- Every P-P loop has both anchors intersecting the promoter BED; no loop is labeled E-P with two promoter-overlapping anchors.
- Input loop BED properties (PET count >= 3, peak support, unique loop ID per row) are preserved verbatim in each subset file.

## Limitations

- Loops whose anchors overlap neither an enhancer nor a promoter fall into the 'neither' subset; classification is purely coordinate-overlap-based and does not resolve overlapping or ambiguous enhancer/promoter definitions.
- The custom Python script used for E-P/P-P loop classification is not named, versioned, or hosted at a stated location, so reproducibility depends on reimplementation against the described logic.
- The article section does not specify the reference genome version used for anchor/annotation coordinates, so BED coordinate system (e.g., hg19) must be confirmed independently.

## Evidence

- [discussion] After loops are called (PET count ≥3 and peak support), each loop is assigned a unique ID.: "After loops are called (PET count ≥3 and peak support), each loop is assigned a unique ID."
- [readme] CPU processes ChIA-PET data to interactions: [evidence span withheld — anchor status: not_found]
