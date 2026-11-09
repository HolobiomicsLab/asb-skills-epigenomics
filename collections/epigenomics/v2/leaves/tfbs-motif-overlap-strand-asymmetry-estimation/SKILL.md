---
name: tfbs-motif-overlap-strand-asymmetry-estimation
description: Use when both a regions BED file and a motifs BED file carry strand annotation
  ('+'/'-') and the goal is to test whether motif orientation is significantly biased
  relative to region orientation, e.g. TFBS orientation within CTCF ChIP-seq peaks
  or relative to transcriptional direction.
when_to_use_negative:
- When comparing consecutive occurrences of a single feature within one file rather
  than motifs contained in regions — use Asymmetron's consecutive_patterns function
  instead
- When the features are unannotated for strand — use Asymmetron's orientation function
  to assign strand based on an overlapping feature before testing asymmetry
- When the interest is proximity relationships between two motifs rather than containment
  — use pairwise_asymmetries instead
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0749
tools:
- name: Asymmetron
  role: Provides the contained_asymmetries function used to estimate strand asymmetry
    of motifs within strand-assigned regions
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: Pybedtools
  role: Underlies Asymmetron's core BED-formatted file comparison functions (interval
    overlap between motifs and regions)
- name: scipy
  role: Two-tailed binomial test P-value calculation assuming P = 0.5
- name: pandas
  role: Tabulation of asymmetry scores and corrected P-values to TSV output
- name: numpy
  role: Numerical computation of strand asymmetry ratios
- name: matplotlib
  role: Barplots of same versus opposite orientation counts (with seaborn)
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkaa1052
    title: 'Asymmetron: a toolkit for the identification of strand asymmetry patterns
      in biological sequences'
  source_content_hash: sha256:3887ef74fbea7ced55ade8bbaed984181274a218b37bbc590d87b08ef3d3c380
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
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# tfbs-motif-overlap-strand-asymmetry-estimation

## Summary

Uses Asymmetron's contained_asymmetries function to quantify whether strand-annotated motifs (e.g. TFBSs) are biased toward the same or opposite strand of overlapping strand-assigned regions (e.g. ChIP-seq peaks, transcribed gene regions), with two-tailed binomial testing and Bonferroni correction.

## When to use

Use when both a regions BED file and a motifs BED file carry strand annotation ('+'/'-') and the goal is to test whether motif orientation is significantly biased relative to region orientation, e.g. TFBS orientation within CTCF ChIP-seq peaks or relative to transcriptional direction.

## When NOT to use

- When comparing consecutive occurrences of a single feature within one file rather than motifs contained in regions — use Asymmetron's consecutive_patterns function instead
- When the features are unannotated for strand — use Asymmetron's orientation function to assign strand based on an overlapping feature before testing asymmetry
- When the interest is proximity relationships between two motifs rather than containment — use pairwise_asymmetries instead

## Inputs

- Strand-annotated regions BED file (e.g. oriented CTCF ChIP-seq peaks or GENCODE v33 hg38 protein-coding gene regions)
- Strand-annotated motifs BED file(s) (e.g. Unibind PWM motif maps 2019 release, or JASPAR 2020 CORE FIMO scan results with P < 10^-6 threshold)
- expected_asym parameter value (default 0.5, random orientation assumption)

## Outputs

- Per-motif strand asymmetry table (same/opposite counts, ratio, two-tailed binomial P-value, Bonferroni-corrected P-value), e.g. tfbs_chipseq_strand_asymmetries.tsv
- Barplots of same versus opposite orientation counts per comparison (matplotlib/seaborn)
- Count or fraction of motifs with significant strand bias (e.g. 248/551 PWMs, 45%)

## How to apply

Load the strand-annotated regions BED (e.g. oriented CTCF ChIP-seq peaks or GENCODE v33 hg38 protein-coding gene regions) and the strand-annotated motifs BED (e.g. Unibind 2019 TFBS maps or JASPAR 2020 FIMO scans with minimum P-value <10^-6) into Asymmetron, whose core BED comparison functions use pybedtools. Run contained_asymmetries with expected_asym = 0.5 (random orientation assumption) so that each overlapping motif/region pair is classified as same strand if the '+'/'-' signs match and opposite otherwise. Compute the asymmetry ratio as same occurrences divided by same plus opposite occurrences (e.g. non-template/template strand ratio). Calculate two-tailed binomial test P-values with the scipy package in Python assuming P = 0.5, and adjust with Bonferroni correction for the multiple TF comparisons. Tabulate ratio, P-value and corrected P-value per motif set (e.g. to TSV via pandas) and visualize same-versus-opposite counts with matplotlib/seaborn.

## Related tools

- **Asymmetron** (Provides the contained_asymmetries function used to estimate strand asymmetry of motifs within strand-assigned regions) — https://github.com/Ahituv-lab/Asymmetron
- **Pybedtools** (Underlies Asymmetron's core BED-formatted file comparison functions (interval overlap between motifs and regions))
- **scipy** (Two-tailed binomial test P-value calculation assuming P = 0.5)
- **pandas** (Tabulation of asymmetry scores and corrected P-values to TSV output)
- **numpy** (Numerical computation of strand asymmetry ratios)
- **matplotlib** (Barplots of same versus opposite orientation counts (with seaborn))

## Examples

```
conda activate asymmetron; python -c "from Asymmetron import contained_asymmetries; contained_asymmetries(regions='ctcf_peaks.bed', motifs='tfbs.bed', expected_asym=0.5)"
```

## Evaluation signals

- Binomial P-values with Bonferroni correction should be < 0.05 for motifs with genuine bias; 248 of 551 TF PWMs (~45%) reached significance in transcribed regions
- CTCF validation: 194 out of 202 ChIP-seq experiments showed statistically significant orientation preference after multiple testing correction
- Same/opposite strand ratio for unbiased motifs should be near 0.5 (expected_asym = 0.5); e.g. LINEs/SINEs/LTRs template-strand ratios of 0.392/0.471/0.316 had Bonferroni-corrected P < 0.001
- Output table must contain per-TF rows with asymmetry score, raw P-value, and Bonferroni-corrected P-value columns
- Asymmetron's permutation option (N simulations, default N = 1000) can provide empirical P-values to cross-check the binomial results

## Limitations

- The default binomial test assumes expected asymmetry of 0.5 (random orientation); deviations from this null assumption require permutation-based empirical P-values
- Motif scanning with FIMO uses a minimum P-value < 10^-6 threshold and PWMs with no matches meeting the threshold are filtered out (leaving 551 of the original PWMs), so results depend on motif-scanning sensitivity
- The mechanism of TFBS orientation bias (preferential motif affinity for forward vs reverse-complement orientation, or interaction/interference with RNA polymerase) is not established by the analysis

## Evidence

- [methods] Asymmetron enables versatile genomic investigations of strand asymmetry patterns across different biological problems. It is a Python-based toolkit: "Asymmetron enables versatile genomic investigations of strand asymmetry patterns across different biological problems. It is a Python-based toolkit"
- [methods] its core BED-formatted file comparison functions use the package Pybedtools: "its core BED-formatted file comparison functions use the package Pybedtools"
- [methods] The corresponding P-values are calculated using the ‘scipy’ package in Python: "The corresponding P-values are calculated using the ‘scipy’ package in Python"
- [abstract] the orientation of the binding sites of 45% of human transcription factors analyzed have a significant DNA strand bias in transcribed regions, that is also corroborated in ChIP-seq analyses: "the orientation of the binding sites of 45% of human transcription factors analyzed have a significant DNA strand bias in transcribed regions, that is also corroborated in ChIP-seq analyses, and is"
- [supplementary] A Python implementation package can be found in GitHub: "A Python implementation package can be found in GitHub: https://github.com/Ahituv-lab/Asymmetron."
