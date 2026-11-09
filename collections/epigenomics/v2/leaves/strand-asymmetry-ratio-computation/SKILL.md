---
name: strand-asymmetry-ratio-computation
description: Use when input is strand-annotated BED files of DNA features (e.g. TFBS
  motifs, ChIP-seq peaks, transposable elements) and the goal is to quantify whether
  same/opposite or convergent/divergent strand orientations deviate from the random
  0.5 expectation.
when_to_use_negative:
- The feature lacks strand annotation — use Asymmetron's orientation function to assign
  strand first, or results will be meaningless.
- Features are not palindromic-definable or cannot be oriented relative to one another
  (non-palindromic orientation is required for same/opposite classification).
- The question is mutation-level compositional strand asymmetry relative to transcription/replication
  rather than orientation ratios between feature occurrences.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Asymmetron
  role: Primary toolkit providing consecutive_patterns, contained_asymmetries, and
    pairwise_asymmetries functions for computing strand asymmetry ratios
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: Pybedtools
  role: Backend for Asymmetron's core BED-formatted file comparison functions
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: bedtools
  role: Provides the 'bedtools closest' function used by pairwise_asymmetries to pair
    each element with its closest element within the distance range
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: scipy
  role: Two-tailed binomial test P-value calculation with expected probability 0.5
- name: pandas
  role: Statistical analysis and writing the asymmetry results table (e.g. .tsv output)
- name: numpy
  role: Numerical computation during statistical analysis
- name: matplotlib
  role: Visualization of same/opposite orientation counts as barplots
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkaa1052
    title: 'Asymmetron: a toolkit for the identification of strand asymmetry patterns
      in biological sequences'
  source_content_hash: sha256:ee2fa2b8bfe4c25bb2889e0e58f0ae7cceeff7e9e30510ef478449bbfcb5d64c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ahituv-lab/Asymmetron
  tools:
  - Asymmetron
  - Pybedtools
  - Python
  - bedtools
  - matplotlib
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# strand-asymmetry-ratio-computation

## Summary

Computes strand orientation asymmetry ratios for strand-annotated genomic features (BED files) using Asymmetron's contained_asymmetries and pairwise_asymmetries functions, with two-tailed binomial tests and Bonferroni correction. Grounded in the Asymmetron paper's analyses of TFBS orientation, transposable elements, and ChIP-seq peak orientation.

## When to use

Use when input is strand-annotated BED files of DNA features (e.g. TFBS motifs, ChIP-seq peaks, transposable elements) and the goal is to quantify whether same/opposite or convergent/divergent strand orientations deviate from the random 0.5 expectation.

## When NOT to use

- The feature lacks strand annotation — use Asymmetron's orientation function to assign strand first, or results will be meaningless.
- Features are not palindromic-definable or cannot be oriented relative to one another (non-palindromic orientation is required for same/opposite classification).
- The question is mutation-level compositional strand asymmetry relative to transcription/replication rather than orientation ratios between feature occurrences.

## Inputs

- Strand-annotated BED files (chromosome, start, end, strand) of genomic features, e.g. Unibind TFBS BED files, oriented CTCF ChIP-seq peak BED files, UCSC hg19/hg38 tfbsConsSites exports
- Two BED files of distinct motifs for pairwise_asymmetries (e.g. SP1_tfbsConsSites.bed and YY1_tfbsConsSites.bed)
- Optional: strand-assigned region BED files for contained_asymmetries (e.g. promoters, transcribed regions)

## Outputs

- Asymmetry ratio table (e.g. tfbs_chipseq_strand_asymmetries.tsv or pairwise_asymmetries_results.tsv) with same/opposite and convergent/divergent ratios, binomial P-values, and Bonferroni-corrected P-values
- Barplots of same versus opposite (or convergent versus divergent) orientation counts per comparison, rendered with matplotlib/seaborn

## How to apply

Load the strand-annotated BED files into Asymmetron, whose core BED comparison functions use pybedtools, and select the appropriate function: contained_asymmetries for motifs within strand-assigned regions (e.g. Unibind TFBS within oriented CTCF ChIP-seq peaks), pairwise_asymmetries for two motifs in proximity using 'bedtools closest' with a distance range such as --min_distance=1 --max_distance=200, or consecutive_patterns for consecutive occurrences of a single feature. Classify each overlapping or proximal pair as same orientation if the strand signs (+/−) match and opposite otherwise, subdividing opposite-orientation pairs into convergent or divergent based on the strand sign of the leftmost element. Compute each asymmetry ratio as occurrences-in-class divided by total occurrences (e.g. same / (same + opposite)), assuming a default expected asymmetry of 0.5 (random orientation). Calculate two-tailed binomial test P-values with the scipy package (expected probability 0.5) and apply Bonferroni correction when testing multiple features (e.g. the 551 TF PWMs that passed the P-value <10−6 motif threshold filter). Optionally run N permutation simulations (default N = 1000) with random '+'/'−' strand assignment for empirical P-values, and visualize same/opposite or convergent/divergent counts with matplotlib/seaborn barplots.

## Related tools

- **Asymmetron** (Primary toolkit providing consecutive_patterns, contained_asymmetries, and pairwise_asymmetries functions for computing strand asymmetry ratios) — https://github.com/Ahituv-lab/Asymmetron
- **Pybedtools** (Backend for Asymmetron's core BED-formatted file comparison functions) — https://github.com/Ahituv-lab/Asymmetron
- **bedtools** (Provides the 'bedtools closest' function used by pairwise_asymmetries to pair each element with its closest element within the distance range) — https://github.com/Ahituv-lab/Asymmetron
- **scipy** (Two-tailed binomial test P-value calculation with expected probability 0.5)
- **pandas** (Statistical analysis and writing the asymmetry results table (e.g. .tsv output))
- **numpy** (Numerical computation during statistical analysis)
- **matplotlib** (Visualization of same/opposite orientation counts as barplots)

## Examples

```
python pairwise_asymmetries.py SP1_tfbsConsSites.bed YY1_tfbsConsSites.bed --min_distance=1 --max_distance=200
```

## Evaluation signals

- The output TSV (e.g. pairwise_asymmetries_results.tsv) contains columns for same/opposite and convergent/divergent asymmetry ratios with associated binomial p-values and Bonferroni-corrected p-values.
- Each asymmetry ratio lies in [0, 1]; a ratio near 0.5 indicates no strand asymmetry (the default random-orientation expectation).
- For validation against the paper, the motif-level strand bias should be corroborated by ChIP-seq analysis — e.g. CTCF motif orientation within ChIP-seq peaks showed significant preference for the non-template strand in the promoter upstream region, with 194 out of 202 ChIP-seq experiments significant after Bonferroni correction.
- Permutation-based empirical P-values (default N = 1000 random '+'/'−' strand assignments) should be consistent with the analytic two-tailed binomial P-values.
- Strand signs in output classifications must exactly match input BED strand annotations (+/+ same, +/− or −/+ opposite; leftmost element's sign determines convergent vs divergent).

## Limitations

- The default expected asymmetry of 0.5 assumes random orientation; background strand preferences must be accounted for when interpreting mutational strand imbalances.
- pairwise_asymmetries pairs each element with only its closest neighbor ('bedtools closest') within the specified distance range, so distant or multiple-neighbor relationships are not evaluated.
- TFBS orientation preference may reflect preferential affinity for forward vs reverse-complement orientations or interaction/interference with RNA polymerase; the mechanism is not established.
- Motif scanning thresholds matter — the paper filtered PWMs to 551 with matches meeting a minimum FIMO P-value <10−6; weaker thresholds change which features are analyzable.

## Evidence

- [methods] strand asymmetry estimation of motifs within regions: "(ii) strand asymmetry estimation of strand-assigned motifs within strand-assigned regions"
- [methods] two-tailed binomial test and Bonferroni: "We then calculate the P-value for the two-tailed binomial test as follows"
- [methods] bedtools closest for pairwise: "we use the ‘bedtools closest’ function to determine the closest element in S2"
- [methods] Pybedtools backend: "its core BED-formatted file comparison functions use the package Pybedtools"
- [methods] scipy for P-values: "The corresponding P-values are calculated using the ‘scipy’ package in Python"
- [methods] permutation simulations: [evidence span withheld — anchor status: not_found]
- [results] CTCF ChIP-seq validation finding: "the orientation of the binding sites of 45% of human transcription factors analyzed have a significant DNA strand bias in transcribed regions"
- [readme] non-palindromic orientation requirement: "Sequences that are non palindromic can be oriented relative to one another."
