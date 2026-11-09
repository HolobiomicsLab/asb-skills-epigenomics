---
name: chip-seq-peak-strand-orientation-analysis
description: Use when strand-annotated ChIP-seq peak BED files and strand-assigned
  TFBS BED files are available and the goal is to test whether motif-level strand
  asymmetry (e.g. CTCF non-template strand preference) is recapitulated within peaks.
when_to_use_negative:
- The peak or TFBS features are not strand-annotated and no independent orientation
  source exists — use Asymmetron's orientation function first or a different approach
- The question concerns consecutive occurrences of a single element or pairs of motifs,
  not motifs contained within regions — use consecutive_patterns or pairwise_asymmetries
  instead
- Motifs are palindromic, so +/− orientation is uninformative (e.g. some motif classes)
  and same/opposite classification is meaningless
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: Asymmetron
  role: Provides the contained_asymmetries function that estimates strand asymmetry
    of strand-assigned motifs within strand-assigned regions (TFBS within ChIP-seq
    peaks)
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: Pybedtools
  role: Underlies Asymmetron's core BED-formatted file comparison functions used to
    intersect motif and peak BED files
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: scipy
  role: Computes two-tailed binomial test P-values (p = 0.5) for the same/opposite
    orientation counts
- name: pandas
  role: Statistical analysis and writing of the TFBS/ChIP-seq strand asymmetry table
- name: matplotlib
  role: Visualization of same versus opposite orientation count barplots (with seaborn)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkaa1052
    title: 'Asymmetron: a toolkit for the identification of strand asymmetry patterns
      in biological sequences'
  source_content_hash: sha256:898552a02efa91645b6190efa0ded9402eecf869413e00d668ff497c167fc296
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ahituv-lab/Asymmetron
  tools:
  - Asymmetron
  - Pybedtools
  - Python
  - matplotlib
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# chip-seq-peak-strand-orientation-analysis

## Summary

A validation skill that tests whether transcription factor binding site (TFBS) orientation biases are corroborated by an independent analysis of TF orientation within strand-annotated ChIP-seq peaks, using Asymmetron's contained_asymmetries function and two-tailed binomial tests with Bonferroni correction.

## When to use

Use when strand-annotated ChIP-seq peak BED files and strand-assigned TFBS BED files are available and the goal is to test whether motif-level strand asymmetry (e.g. CTCF non-template strand preference) is recapitulated within peaks.

## When NOT to use

- The peak or TFBS features are not strand-annotated and no independent orientation source exists — use Asymmetron's orientation function first or a different approach
- The question concerns consecutive occurrences of a single element or pairs of motifs, not motifs contained within regions — use consecutive_patterns or pairwise_asymmetries instead
- Motifs are palindromic, so +/− orientation is uninformative (e.g. some motif classes) and same/opposite classification is meaningless

## Inputs

- Oriented ChIP-seq peak BED file with strand annotation (e.g. CTCF peaks, used as regions)
- Strand-assigned TFBS BED files from Unibind PWM motif maps (2019 release), used as motifs
- Optional: reference genome hg38 and GENCODE v33 annotation for promoter/transcribed region context

## Outputs

- Per-TFBS-set same/opposite strand asymmetry ratio table (e.g. tfbs_chipseq_strand_asymmetries.tsv)
- Two-tailed binomial test P-values (p = 0.5) with Bonferroni-adjusted values per TFBS set
- Barplots of same versus opposite orientation counts per TFBS comparison (matplotlib/seaborn)

## How to apply

Load the oriented ChIP-seq peak BED file (regions) and the TFBS BED files (motifs, e.g. Unibind 2019 release PWM motif maps) into Asymmetron, whose core BED comparison functions use Pybedtools. Run the contained_asymmetries function with the oriented peaks as regions and the TFBS files as motifs, using the default expected asymmetry of 0.5 (random orientation assumption). For each overlapping motif-peak pair, classify it as same strand orientation if the strand signs (+/−) match and opposite otherwise, then compute the same/opposite asymmetry ratio as same divided by same plus opposite for each TFBS set. Calculate two-tailed binomial test P-values with scipy (p = 0.5) and adjust them with Bonferroni correction for the multiple TF comparisons. Generate barplots of same versus opposite orientation counts with matplotlib/seaborn and write the results to an asymmetry table (e.g. tfbs_chipseq_strand_asymmetries.tsv). The rationale is that corroboration between motif-level asymmetry in transcribed regions and ChIP-seq peak orientation (e.g. 45% of TFs with significant strand bias) strengthens the biological validity of the pattern.

## Related tools

- **Asymmetron** (Provides the contained_asymmetries function that estimates strand asymmetry of strand-assigned motifs within strand-assigned regions (TFBS within ChIP-seq peaks)) — https://github.com/Ahituv-lab/Asymmetron
- **Pybedtools** (Underlies Asymmetron's core BED-formatted file comparison functions used to intersect motif and peak BED files) — https://github.com/Ahituv-lab/Asymmetron
- **scipy** (Computes two-tailed binomial test P-values (p = 0.5) for the same/opposite orientation counts)
- **pandas** (Statistical analysis and writing of the TFBS/ChIP-seq strand asymmetry table)
- **matplotlib** (Visualization of same versus opposite orientation count barplots (with seaborn))

## Examples

```
git clone https://github.com/Ahituv-lab/Asymmetron; conda create --name asymmetron pybedtools python=3.5 seaborn numpy; conda activate asymmetron
```

## Evaluation signals

- The asymmetry ratio (same / (same + opposite)) falls between 0 and 1, with 0.5 corresponding to random orientation under the default expected asymmetry
- Statistically significant results replicate across independent ChIP-seq experiments: in the source analysis, 194 out of 202 CTCF ChIP-seq experiments showed significant orientation preference after Bonferroni correction
- The written TSV table contains one row per TFBS set with counts, ratios, raw and Bonferroni-corrected P-values
- Significant ChIP-seq peak orientation biases agree in direction with the motif-level transcriptional strand asymmetries in transcribed regions (45% of analyzed TFs showed significant strand bias)
- Barplots show same versus opposite orientation counts matching the counts in the TSV output

## Limitations

- The test assumes random orientation (expected asymmetry 0.5); deviations from this baseline assumption could inflate apparent asymmetries
- The biological mechanism of TFBS orientation preference is not established — it could reflect preferential affinity for forward versus reverse-complement motifs or interaction/interference with RNA polymerase
- Strong asymmetry patterns are motif-dependent; palindromic or weakly oriented motifs cannot yield meaningful same/opposite classifications

## Evidence

- [abstract] the orientation of the binding sites of 45% of human transcription factors analyzed have a significant DNA strand bias in transcribed regions, that is also corroborated in ChIP-seq analyses: "the orientation of the binding sites of 45% of human transcription factors analyzed have a significant DNA strand bias in transcribed regions, that is also corroborated in ChIP-seq analyses"
- [methods] its core BED-formatted file comparison functions use the package Pybedtools: "its core BED-formatted file comparison functions use the package Pybedtools"
- [methods] Statistical analysis was performed in Python with the packages ‘math’, ‘scipy’, ‘pandas’ and ‘numpy’: "Statistical analysis was performed in Python with the packages ‘math’, ‘scipy’, ‘pandas’ and ‘numpy’"
- [methods] strand asymmetry estimation of strand-assigned motifs within strand-assigned regions: "(ii) strand asymmetry estimation of strand-assigned motifs within strand-assigned regions"
- [supplementary] A Python implementation package can be found in GitHub: https://github.com/Ahituv-lab/Asymmetron.: "A Python implementation package can be found in GitHub: https://github.com/Ahituv-lab/Asymmetron."
