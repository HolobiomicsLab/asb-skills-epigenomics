---
name: ctcf-chipseq-peak-orientation
description: Use when you have BED-formatted genomic peaks or features lacking strand
  annotation and a second BED file of the same features with '+'/'-' strand labels,
  and you need every feature oriented before strand asymmetry analysis.
when_to_use_negative:
- Input features already carry strand annotation — Asymmetron's orientation step is
  only needed for unassigned features
- Peaks do not overlap any strand-annotated motif, so no strand sign can be transferred;
  use a nearest-neighbor annotation strategy instead
- The goal is computing strand asymmetry P-values rather than assigning orientation
  — use the consecutive_patterns, contained_asymmetries, or pairwise_asymmetries functions
  instead
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Asymmetron
  role: Provides the orientation function that performs strand assignment of an unassigned
    feature based on another overlapping feature
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: Pybedtools
  role: Implements the BED-formatted interval overlap used to find overlapping motif
    instances and the closest center distance
- name: Python
  role: Host language for the Asymmetron toolkit and the peak–motif overlap/orientation
    script
- name: pandas
  role: Loads and filters the peak and motif BED tables during orientation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkaa1052
    title: 'Asymmetron: a toolkit for the identification of strand asymmetry patterns
      in biological sequences'
  source_content_hash: sha256:c42bdb1ad5b82a956e18fdd767e29f23a5b80ec2e3e568073a4ee2dd5010e73a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ahituv-lab/Asymmetron
  tools:
  - Asymmetron
  - Pybedtools
  - Python
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# ctcf-chipseq-peak-orientation

## Summary

Orients strand-unannotated CTCF ChIP-seq peaks (BED, hg38) by transferring the strand sign of overlapping strand-annotated CTCF motif occurrences (e.g. MA0139.1 JASPAR/Unibind maps), using Asymmetron's orientation function implemented with pybedtools interval overlap. This enables downstream strand asymmetry analysis of the peaks.

## When to use

Use when you have BED-formatted genomic peaks or features lacking strand annotation and a second BED file of the same features with '+'/'-' strand labels, and you need every feature oriented before strand asymmetry analysis.

## When NOT to use

- Input features already carry strand annotation — Asymmetron's orientation step is only needed for unassigned features
- Peaks do not overlap any strand-annotated motif, so no strand sign can be transferred; use a nearest-neighbor annotation strategy instead
- The goal is computing strand asymmetry P-values rather than assigning orientation — use the consecutive_patterns, contained_asymmetries, or pairwise_asymmetries functions instead

## Inputs

- Unannotated CTCF ChIP-seq peak BED file (chromosome, start, end; hg38)
- Strand-annotated CTCF motif occurrence BED file (e.g. MA0139.1 from JASPAR 2020 / Unibind PWM motif maps, 2019 release; hg38)

## Outputs

- ctcf_peaks_oriented.bed: strand-assigned peaks with '+'/'-' strand labels
- Reported counts of peaks successfully assigned a strand versus peaks left unassigned

## How to apply

Load the unannotated CTCF ChIP-seq peak BED file and the strand-annotated MA0139.1 CTCF motif occurrence BED file (both on hg38) with pybedtools/pandas. For each unannotated peak interval (c, s, e), find all annotated motif intervals (c', s', e', r') that overlap it, i.e. sharing at least one coordinate k in [s', e') ∩ [s, e). When multiple motifs overlap a peak, retain only the motif minimizing the center-to-center distance |(s_i+e_i)/2 − (s'_i+e'_i)/2|, mirroring Asymmetron's 'bedtools closest' logic, and copy its strand sign to the peak. Emit oriented peaks as BED records to ctcf_peaks_oriented.bed, and report the count of peaks successfully assigned a strand versus those left unassigned. The rationale is that Asymmetron's fourth function performs strand assignment of an unassigned feature based on another overlapping feature, so oriented peaks can then feed the consecutive, contained, or pairwise asymmetry analyses.

## Related tools

- **Asymmetron** (Provides the orientation function that performs strand assignment of an unassigned feature based on another overlapping feature) — https://github.com/Ahituv-lab/Asymmetron
- **Pybedtools** (Implements the BED-formatted interval overlap used to find overlapping motif instances and the closest center distance)
- **Python** (Host language for the Asymmetron toolkit and the peak–motif overlap/orientation script)
- **pandas** (Loads and filters the peak and motif BED tables during orientation)

## Examples

```
import pybedtools as p; peaks = p.BedTool('ctcf_peaks.bed'); motifs = p.BedTool('ctcf_motifs_stranded.bed'); oriented = peaks.intersect(motifs, wa=True, wb=True)
```

## Evaluation signals

- Every peak in ctcf_peaks_oriented.bed that received a strand must overlap at least one annotated motif interval (no orientation without overlap)
- For peaks with multiple overlapping motifs, the retained motif must have the minimal |(s_i+e_i)/2 − (s'_i+e'_i)/2| center-distance among all overlapping motifs
- Oriented records must be valid 6-column BED entries with strand restricted to '+' or '-'
- The assignment versus unassignment counts must sum to the total number of input peaks
- Sanity check against Asymmetron's own orientation output or 'bedtools closest' on the same inputs should agree on the chosen motif for ambiguous peaks

## Limitations

- Peaks without any overlapping strand-annotated motif remain unassigned and cannot participate in downstream strand asymmetry analysis
- Asymmetron's core BED comparisons rely on pybedtools, which requires a conda environment (e.g. pybedtools with python 3.5, seaborn, numpy per the README); bedtools must be available on PATH
- Orientation is transferred, not measured: the peak strand is only as reliable as the underlying motif occurrence annotation (e.g. Unibind/JASPAR MA0139.1 maps)

## Evidence

- [methods] A fourth function performs the strand assignment of an unassigned feature based on another overlapping feature: "A fourth function performs the strand assignment of an unassigned feature based on another overlapping feature"
- [other] The orientation mechanism assigns each unannotated feature the strand sign of an overlapping annotated feature; when multiple overlaps occur, the sign of the annotated region whose center is closest to the un-annotated region's center is used.: [evidence span withheld — anchor status: not_found]
- [readme] Asymmetron can assign strand orientation to third features such as mutations, by orienting them relative to other genomic elements.: [evidence span withheld — anchor status: external_doc]
- [readme] The orientation of the CTCF motif dictates chromatin looping and three dimensional genome topology.: "The orientation of the CTCF motif dictates chromatin looping and three dimensional genome topology."
