---
name: binomial-strand-orientation-testing
description: Use when you have BED files with '+'/'-' strand annotations for both
  motifs (e.g., TFBS occurrences) and regions (e.g., GENCODE transcribed regions)
  and need to quantify orientation bias per motif with statistical significance.
when_to_use_negative:
- The motif is palindromic (e.g., some restriction-site-like motifs) since it cannot
  be meaningfully oriented relative to the region strand
- Features lack strand annotation and no overlapping strand-assigned feature is available
  for orientation assignment (use Asymmetron's separate 'orientation' function first)
- You need empirical rather than binomial P-values — in that case use Asymmetron's
  permutation simulations (default N = 1000 random '+'/'-' assignments) instead
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0749
tools:
- name: Asymmetron
  role: Provides the contained_asymmetries function for strand asymmetry estimation
    of strand-assigned motifs within strand-assigned regions
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: Pybedtools
  role: Underlies Asymmetron's core BED-formatted file comparison (interval overlap)
    used to classify same/opposite strand orientation
- name: scipy
  role: Computes the two-tailed binomial test P-values for each motif's strand orientation
    counts
- name: pandas
  role: Tabulation of strand asymmetry scores, P-values and Bonferroni-corrected P-values
    per transcription factor
- name: numpy
  role: Numerical analysis of strand asymmetry statistics in Python
- name: FIMO
  role: Scans PWMs against the human genome using a background model of human nucleotide
    frequencies, with a minimum P-value < 10-6 threshold, to generate the TFBS occurrences
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkaa1052
    title: 'Asymmetron: a toolkit for the identification of strand asymmetry patterns
      in biological sequences'
  source_content_hash: sha256:08d37b5ed74307e619529360d76718acada726dd07ed943a78ebf1964c1e8ea8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ahituv-lab/Asymmetron
  tools:
  - Asymmetron
  - FIMO
  - Pybedtools
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# binomial-strand-orientation-testing

## Summary

Tests whether strand-annotated motifs (e.g., TFBSs) are non-randomly oriented relative to strand-annotated genomic regions (e.g., transcribed gene regions) using Asymmetron's contained_asymmetries overlap classification followed by a two-tailed binomial test with Bonferroni correction.

## When to use

Use when you have BED files with '+'/'-' strand annotations for both motifs (e.g., TFBS occurrences) and regions (e.g., GENCODE transcribed regions) and need to quantify orientation bias per motif with statistical significance.

## When NOT to use

- The motif is palindromic (e.g., some restriction-site-like motifs) since it cannot be meaningfully oriented relative to the region strand
- Features lack strand annotation and no overlapping strand-assigned feature is available for orientation assignment (use Asymmetron's separate 'orientation' function first)
- You need empirical rather than binomial P-values — in that case use Asymmetron's permutation simulations (default N = 1000 random '+'/'-' assignments) instead

## Inputs

- Strand-annotated BED file of regions (e.g., GENCODE v33 protein-coding gene regions, hg38)
- Strand-annotated BED file of motif occurrences (e.g., TFBSs from Unibind PWM motif maps 2019 release)
- Position weight matrices (e.g., JASPAR 2020 CORE vertebrates non-redundant) if motifs are scanned with FIMO (minimum P-value < 10-6)

## Outputs

- Per-motif table of strand asymmetry ratio (non-template / (template + non-template)), raw binomial P-value, and Bonferroni-corrected P-value
- Count/fraction of motifs with significant transcriptional strand bias (e.g., 248/551 = 45% of TF PWMs)

## How to apply

Prepare a strand-annotated BED file of regions (e.g., protein-coding genes from GENCODE v33 on hg38) and a strand-annotated BED file of motif occurrences (e.g., TFBSs from Unibind PWM motif maps). Run Asymmetron's contained_asymmetries function with regions as regions and motifs as motifs, using Pybedtools-based interval overlap to classify each overlapping motif/region pair as same or opposite strand orientation, with expected_asym = 0.5. Compute a strand asymmetry ratio per motif as occurrences on the non-template strand divided by total occurrences on template plus non-template strands. Calculate two-tailed binomial P-values with the scipy package in Python assuming P = 0.5, and adjust with Bonferroni correction for multiple tests across motifs. In the source analysis, 551 TF PWMs (from JASPAR 2020 CORE, FIMO-scanned with minimum P-value < 10-6) yielded 248 (45%) with significant transcriptional strand asymmetries at Bonferroni-corrected P < 0.05.

## Related tools

- **Asymmetron** (Provides the contained_asymmetries function for strand asymmetry estimation of strand-assigned motifs within strand-assigned regions) — https://github.com/Ahituv-lab/Asymmetron
- **Pybedtools** (Underlies Asymmetron's core BED-formatted file comparison (interval overlap) used to classify same/opposite strand orientation)
- **scipy** (Computes the two-tailed binomial test P-values for each motif's strand orientation counts)
- **pandas** (Tabulation of strand asymmetry scores, P-values and Bonferroni-corrected P-values per transcription factor)
- **numpy** (Numerical analysis of strand asymmetry statistics in Python)
- **FIMO** (Scans PWMs against the human genome using a background model of human nucleotide frequencies, with a minimum P-value < 10-6 threshold, to generate the TFBS occurrences)

## Examples

```
from asymmetron import contained_asymmetries; contained_asymmetries(gencode_genes_bed, tfbs_bed, expected_asym=0.5)
```

## Evaluation signals

- The fraction of motifs with significant Bonferroni-corrected strand bias should be reproducible; for the source task it must match the reported 45% (248 of 551 PWMs)
- Every overlapping motif/region pair must be classified as same or opposite strand with expected_asym = 0.5 (equal probability under the null)
- Raw binomial P-values must come from a two-tailed test with scipy and be Bonferroni-adjusted across all tested motifs (551 in the source analysis)
- Each TFBS occurrence counted must lie within a transcribed region and carry consistent '+'/'-' strand annotation in both BED inputs
- Motifs with no matches meeting the P-value threshold (e.g., FIMO P < 10-6) must be filtered out before testing, matching the 551-PWM denominator

## Limitations

- The binomial test assumes P = 0.5 (no strand preference); background strand composition biases in the input regions can confound the interpretation
- The mechanism behind TFBS orientation bias (preferential affinity for forward vs reverse-complement orientation or interference with RNA polymerase) is not established by the test itself
- Significant asymmetry detection depends on motif occurrence counts; weakly represented PWMs filtered out (no FIMO matches at P < 10-6) are excluded from the denominator
- Asymmetron is a Python toolkit (conda environment with pybedtools, python=3.5, seaborn, numpy per README) so BED tool dependencies and pybedtools availability are required

## Evidence

- [abstract] the orientation of the binding sites of 45% of human transcription factors analyzed have a significant DNA strand bias in transcribed regions: "the orientation of the binding sites of 45% of human transcription factors analyzed have a significant DNA strand bias in transcribed regions"
- [methods] strand asymmetry estimation of strand-assigned motifs within strand-assigned regions: "(ii) strand asymmetry estimation of strand-assigned motifs within strand-assigned regions"
- [methods] The corresponding P-values are calculated using the ‘scipy’ package in Python: "The corresponding P-values are calculated using the ‘scipy’ package in Python"
- [results] We filtered out transcription factor Position Weight Matrices (PWMs) for which there were no matches meeting the P-value threshold, resulting in 551 PWMs: "We filtered out transcription factor Position Weight Matrices (PWMs) for which there were no matches meeting the P-value threshold, resulting in 551 PWMs"
- [methods] its core BED-formatted file comparison functions use the package Pybedtools: "its core BED-formatted file comparison functions use the package Pybedtools"
