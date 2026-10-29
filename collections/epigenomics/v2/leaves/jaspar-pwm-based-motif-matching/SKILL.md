---
name: jaspar-pwm-based-motif-matching
description: 'Use when testing the loop-extrusion convergent-orientation rule at two-anchor
  loop calls: you need CTCF motif occurrence and strand per anchor to classify each
  loop as convergent, tandem, or divergent against a reference genome such as mm10.'
when_to_use_negative:
- The goal is de novo motif discovery — FIMO with a fixed JASPAR PWM (MA0139.1) only
  scans for a known CTCF motif; it cannot discover new motifs.
- Loops lack CTCF binding on both anchors, or the loop calls use a different genome
  build than the sequence/PWM scan (e.g., hg38 calls scanned against mm10) — orientation
  cannot be validly classified.
- Input is *.postprocessed.all_candidates.bedpe rather than the summit-level *.postprocessed.summits.bedpe
  — the orientation analysis operates on loop summits with per-loop cell support (outlier_count),
  not on unsummarized candidate clusters.
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0622
tools:
- name: FIMO
  role: Scans the extracted anchor sequences with the JASPAR MA0139.1 CTCF position
    weight matrix to report motif occurrences and strand at each anchor
- name: SnapHiC
  role: Produces the *.postprocessed.summits.bedpe loop calls whose anchor coordinates
    (chr1, x1, x2, chr2, y1, y2) and outlier_count feed the motif-orientation analysis
  repo: https://github.com/HuMingLab/SnapHiC
- name: JASPAR
  role: Source of the MA0139.1 CTCF position weight matrix used for motif scanning
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-021-01231-2
    title: 'SnapHiC: a computational pipeline to identify chromatin loops from single-cell
      Hi-C data'
  source_content_hash: sha256:3ea0fc090c897a7912b4927af8573792a0771061825e55366334e4c600c81d2d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuMingLab/SnapHiC
  tools:
  - FIMO
  - JASPAR
  - SnapHiC
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# jaspar-pwm-based-motif-matching

## Summary

Scans genomic sequences against a known transcription factor position weight matrix — here FIMO with the JASPAR MA0139.1 CTCF PWM on mm10 anchor sequences extracted from SnapHiC loop calls — recovering each motif's occurrence and strand. The strand information is what enables convergent/tandem/divergent orientation tests of loop-extrusion predictions at scHi-C loop anchors.

## When to use

Use when testing the loop-extrusion convergent-orientation rule at two-anchor loop calls: you need CTCF motif occurrence and strand per anchor to classify each loop as convergent, tandem, or divergent against a reference genome such as mm10.

## When NOT to use

- The goal is de novo motif discovery — FIMO with a fixed JASPAR PWM (MA0139.1) only scans for a known CTCF motif; it cannot discover new motifs.
- Loops lack CTCF binding on both anchors, or the loop calls use a different genome build than the sequence/PWM scan (e.g., hg38 calls scanned against mm10) — orientation cannot be validly classified.
- Input is *.postprocessed.all_candidates.bedpe rather than the summit-level *.postprocessed.summits.bedpe — the orientation analysis operates on loop summits with per-loop cell support (outlier_count), not on unsummarized candidate clusters.

## Inputs

- SnapHiC loop calls: *.postprocessed.summits.bedpe, 17-column tab-separated BEDPE with anchor coordinates (chr1, x1, x2, chr2, y1, y2) and outlier_count
- mm10 reference genome FASTA matching the genome build of the loop calls (SnapHiC supports mm10, hg19, hg38)
- JASPAR MA0139.1 CTCF position weight matrix in FIMO-compatible motif format

## Outputs

- Per-anchor CTCF motif occurrence table: motif coordinates and strand (+/-) at each loop anchor
- Per-loop orientation labels: convergent, tandem, or divergent, for loops with a motif on both anchors
- Convergent-orientation fraction among loops with outlier_count >= 50 (expected within the 63.6%-78.7% range) plus the per-loop orientation table

## How to apply

Load the SnapHiC loop calls from the 17-column tab-separated *.postprocessed.summits.bedpe, retaining the anchor coordinates (chr1, x1, x2, chr2, y1, y2) and the outlier_count column that records how many single cells support each loop. Extract both anchor sequences from the mm10 reference genome FASTA using these coordinates, keeping the same genome build as the loop calls (SnapHiC supports mm10, hg19, hg38). Scan every anchor sequence with FIMO using the JASPAR MA0139.1 CTCF position weight matrix, recording each motif hit's position and strand (+/-). For loops with a CTCF motif on both anchors, classify the pair's orientation from the two strands — convergent when the motifs face one another, tandem when they point the same way, divergent when they face apart. Subset to loops detected in at least 50 cells (outlier_count >= 50) and compute the fraction of convergent-orientation loops, reporting the overall fraction alongside the full per-loop orientation table. The rationale is that the loop-extrusion model predicts convergent CTCF orientation, so a clear convergent preference (63.6%-78.7% in the reference analysis at >= 50 cells) validates both the motif scan and the SnapHiC loop calls.

## Related tools

- **FIMO** (Scans the extracted anchor sequences with the JASPAR MA0139.1 CTCF position weight matrix to report motif occurrences and strand at each anchor)
- **SnapHiC** (Produces the *.postprocessed.summits.bedpe loop calls whose anchor coordinates (chr1, x1, x2, chr2, y1, y2) and outlier_count feed the motif-orientation analysis) — https://github.com/HuMingLab/SnapHiC
- **JASPAR** (Source of the MA0139.1 CTCF position weight matrix used for motif scanning)

## Examples

```
fimo MA0139.1.meme mm10_anchor_sequences.fa
```

## Evaluation signals

- The convergent-orientation fraction among loops with outlier_count >= 50 falls within the reported 63.6%-78.7% range, showing a clear preference for convergent orientation.
- Every orientation-labeled loop has a strand-assigned CTCF motif call on each anchor; loops without motifs on both anchors are excluded rather than assigned a default label.
- FIMO hit coordinates lie inside the anchor intervals (chr1:x1-x2 and chr2:y1-y2) taken from the 17-column *.postprocessed.summits.bedpe.
- The number of loops retained after the outlier_count >= 50 subset matches the count of summits.bedpe rows meeting that threshold.
- Orientation labels are invariant to swapping anchor1/anchor2 order in the bedpe (convergent stays convergent, divergent stays divergent).

## Limitations

- Only loops with CTCF binding on both anchors can be orientation-classified; single-anchor or motif-free loops must be dropped, shrinking the analyzed set.
- The convergent fraction is sensitive to the cell-support threshold: the 63.6%-78.7% range applies to loops detected in at least 50 cells (outlier_count >= 50), so including lower-support loops will shift the estimate.
- The anchor-sequence extraction must use the same reference build as the loop calls; SnapHiC currently accepts only mm10, hg19 and hg38, and the mESC analysis uses mm10. FIMO detection also depends on the motif score threshold chosen, which determines which CTCF occurrences count.

## Evidence

- [abstract] Using scHi-C data from 742 mouse embryonic stem cells, we benchmark SnapHiC against a number of computational tools: "Using scHi-C data from 742 mouse embryonic stem cells, we benchmark SnapHiC against a number of computational tools"
- [methods] We removed all 10KB bins with mappability <=0.8, and all 10KB bins overlapped with the ENCODE blacklist regions.: "We removed all 10KB bins with mappability <=0.8, and all 10KB bins overlapped with the ENCODE blacklist regions."
- [readme] It is the prefix of the output files, including *.postprocessed.summits.bedpe*, *.postprocessed.all_candidates.bedpe*, and *allChr.[hic/cool]*.: "It is the prefix of the output files, including *.postprocessed.summits.bedpe*, *.postprocessed.all_candidates.bedpe*, and *allChr.[hic/cool]*."
