---
name: bed-coordinate-processing
description: Use when a BED file of genomic intervals (e.g. CTCF ChIP-seq peaks on
  hg38) lacks strand annotation and strand signs must be inherited from an overlapping
  strand-annotated feature file such as MA0139.1 CTCF motif occurrences.
when_to_use_negative:
- Intervals already carry strand annotation — skip orientation and run Asymmetron's
  consecutive_patterns, contained_asymmetries or pairwise_asymmetries directly
- No strand-annotated reference feature overlaps the unannotated intervals, so no
  strand sign can be inherited and peaks would remain unassigned
- The goal is de novo motif discovery or scanning (FIMO with the minimum P-value <10−6
  threshold) rather than strand inheritance from existing annotations
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0102
tools:
- name: Asymmetron
  role: Provides the orientation function that assigns each unannotated BED feature
    the strand sign of the nearest-center overlapping annotated feature
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: Pybedtools
  role: Implements the interval-overlap comparisons that enumerate every motif instance
    overlapping an unannotated peak before nearest-center selection
- name: bedtools
  role: Its 'bedtools closest' function determines the closest annotated element,
    reproducing the nearest-center strand inheritance
- name: Python
  role: Host language of the orientation implementation, loading peak and motif BED
    files with pandas/pybedtools and writing oriented records
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkaa1052
    title: 'Asymmetron: a toolkit for the identification of strand asymmetry patterns
      in biological sequences'
  source_content_hash: sha256:933f1c630773f115b6018411f1fc8602c7fdfd5b8cd6da17683861d7a6b4baf4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ahituv-lab/Asymmetron
  tools:
  - Asymmetron
  - Pybedtools
  - Python
  - bedtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# bed-coordinate-processing

## Summary

Reconstructs Asymmetron's orientation function: unannotated genomic intervals (e.g. CTCF ChIP-seq peaks on hg38) inherit the strand sign of the overlapping strand-annotated feature (e.g. MA0139.1 CTCF motif occurrences) whose center is nearest, enabling downstream strand asymmetry analysis.

## When to use

Use when a BED file of genomic intervals (e.g. CTCF ChIP-seq peaks on hg38) lacks strand annotation and strand signs must be inherited from an overlapping strand-annotated feature file such as MA0139.1 CTCF motif occurrences.

## When NOT to use

- Intervals already carry strand annotation — skip orientation and run Asymmetron's consecutive_patterns, contained_asymmetries or pairwise_asymmetries directly
- No strand-annotated reference feature overlaps the unannotated intervals, so no strand sign can be inherited and peaks would remain unassigned
- The goal is de novo motif discovery or scanning (FIMO with the minimum P-value <10−6 threshold) rather than strand inheritance from existing annotations

## Inputs

- Unannotated BED file of genomic intervals without strand (e.g. CTCF ChIP-seq peak BED on hg38)
- Strand-annotated BED file of reference features carrying '+'/'-' signs (e.g. MA0139.1 CTCF motif occurrence BED derived from JASPAR 2020 PWMs on hg38)

## Outputs

- Strand-assigned BED file (e.g. ctcf_peaks_oriented.bed) where each peak inherits the strand sign of its nearest-center overlapping motif instance
- Assignment summary: number of peaks successfully assigned a strand versus peaks left unassigned

## How to apply

Load the unannotated peak BED (e.g. CTCF ChIP-seq peaks on hg38) and the strand-annotated motif BED (e.g. MA0139.1 CTCF motif occurrences from JASPAR 2020 / Unibind PWM maps) with pybedtools/pandas, since Asymmetron's core BED-formatted file comparison functions use Pybedtools. For each unannotated interval (s, e), find every annotated interval (s', e') that overlaps it, i.e. some coordinate k lies in both [s, e) and [s', e'). Decision point: when several motif instances overlap one peak, keep only the instance minimizing the center distance |(s_i+e_i)/2 - (s'_i+e'_i)/2| (nearest-center rule; the same closest-element selection can be reproduced with 'bedtools closest'). Copy the retained motif's strand sign to the peak and emit oriented records to a BED file (e.g. ctcf_peaks_oriented.bed), reporting the number of peaks assigned a strand versus peaks left unassigned. Rationale: Asymmetron's asymmetry analyses (consecutive_patterns, contained_asymmetries, pairwise_asymmetries) require strand annotation, so the orientation filter step must first assign strand from an overlapping strand-assigned feature.

## Related tools

- **Asymmetron** (Provides the orientation function that assigns each unannotated BED feature the strand sign of the nearest-center overlapping annotated feature) — https://github.com/Ahituv-lab/Asymmetron
- **Pybedtools** (Implements the interval-overlap comparisons that enumerate every motif instance overlapping an unannotated peak before nearest-center selection)
- **bedtools** (Its 'bedtools closest' function determines the closest annotated element, reproducing the nearest-center strand inheritance)
- **Python** (Host language of the orientation implementation, loading peak and motif BED files with pandas/pybedtools and writing oriented records)

## Examples

```
python -c "import pybedtools; peaks=pybedtools.BedTool('ctcf_peaks.bed'); motifs=pybedtools.BedTool('ctcf_motifs_hg38.bed'); print(peaks.intersect(motifs, wa=True, wb=True))"
```

## Evaluation signals

- Every peak record in ctcf_peaks_oriented.bed carries a strand sign equal to that of an annotated motif instance overlapping the peak interval
- For peaks with multiple overlapping motifs, the assigned strand equals the sign of the motif with minimal |(s_i+e_i)/2 - (s'_i+e'_i)/2|; recomputing center distances per peak must reproduce the call
- Chrom, start and end columns of oriented peaks are unchanged from the input BED — only the strand field differs
- assigned_count + unassigned_count equals the number of input peak rows, and both counts are reported as the workflow requires
- Cross-checking with 'bedtools closest' restricted to overlapping elements yields the same strand assignment for every peak

## Limitations

- Peaks with no overlapping annotated motif cannot inherit a strand and remain unassigned; completeness of the motif occurrence file (e.g. JASPAR 2020 / Unibind CTCF instances on hg38) directly bounds the assignable fraction
- Only the single nearest-center overlapping motif contributes a strand sign, so near-ties in center distance make assignments sensitive to motif annotation noise or file version
- The inherited orientation is biologically meaningful only where feature orientation is coupled to the reference feature (as with CTCF motif orientation within CTCF ChIP peaks); it does not infer directionality de novo

## Evidence

- [methods] Orientation function assigns strand from an overlapping feature: "A fourth function performs the strand assignment of an unassigned feature based on another overlapping feature"
- [methods] Closest-element selection via bedtools closest: "we use the ‘bedtools closest’ function to determine the closest element in S2"
- [methods] Pybedtools implements BED file comparisons: "its core BED-formatted file comparison functions use the package Pybedtools"
- [readme] Strand assignment of third features relative to genomic elements: [evidence span withheld — anchor status: external_doc]
- [abstract] Asymmetron requires strand-annotated input features: "Asymmetron takes as input DNA features provided with strand annotation and outputs strand asymmetries for consecutive occurrences of a single DNA feature or between pairs of features"
