---
name: fimo-motif-scanning
description: Use when you have FASTA sequences for two sets of genomic anchors (e.g.,
  loop endpoints from a bedpe) and need strand-resolved motif occurrences to classify
  pairwise motif orientation.
when_to_use_negative:
- The analysis goal is motif discovery rather than scanning known motifs against sequences
  — FIMO requires a supplied PWM such as MA0139.1
- Loop anchors are from a genome build other than the reference used to build the
  motif scan FASTA (e.g., hg19/hg38 anchors against mm10) — coordinates must match
  the FASTA build
- Only one anchor per pair has CTCF binding — orientation classification requires
  motifs on both anchors
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: FIMO
  role: Scans extracted anchor sequences with the JASPAR MA0139.1 CTCF PWM to report
    motif occurrences and strand at each loop anchor
- name: SnapHiC
  role: Produces the *.postprocessed.summits.bedpe loop calls whose anchors are the
    intervals scanned for CTCF motifs
  repo: https://github.com/HuMingLab/SnapHiC
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-021-01231-2
    title: 'SnapHiC: a computational pipeline to identify chromatin loops from single-cell
      Hi-C data'
  source_content_hash: sha256:b4b666bbe6051b504b3273f9c03b17478929b04be7b2c6770c4e9ee20a4ab0fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuMingLab/SnapHiC
  tools:
  - FIMO
  - SnapHiC
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# fimo-motif-scanning

## Summary

Scan genomic anchor sequences with FIMO using the JASPAR MA0139.1 CTCF position weight matrix to identify CTCF motif occurrences and their strand, enabling classification of motif orientation (convergent, tandem, divergent) across pairs of genomic intervals such as loop anchors.

## When to use

Use when you have FASTA sequences for two sets of genomic anchors (e.g., loop endpoints from a bedpe) and need strand-resolved motif occurrences to classify pairwise motif orientation.

## When NOT to use

- The analysis goal is motif discovery rather than scanning known motifs against sequences — FIMO requires a supplied PWM such as MA0139.1
- Loop anchors are from a genome build other than the reference used to build the motif scan FASTA (e.g., hg19/hg38 anchors against mm10) — coordinates must match the FASTA build
- Only one anchor per pair has CTCF binding — orientation classification requires motifs on both anchors

## Inputs

- 17-column tab-separated bedpe of SnapHiC loop summits (*.postprocessed.summits.bedpe) with anchor coordinates (chr1, x1, x2, chr2, y1, y2) and outlier_count
- mm10 reference genome FASTA
- JASPAR MA0139.1 CTCF position weight matrix in MEME format

## Outputs

- FIMO hit table of CTCF motif occurrences with coordinates and strand per anchor
- Per-loop motif orientation table (convergent / tandem / divergent)
- Overall convergent-orientation fraction, optionally stratified by outlier_count threshold

## How to apply

Extract the anchor genomic sequences for both anchors of every loop from the mm10 reference genome FASTA, then scan them with FIMO using the JASPAR MA0139.1 CTCF position weight matrix to recover motif positions and strands. For each loop with CTCF binding on both anchors, combine the two anchors' motif strands to classify the pair as convergent, tandem, or divergent, per the loop extrusion model prediction that convergent CTCF motifs flank loops. Subset the loops by detection support (e.g., outlier_count >= 50 cells from the *.postprocessed.summits.bedpe file) and compute the fraction of convergent-orientation loops. Judge success by comparing the convergent fraction to the published range.

## Related tools

- **FIMO** (Scans extracted anchor sequences with the JASPAR MA0139.1 CTCF PWM to report motif occurrences and strand at each loop anchor)
- **SnapHiC** (Produces the *.postprocessed.summits.bedpe loop calls whose anchors are the intervals scanned for CTCF motifs) — https://github.com/HuMingLab/SnapHiC

## Examples

```
bedtools getfasta -fi mm10.fa -bed anchors.bed -name -s > anchors.fa && fimo --thresh 1e-4 MA0139.1.meme anchors.fa
```

## Evaluation signals

- The convergent-orientation fraction among loops with CTCF binding on both anchors and outlier_count >= 50 falls within the reported 63.6%-78.7% range
- FIMO output assigns each hit a strand, and every loop in the orientation table has exactly one motif-pair class (convergent, tandem, or divergent)
- Only loops with motif occurrences on both anchors are retained for orientation classification
- Anchor sequences extracted from the mm10 FASTA match the bedpe anchor coordinates in length and position

## Limitations

- The orientation analysis requires CTCF binding on both anchors; loops with a motif on only one anchor are excluded from the convergent-fraction calculation
- Convergent fraction depends on the detection threshold applied (the reported 63.6%-78.7% range applies when at least 50 cells are used)
- Mappability and blacklist filtering upstream (10KB bins with mappability <=0.8 and ENCODE blacklist regions) affects which loop anchors are available for scanning

## Evidence

- [other] Extract the anchor genomic sequences from the mm10 reference genome FASTA for both anchors of every loop.: [evidence span withheld — anchor status: not_found]
- [other] Scan the anchor sequences with FIMO using the JASPAR MA0139.1 CTCF position weight matrix: [evidence span withheld — anchor status: not_found]
- [other] For SnapHiC-identified loops with CTCF binding on both anchors, there was a clear preference in convergent orientation, ranging from 63.6% to 78.7% when at least 50 cells are used.: [evidence span withheld — anchor status: not_found]
- [readme] Files for mm10, hg19 and hg38 are included in the `ext` directory.: "chrom.sizes file for the genome of interest, which can be downloaded from [here](https://hgdownload.soe.ucsc.edu/downloads.html). Files for mm10, hg19 and hg38 are included in the `ext` directory."
