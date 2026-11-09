---
name: chia-pet-loop-anchor-processing
description: Use when ChIA-PIPE has called RNAPII ChIA-PET loops (PET count ≥3 with
  peak support, unique loop IDs) and you need to classify each loop as E-P, P-P, or
  neither using enhancer/promoter genomic coordinates.
when_to_use_negative:
- Input loops have not yet passed ChIA-PIPE loop calling thresholds (PET count ≥3
  with peak support) — classify only the called loop set
- No reference enhancer or promoter BED annotations are available for the cell type/genome,
  so anchor-overlap classification cannot be made
- The goal is peak or loop calling itself (linker filtering, mapping, peak calling,
  loop calling), not post-hoc E-P/P-P annotation of already-called loops
tools:
- name: ChIA-PIPE
  role: Produces the peak-supported called loops and anchor BED files that this skill
    annotates; includes the enhancer-promoter loop annotation step
  repo: https://github.com/TheJacksonLaboratory/ChIA-PIPE
- name: CPU (ChIA-PET Utilities)
  role: Modularized executables integrated by ChIA-PIPE for processing ChIA-PET data
    up to called interactions
  repo: https://github.com/cheehongsg/CPU
- name: bedtools
  role: bedtools intersect flags which loop anchors overlap enhancer or promoter coordinates
    on the same chromosome
- name: BASIC Browser
  role: Displays the per-category loop subset files (E-P, P-P) as separate interactive
    loop tracks
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1126/sciadv.aay2078
    title: 'ChIA-PIPE: A fully automated pipeline for comprehensive ChIA-PET data
      analysis and visualization'
  source_content_hash: sha256:19b1cc667741b1118b9a7f93d419e5bf0277d754aeedaf7ecd48ebb82e04c74b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TheJacksonLaboratory/ChIA-PIPE
  tools:
  - BASIC Browser
  - CPU (ChIA-PET Utilities)
  - ChIA-PIPE
  - bedtools
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# chia-pet-loop-anchor-processing

## Summary

Annotates peak-supported RNAPII ChIA-PET loops (PET count ≥3) as enhancer-promoter (E-P) or promoter-promoter (P-P) by testing anchor overlap against enhancer and promoter BED files, then emits per-category loop subsets for separate BASIC Browser tracks.

## When to use

Use when ChIA-PIPE has called RNAPII ChIA-PET loops (PET count ≥3 with peak support, unique loop IDs) and you need to classify each loop as E-P, P-P, or neither using enhancer/promoter genomic coordinates.

## When NOT to use

- Input loops have not yet passed ChIA-PIPE loop calling thresholds (PET count ≥3 with peak support) — classify only the called loop set
- No reference enhancer or promoter BED annotations are available for the cell type/genome, so anchor-overlap classification cannot be made
- The goal is peak or loop calling itself (linker filtering, mapping, peak calling, loop calling), not post-hoc E-P/P-P annotation of already-called loops

## Inputs

- Loop left anchor BED file from ChIA-PIPE loop calling (loops with PET count ≥3 and peak support, unique loop IDs)
- Loop right anchor BED file from ChIA-PIPE loop calling
- Enhancer BED file (genomic coordinates on the same reference chromosome space)
- Promoter BED file (genomic coordinates on the same reference chromosome space)

## Outputs

- ep_loops.bed: subset of E-P-classified loops (original loop BED columns plus class label column)
- pp_loops.bed: subset of P-P-classified loops (original loop BED columns plus class label column)
- neither_loops.bed: loops classified as neither E-P nor P-P (original loop BED columns plus class label column)
- Per-category loop tracks loadable in BASIC Browser

## How to apply

Start from the loop left/right anchor BED files produced by ChIA-PIPE loop calling, where each loop with PET count ≥3 and peak support carries a unique loop ID. Run bedtools intersect between each anchor BED file and the enhancer BED file, and again against the promoter BED file, requiring coordinate overlap on the same chromosome to flag which anchors overlap an enhancer or a promoter. Classify each loop ID as E-P if exactly one anchor overlaps an enhancer and the other a promoter, as P-P if both anchors overlap promoters, and as neither otherwise; in the article this step is performed by a custom Python script. Write the classified loops into three per-category subset files (e.g. ep_loops.bed, pp_loops.bed, neither_loops.bed), preserving the original loop BED columns and adding a class label column, so each category can be rendered as a separate BASIC Browser track.

## Related tools

- **ChIA-PIPE** (Produces the peak-supported called loops and anchor BED files that this skill annotates; includes the enhancer-promoter loop annotation step) — https://github.com/TheJacksonLaboratory/ChIA-PIPE
- **CPU (ChIA-PET Utilities)** (Modularized executables integrated by ChIA-PIPE for processing ChIA-PET data up to called interactions) — https://github.com/cheehongsg/CPU
- **bedtools** (bedtools intersect flags which loop anchors overlap enhancer or promoter coordinates on the same chromosome)
- **BASIC Browser** (Displays the per-category loop subset files (E-P, P-P) as separate interactive loop tracks)

## Evaluation signals

- Union of ep_loops.bed, pp_loops.bed, and neither_loops.bed covers every input loop ID exactly once (no loops lost or duplicated)
- Each emitted per-category file contains a non-empty class label column
- Every loop is assigned exactly one of the three classes (E-P, P-P, neither) — categories are mutually exclusive per loop ID
- Original loop BED columns are preserved unchanged in each subset file, with only the class label appended
- E-P and P-P assignments match the anchor-overlap rule: E-P = one enhancer-overlapping anchor + one promoter-overlapping anchor; P-P = both anchors promoter-overlapping

## Limitations

- The custom Python script used to determine whether each loop ID qualifies as E-P, P-P, or neither is not named, versioned, or hosted at a stated location in the paper
- Classification quality depends entirely on the supplied enhancer and promoter BED annotations; loops are labeled 'neither' when anchors match neither feature
- Enhancer and promoter BED files must be coordinate-compatible (same chromosome names and genome build) with the ChIA-PIPE loop anchor files for bedtools intersect to produce valid overlaps

## Evidence
