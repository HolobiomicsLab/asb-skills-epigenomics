---
name: center-distance-feature-matching
description: Use when a BED feature file (e.g. CTCF ChIP-seq peaks on hg38) lacks
  strand annotation but an overlapping strand-assigned reference file (e.g. MA0139.1
  CTCF motif occurrences) exists, and downstream strand-asymmetry analysis requires
  oriented features.
when_to_use_negative:
- The features already carry strand annotation — apply Asymmetron's consecutive/contained/pairwise
  asymmetry functions directly instead of orienting.
- No overlapping strand-assigned reference feature exists for the input coordinates
  — the orientation function depends on overlap and cannot assign a strand by proximity
  alone.
- The reference motifs are palindromic or otherwise strand-symmetric — the transferred
  sign carries no directional information for asymmetry analysis.
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Asymmetron
  role: Provides the 'orientation' function that performs strand assignment of an
    unassigned feature based on another overlapping feature
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: Pybedtools
  role: Used for interval overlap detection between the unannotated BED features and
    the strand-annotated BED reference; core of Asymmetron's BED comparison functions
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: pandas
  role: Used in Python to load and manipulate the BED feature tables when computing
    center distances
- name: bedtools
  role: Underlying 'closest' logic (determine the closest element in the annotated
    set) that the nearest-center tie-breaking mirrors
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkaa1052
    title: 'Asymmetron: a toolkit for the identification of strand asymmetry patterns
      in biological sequences'
  source_content_hash: sha256:7b03644d3a2633c9db1a146c9e25ae10dbe75df7ac5d26b902da92954b29cd69
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Center-distance-feature-matching

## Summary

Assigns a strand sign ('+' or '-') to an unannotated genomic feature (e.g. a CTCF ChIP-seq peak) using the strand sign of an overlapping annotated feature (e.g. a JASPAR/Unibind CTCF motif occurrence); when multiple annotated features overlap, the one whose center is closest to the unannotated feature's center wins. This is the Asymmetron 'orientation' function, implemented on BED files via Pybedtools.

## When to use

Use when a BED feature file (e.g. CTCF ChIP-seq peaks on hg38) lacks strand annotation but an overlapping strand-assigned reference file (e.g. MA0139.1 CTCF motif occurrences) exists, and downstream strand-asymmetry analysis requires oriented features.

## When NOT to use

- The features already carry strand annotation — apply Asymmetron's consecutive/contained/pairwise asymmetry functions directly instead of orienting.
- No overlapping strand-assigned reference feature exists for the input coordinates — the orientation function depends on overlap and cannot assign a strand by proximity alone.
- The reference motifs are palindromic or otherwise strand-symmetric — the transferred sign carries no directional information for asymmetry analysis.

## Inputs

- Unannotated genomic feature BED file (e.g. CTCF ChIP-seq peaks, hg38)
- Strand-annotated reference BED file of overlapping features (e.g. MA0139.1 CTCF motif occurrences from JASPAR 2020 / Unibind PWM maps, hg38)

## Outputs

- Strand-assigned peaks written to a BED file (e.g. ctcf_peaks_oriented.bed) with '+'/'-' strand column filled
- Count of peaks successfully assigned a strand versus peaks left unassigned

## How to apply

Load the unannotated peak BED file and the strand-annotated motif occurrence BED file (both hg38) with pybedtools/pandas. For each unannotated peak vector (c, s, e), find every annotated motif vector (c', s', e', r') that overlaps it, i.e. for which some coordinate k lies in both [s, e) and [s', e'). When multiple overlaps exist, retain the motif instance minimizing the distance between peak and motif centers, |(s_i + e_i)/2 - (s'_i + e'_i)/2| (the same 'closest element' logic Asymmetron implements with Pybedtools and the bedtools closest concept). Set the peak's strand sign equal to the retained motif's sign and emit the oriented peak as a BED record. The rationale is that non-palindromic motifs such as CTCF (whose motif orientation dictates chromatin looping) carry strand information that can be transferred to overlapping unannotated features.

## Related tools

- **Asymmetron** (Provides the 'orientation' function that performs strand assignment of an unassigned feature based on another overlapping feature) — https://github.com/Ahituv-lab/Asymmetron
- **Pybedtools** (Used for interval overlap detection between the unannotated BED features and the strand-annotated BED reference; core of Asymmetron's BED comparison functions) — https://github.com/Ahituv-lab/Asymmetron
- **pandas** (Used in Python to load and manipulate the BED feature tables when computing center distances)
- **bedtools** (Underlying 'closest' logic (determine the closest element in the annotated set) that the nearest-center tie-breaking mirrors)

## Examples

```
best = min(overlaps, key=lambda m: abs((p.start+p.end)/2 - (m.start+m.end)/2)); p.strand = best.strand
```

## Evaluation signals

- Every oriented peak's strand sign equals the strand sign of the retained overlapping motif instance.
- Among overlapping motif instances, the retained one has the minimal |(s_i + e_i)/2 - (s'_i + e'_i)/2| center distance.
- Output BED has valid 6-column BED schema with strand column populated as '+' or '-' for assigned records.
- The report of assigned vs unassigned peaks is consistent: peaks with no overlapping motif remain unassigned.
- Count of output records equals count of input records (orientation adds a strand attribute, it does not drop or duplicate features).

## Limitations

- Features without any overlapping annotated feature cannot be oriented and are left unassigned.
- If multiple motifs overlap, only the nearest-center one contributes a sign; orientation of more distant overlapping motifs is discarded.
- The transferred orientation is only as correct as the reference motif annotation; motif orientation reflects PWM strand, not necessarily the bound TF orientation.

## Evidence

- [methods] A fourth function performs the strand assignment of an unassigned feature based on another overlapping feature: "A fourth function performs the strand assignment of an unassigned feature based on another overlapping feature"
- [methods] closest element via bedtools: [evidence span withheld — anchor status: not_found]
- [methods] core comparison uses Pybedtools: "It is a Python-based toolkit and its core BED-formatted file comparison functions use the package Pybedtools"
- [readme] README orientation capability: [evidence span withheld — anchor status: external_doc]
