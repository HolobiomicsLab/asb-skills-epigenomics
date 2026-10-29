---
name: ctcf-motif-orientation-classification
description: Use when you have SnapHiC loop calls (*.postprocessed.summits.bedpe)
  with CTCF binding on both anchors and need to quantify convergent vs tandem vs divergent
  CTCF motif orientation.
when_to_use_negative:
- Loops where CTCF does not bind both anchors — motif orientation is undefined without
  motifs on both anchors
- Loop calls in a format other than SnapHiC's postprocessed summits bedpe, or from
  a genome build other than mm10 (the JASPAR scan and reference sequences assume mm10
  coordinates)
- Bulk Hi-C or other chromatin-interaction datasets where per-cell detection counts
  (outlier_count) needed for the >= 50 cell filter are not available
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: SnapHiC
  role: Produces the loop calls (*.postprocessed.summits.bedpe) whose anchors are
    scanned for CTCF motif orientation
  repo: https://github.com/HuMingLab/SnapHiC
- name: FIMO
  role: Scans loop anchor sequences with the JASPAR MA0139.1 CTCF PWM to identify
    motif occurrences and strands at each anchor
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-021-01231-2
    title: 'SnapHiC: a computational pipeline to identify chromatin loops from single-cell
      Hi-C data'
  source_content_hash: sha256:d1e5b38c28e65e0c16e18254df197650e824befc536bff38e6f3678b293bdf8a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuMingLab/SnapHiC
  tools:
  - FIMO
  - SnapHiC
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# ctcf-motif-orientation-classification

## Summary

Classifies each SnapHiC-identified chromatin loop as convergent, tandem, or divergent based on CTCF motif strands at its two anchors, testing the loop extrusion model's prediction that loops favor convergent orientation.

## When to use

Use when you have SnapHiC loop calls (*.postprocessed.summits.bedpe) with CTCF binding on both anchors and need to quantify convergent vs tandem vs divergent CTCF motif orientation.

## When NOT to use

- Loops where CTCF does not bind both anchors — motif orientation is undefined without motifs on both anchors
- Loop calls in a format other than SnapHiC's postprocessed summits bedpe, or from a genome build other than mm10 (the JASPAR scan and reference sequences assume mm10 coordinates)
- Bulk Hi-C or other chromatin-interaction datasets where per-cell detection counts (outlier_count) needed for the >= 50 cell filter are not available

## Inputs

- SnapHiC loop calls: *.postprocessed.summits.bedpe (17-column tab-separated bedpe with chr1, x1, x2, chr2, y1, y2 and outlier_count per loop)
- mm10 reference genome FASTA
- JASPAR MA0139.1 CTCF position weight matrix (FIMO format)
- Optional: CTCF ChIP-seq peak calls to subset loops with CTCF binding on both anchors

## Outputs

- Per-loop orientation table assigning convergent, tandem, or divergent to each loop with motifs on both anchors
- Overall convergent-orientation fraction among loops with outlier_count >= 50

## How to apply

Load the 17-column tab-separated *.postprocessed.summits.bedpe file, keeping anchor coordinates (chr1, x1, x2, chr2, y1, y2) and outlier_count. Extract the anchor sequences from the mm10 reference genome FASTA for both anchors of every loop. Scan the anchor sequences with FIMO using the JASPAR MA0139.1 CTCF position weight matrix to identify motif occurrences and their strand at each anchor. Classify each loop's motif pair as convergent, tandem, or divergent from the relative strands of motifs on the two anchors. Finally, subset to loops detected in at least 50 cells (outlier_count >= 50) and compute the fraction of convergent loops; loop extrusion predicts this fraction should be clearly enriched above chance.

## Related tools

- **SnapHiC** (Produces the loop calls (*.postprocessed.summits.bedpe) whose anchors are scanned for CTCF motif orientation) — https://github.com/HuMingLab/SnapHiC
- **FIMO** (Scans loop anchor sequences with the JASPAR MA0139.1 CTCF PWM to identify motif occurrences and strands at each anchor)

## Examples

```
fimo --motif MA0139.1 MA0139.1.meme mm10_anchor_sequences.fa
```

## Evaluation signals

- The convergent-orientation fraction among loops detected in >= 50 cells (outlier_count >= 50) falls within the reported 63.6%-78.7% range
- Every loop in the orientation table has a motif strand call on both anchors; loops lacking motifs on either anchor are excluded rather than misclassified
- Orientation classes are exhaustive and mutually exclusive (each loop labeled exactly one of convergent, tandem, or divergent)
- Anchor sequences extracted from the FASTA match the bedpe coordinates in the mm10 genome build (no coordinate offset errors)
- The subset fraction is computed on the outlier_count >= 50 subset, not the full loop set

## Limitations

- SnapHiC loop calling quality depends on mappability filtering: 10KB bins with mappability <=0.8 and ENCODE blacklist regions are removed, so loops near filtered bins are absent and cannot be assessed
- The convergent fraction varies (63.6%-78.7%) with the minimum cell-count threshold; results depend on choosing outlier_count >= 50
- Orientation is only assessable when a CTCF motif is found on both anchors; anchors without motif hits must be dropped, potentially biasing the fraction

## Evidence

- [methods] postprocessing produces summit loop calls: "(5) postprocessing: clustering nearby loop candidates and identifying loop summits"
- [results] convergent orientation finding: [evidence span withheld — anchor status: not_found]
- [methods] mm10 reference genome files: "Files for mm10, hg19 and hg38 are included in the `ext` directory."
- [readme] summits bedpe output format: "It is the prefix of the output files, including *.postprocessed.summits.bedpe*, *.postprocessed.all_candidates.bedpe*, and *allChr.[hic/cool]*"
- [readme] mappability and blacklist filtering: "We removed all 10KB bins with mappability <=0.8, and all 10KB bins overlapped with the ENCODE blacklist regions."
