---
name: proximal-motif-pairing-with-bedtools-closest
description: Use when two strand-annotated BED files of genomic elements (e.g. two
  transcription factor motif sets such as SP1 and YY1 tfbsConsSites) must be paired
  at a distance range to test strand orientation preferences.
when_to_use_negative:
- You need strand asymmetries within consecutive occurrences of a single element rather
  than between two different elements — use Asymmetron's consecutive_patterns function
  instead.
- You need strand asymmetry of a strand-assigned motif within strand-assigned regions
  (e.g. motif orientation relative to transcription) — use Asymmetron's contained_asymmetries
  function instead.
- Input features lack strand annotation entirely and must first be oriented — use
  Asymmetron's 'orientation' function to assign strands before pairing.
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: Asymmetron
  role: Provides the pairwise_asymmetries function that implements proximal motif
    pairing and strand asymmetry estimation between two strand-assigned motifs
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: Pybedtools
  role: Python wrapper through which Asymmetron's core BED-formatted file comparison
    functions are executed
- name: bedtools
  role: Supplies the 'bedtools closest' function used to determine the closest element
    in the second BED file within the distance range
- name: scipy
  role: Calculates the two-tailed binomial test P-values for the same/opposite and
    convergent/divergent orientation ratios
- name: Python
  role: Language in which the toolkit and its statistical analysis (math, scipy, pandas,
    numpy) are implemented
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkaa1052
    title: 'Asymmetron: a toolkit for the identification of strand asymmetry patterns
      in biological sequences'
  source_content_hash: sha256:0b8c004f88fc90d21b9850247d3f90cd50c4b00d8092bec0e76dee5d4de5d0c0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ahituv-lab/Asymmetron
  tools:
  - Asymmetron
  - Pybedtools
  - Python
  - bedtools
  - scipy
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# proximal-motif-pairing-with-bedtools-closest

## Summary

Uses bedtools 'closest' (via Pybedtools, as implemented in Asymmetron's pairwise_asymmetries) to pair each strand-annotated motif of one BED file with its closest element in a second BED file within a user-specified distance range, enabling downstream same/opposite and convergent/divergent strand asymmetry estimation between two motifs (e.g. SP1 and YY1 conserved TFBSs at 1-200 bp).

## When to use

Use when two strand-annotated BED files of genomic elements (e.g. two transcription factor motif sets such as SP1 and YY1 tfbsConsSites) must be paired at a distance range to test strand orientation preferences.

## When NOT to use

- You need strand asymmetries within consecutive occurrences of a single element rather than between two different elements — use Asymmetron's consecutive_patterns function instead.
- You need strand asymmetry of a strand-assigned motif within strand-assigned regions (e.g. motif orientation relative to transcription) — use Asymmetron's contained_asymmetries function instead.
- Input features lack strand annotation entirely and must first be oriented — use Asymmetron's 'orientation' function to assign strands before pairing.

## Inputs

- Strand-annotated BED file of the first motif's binding sites (e.g. SP1_tfbsConsSites.bed with chromosome, start, end, strand)
- Strand-annotated BED file of the second motif's binding sites (e.g. YY1_tfbsConsSites.bed)
- User-specified minimum and maximum pairwise distance bounds (e.g. 1 and 200 bp)

## Outputs

- pairwise_asymmetries_results.tsv containing same/opposite and convergent/divergent asymmetry ratios with associated binomial P-values for the motif-pair comparison

## How to apply

Extract strand-annotated coordinates (chromosome, start, end, strand) for the two motif sets, e.g. SP1 and YY1 conserved binding sites from the UCSC hg19 tfbsConsSites table, into two BED files. Run Asymmetron's pairwise_asymmetries with the two BED files and distance bounds (e.g. --min_distance=1 --max_distance=200); internally the tool uses the 'bedtools closest' function via Pybedtools to pair each element of the first file with its closest element in the second within that range. Classify each proximal pair as same orientation (equal strand signs) or opposite orientation (unequal strand signs), subdividing opposite-orientation pairs into convergent or divergent based on the strand sign of the leftmost element. Compute same/opposite and convergent/divergent asymmetry ratios as occurrences-in-class divided by total occurrences, then evaluate two-tailed binomial P-values (expected probability 0.5) with the scipy package, applying Bonferroni correction for multiple tests.

## Related tools

- **Asymmetron** (Provides the pairwise_asymmetries function that implements proximal motif pairing and strand asymmetry estimation between two strand-assigned motifs) — https://github.com/Ahituv-lab/Asymmetron
- **Pybedtools** (Python wrapper through which Asymmetron's core BED-formatted file comparison functions are executed)
- **bedtools** (Supplies the 'bedtools closest' function used to determine the closest element in the second BED file within the distance range)
- **scipy** (Calculates the two-tailed binomial test P-values for the same/opposite and convergent/divergent orientation ratios)
- **Python** (Language in which the toolkit and its statistical analysis (math, scipy, pandas, numpy) are implemented)

## Examples

```
python pairwise_asymmetries.py SP1_tfbsConsSites.bed YY1_tfbsConsSites.bed --min_distance=1 --max_distance=200
```

## Evaluation signals

- Output table pairwise_asymmetries_results.tsv exists and contains same/opposite and convergent/divergent asymmetry ratio columns for the SP1–YY1 comparison.
- Each reported asymmetry ratio falls in [0, 1], since ratios are occurrences-in-class divided by total occurrences.
- Each ratio is accompanied by a two-tailed binomial P-value (expected probability 0.5) that has been Bonferroni-corrected when multiple distance bins are tested.
- All reported pairs lie within the user-specified distance range (--min_distance=1 to --max_distance=200 bp) as enforced by the bedtools closest pairing step.
- Every pair is unambiguously classified: same orientation iff strand signs are equal; opposite-orientation pairs subdivided into convergent or divergent using the strand sign of the leftmost element.

## Limitations

- Pairing is restricted to the closest element in the second file within the user-specified distance range, so one-to-many or bidirectional pairing at these distances is not captured.
- The two-tailed binomial test assumes expected probability 0.5 under the null; background strand preferences of the motifs themselves are not corrected in this pairwise function.
- Convergent/divergent classification of opposite-orientation pairs depends on the strand sign of the leftmost element, so unsorted or inconsistent strand annotation in input BED files will corrupt orientation classes.
- The repository README instructs installation in a conda environment with pybedtools and python=3.5, so Python version compatibility may be an issue in modern environments.

## Evidence

- [methods] we use the ‘bedtools closest’ function to determine the closest element in S2: "we use the ‘bedtools closest’ function to determine the closest element in S2"
- [methods] strand asymmetry estimation between pairs of motifs: "(iii) strand asymmetry estimation between two strand-assigned motifs in proximity or overlapping each other."
- [methods] core BED-formatted file comparison functions use Pybedtools: "its core BED-formatted file comparison functions use the package Pybedtools"
- [methods] P-values calculated with scipy and Bonferroni correction: "The corresponding P-values are calculated using the ‘scipy’ package in Python"
- [supplementary] tutorial command from source card workflow: "python pairwise_asymmetries.py SP1_tfbsConsSites.bed YY1_tfbsConsSites.bed --min_distance=1 --max_distance=200"
