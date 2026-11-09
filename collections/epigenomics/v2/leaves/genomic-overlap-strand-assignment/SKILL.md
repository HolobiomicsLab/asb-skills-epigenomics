---
name: genomic-overlap-strand-assignment
description: Use when a BED feature set lacks strand annotation but an overlapping
  strand-assigned feature set exists (e.g., orienting hg38 CTCF ChIP-seq peaks using
  MA0139.1 CTCF motif occurrences) prior to strand asymmetry analysis.
when_to_use_negative:
- Input features already carry strand annotation — apply Asymmetron's asymmetry functions
  (consecutive_patterns, contained_asymmetries, pairwise_asymmetries) directly instead
- The goal is computing strand asymmetry statistics or P-values rather than orienting
  features
- No overlapping strand-annotated reference features exist for the genomic regions
  of interest — peaks will remain unassigned
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Asymmetron
  role: Provides the orientation function that performs strand assignment of an unassigned
    feature based on an overlapping feature
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: Pybedtools
  role: Implements Asymmetron's core BED-formatted interval overlap/comparison used
    for the nearest-center overlap assignment
- name: bedtools
  role: Underlying 'bedtools closest' logic determines the closest overlapping element
    when multiple overlaps occur
- name: Python
  role: Language of the Asymmetron toolkit and the orientation implementation
- name: pandas
  role: Used for handling BED interval tables during the overlap and center-distance
    computation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkaa1052
    title: 'Asymmetron: a toolkit for the identification of strand asymmetry patterns
      in biological sequences'
  source_content_hash: sha256:983fcb424d1cc29c28f7395f7efffd55bf4071f825e7517c82440cd4dbf0df6e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ahituv-lab/Asymmetron
  tools:
  - Asymmetron
  - Pybedtools
  - Python
  - bedtools
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# genomic-overlap-strand-assignment

## Summary

Asymmetron's orientation function assigns a strand sign ('+' or '-') to unannotated genomic features (e.g., CTCF ChIP-seq peaks) by transferring the strand of an overlapping strand-annotated feature (e.g., JASPAR/Unibind CTCF motif occurrences), enabling subsequent strand asymmetry analysis.

## When to use

Use when a BED feature set lacks strand annotation but an overlapping strand-assigned feature set exists (e.g., orienting hg38 CTCF ChIP-seq peaks using MA0139.1 CTCF motif occurrences) prior to strand asymmetry analysis.

## When NOT to use

- Input features already carry strand annotation — apply Asymmetron's asymmetry functions (consecutive_patterns, contained_asymmetries, pairwise_asymmetries) directly instead
- The goal is computing strand asymmetry statistics or P-values rather than orienting features
- No overlapping strand-annotated reference features exist for the genomic regions of interest — peaks will remain unassigned

## Inputs

- BED file of unannotated genomic features (e.g., CTCF ChIP-seq peaks, hg38 coordinates)
- BED file of strand-annotated reference features (e.g., MA0139.1 CTCF motif occurrences from JASPAR 2020 / Unibind PWM maps, hg38)

## Outputs

- Strand-assigned BED file of oriented features (e.g., ctcf_peaks_oriented.bed)
- Count of peaks successfully assigned a strand versus peaks left unassigned

## How to apply

Load the unannotated feature BED file (e.g., CTCF ChIP-seq peaks) and the strand-annotated motif BED file (e.g., MA0139.1 CTCF motif occurrences on hg38) with pybedtools/pandas. For each unannotated interval (s, e), find every annotated interval (s', e', r') that overlaps it, i.e. sharing at least one coordinate k in [s', e') ∩ [s, e). When multiple overlapping motifs exist, retain only the instance minimizing the center-to-center distance |(s+e)/2 - (s'+e')/2|, matching the 'bedtools closest' behavior used by Asymmetron. Assign the peak the retained motif's strand sign and write oriented records (e.g., to ctcf_peaks_oriented.bed) via the Asymmetron orientation function, reporting counts of assigned versus unassigned peaks. Rationale: Asymmetron requires strand-annotated input, so this step orients third features such as mutations or peaks relative to other genomic elements before running asymmetry estimation.

## Related tools

- **Asymmetron** (Provides the orientation function that performs strand assignment of an unassigned feature based on an overlapping feature) — https://github.com/Ahituv-lab/Asymmetron
- **Pybedtools** (Implements Asymmetron's core BED-formatted interval overlap/comparison used for the nearest-center overlap assignment)
- **bedtools** (Underlying 'bedtools closest' logic determines the closest overlapping element when multiple overlaps occur)
- **Python** (Language of the Asymmetron toolkit and the orientation implementation)
- **pandas** (Used for handling BED interval tables during the overlap and center-distance computation)

## Examples

```
conda activate asymmetron; python -c "import pybedtools; peaks=pybedtools.BedTool('ctcf_peaks.bed'); motifs=pybedtools.BedTool('ctcf_motifs.bed'); peaks.intersect(motifs, wa=True, wb=True)"
```

## Evaluation signals

- Every assigned peak carries the strand sign ('+' or '-') of the retained nearest-center overlapping motif, never of a farther overlapping motif
- Center-distance invariant: for each assigned peak, |(s+e)/2 - (s'+e')/2| of the chosen motif is minimal among all overlapping motifs
- Output BED record count plus the count of unassigned peaks equals the input peak count (no peaks dropped or duplicated)
- Output file (e.g., ctcf_peaks_oriented.bed) is a valid BED with strand column populated only where an overlap existed
- Orientation result matches Asymmetron's Pybedtools-based overlap implementation on a held-out subset of peaks

## Limitations

- Peaks with no overlapping strand-annotated feature remain unassigned — the method relies entirely on overlap, not proximity beyond overlap
- When multiple motifs overlap, only the nearest-center one is used; conflicting strand signs from other overlapping motifs are discarded
- The transfer assumes the overlapping annotated feature's strand is informative for the unannotated feature (e.g., CTCF motif orientation reflects ChIP-seq peak orientation); README notes Conda environment built with python=3.5/pybedtools dependencies

## Evidence

- [methods] A fourth function performs the strand assignment of an unassigned feature based on another overlapping feature: "A fourth function performs the strand assignment of an unassigned feature based on another overlapping feature"
- [methods] bedtools closest for nearest element: "we use the ‘bedtools closest’ function to determine the closest element in S2"
- [methods] core BED comparison via Pybedtools: "its core BED-formatted file comparison functions use the package Pybedtools"
- [readme] README: orienting third features relative to other genomic elements: [evidence span withheld — anchor status: external_doc]
- [readme] README: CTCF motif orientation example: "An exemplar case reflecting the contribution of strand asymmetries in biological processes is the transcription factor CCCTC-binding factor (CTCF)."
