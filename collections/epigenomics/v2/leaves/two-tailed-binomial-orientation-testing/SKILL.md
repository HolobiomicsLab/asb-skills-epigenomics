---
name: two-tailed-binomial-orientation-testing
description: Use when oriented genomic features (e.g. strand-annotated TFBS BED files)
  must be tested for significant same/opposite strand preference relative to a null
  expectation of 0.5 across many feature sets.
when_to_use_negative:
- Features are not strand-annotated or cannot be oriented relative to the reference
  regions — the binomial counts of same vs opposite strands cannot be formed (use
  Asymmetron's 'orientation' function first to assign strands).
- The null hypothesis is not a symmetric 0.5 same/opposite expectation, e.g. when
  testing against a background strand-composition bias rather than random orientation.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Asymmetron
  role: Toolkit that produces the same/opposite strand asymmetry counts and expected
    asymmetry (0.5) fed into the binomial test
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: scipy
  role: Package used in Python to calculate the two-tailed binomial test P-values
- name: pandas
  role: Used for statistical analysis and assembly of the asymmetry table in Python
- name: numpy
  role: Used with math/scipy/pandas for the statistical analysis in Python
- name: matplotlib
  role: Used with seaborn for visualizations (barplots of same vs opposite orientation
    counts)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkaa1052
    title: 'Asymmetron: a toolkit for the identification of strand asymmetry patterns
      in biological sequences'
  source_content_hash: sha256:bb52a38fbda702071e698e1e7275bf8ff42c89de2ceb6b7d1fc31b774a448ce3
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# two-tailed-binomial-orientation-testing

## Summary

Statistical testing of strand orientation asymmetry for DNA features: same/opposite strand counts are compared against the random-expectation ratio of 0.5 with a two-tailed binomial test (scipy), and P-values are adjusted with Bonferroni correction when multiple features are tested.

## When to use

Use when oriented genomic features (e.g. strand-annotated TFBS BED files) must be tested for significant same/opposite strand preference relative to a null expectation of 0.5 across many feature sets.

## When NOT to use

- Features are not strand-annotated or cannot be oriented relative to the reference regions — the binomial counts of same vs opposite strands cannot be formed (use Asymmetron's 'orientation' function first to assign strands).
- The null hypothesis is not a symmetric 0.5 same/opposite expectation, e.g. when testing against a background strand-composition bias rather than random orientation.

## Inputs

- Table of same-strand and opposite-strand occurrence counts per feature set (e.g. tfbs_chipseq_strand_asymmetries.tsv pre-statistics)
- Asymmetry ratios computed by Asymmetron's contained_asymmetries function (default expected asymmetry 0.5)
- Number of tested feature sets for Bonferroni correction (e.g. TFBS sets from Unibind PWM motif maps, 2019 release)

## Outputs

- Per-feature two-tailed binomial test P-values (scipy, p = 0.5)
- Bonferroni-adjusted P-values for the multiple TF comparisons
- Asymmetry table (same/opposite counts, ratio, corrected P) written to tfbs_chipseq_strand_asymmetries.tsv

## How to apply

After classifying each overlapping motif–region pair as same strand (matching +/− signs) or opposite strand, compute the asymmetry ratio as same / (same + opposite) per feature set. Calculate two-tailed binomial test P-values using the scipy package with the default probability p = 0.5, which encodes the random-orientation null hypothesis used by Asymmetron (e.g. expected asymmetry of 0.5 in the contained_asymmetries function). When testing many feature sets (e.g. 551 TF PWMs or multiple ChIP-seq experiments), adjust P-values with Bonferroni correction to control the family-wise error rate. Report the asymmetry ratio and corrected P-value per feature, and flag features with corrected P < 0.05 (the article used corrected P < 0.001 for retrotransposon calls) as significantly strand-biased.

## Related tools

- **Asymmetron** (Toolkit that produces the same/opposite strand asymmetry counts and expected asymmetry (0.5) fed into the binomial test) — https://github.com/Ahituv-lab/Asymmetron
- **scipy** (Package used in Python to calculate the two-tailed binomial test P-values)
- **pandas** (Used for statistical analysis and assembly of the asymmetry table in Python)
- **numpy** (Used with math/scipy/pandas for the statistical analysis in Python)
- **matplotlib** (Used with seaborn for visualizations (barplots of same vs opposite orientation counts))

## Examples

```
from scipy import stats; p = 2 * stats.binom.cdf(min(same, opposite), n=same+opposite, p=0.5); p_adj = min(p * n_tests, 1.0)
```

## Evaluation signals

- Each feature's same/(same+opposite) ratio must lie in (0, 1), with 0.5 equal to the random-orientation expectation.
- For the CTCF ChIP-seq validation, 194 out of 202 ChIP-seq experiments should show statistically significant orientation preference after Bonferroni correction — a large deviation from this indicates a pipeline error.
- Motif-level corroboration: ~45% of analyzed transcription factors should show significant DNA strand bias in transcribed regions under this test.
- Sanity check: a symmetric synthetic input with equal same/opposite counts must yield a non-significant P-value (≈1) at p = 0.5.
- Bonferroni-adjusted P-values must be monotonically ≥ the raw P-values and capped at 1.

## Limitations

- The default p = 0.5 assumes random orientation; true background strand preferences (e.g. compositional strand asymmetries) can confound the null, and the README notes correction for background strand preferences is needed when interpreting mutational strand imbalance.
- Bonferroni correction is conservative for the large number of tested TFs/PWMs (551 PWMs passed the P-value < 10^-6 threshold), potentially missing weak asymmetries.
- Asymmetry estimates depend on the number of occurrences; sparse feature sets yield low binomial power, and the toolkit otherwise offers N = 1000 permutation simulations for empirical P-values as an alternative.

## Evidence

- [methods] We then calculate the P-value for the two-tailed binomial test as follows: "We then calculate the P-value for the two-tailed binomial test as follows"
- [methods] and are adjusted with Bonferroni correction in case of multiple tests.: "and are adjusted with Bonferroni correction in case of multiple tests."
- [methods] The corresponding P-values are calculated using the ‘scipy’ package in Python: "The corresponding P-values are calculated using the ‘scipy’ package in Python"
- [results] ratios 0.392, 0.471, 0.316, respectively, binomial test, Bonferroni corrected P-values < 0.001: "We found a preference for LINEs, SINEs and LTRs to be at the template strand (ratios 0.392, 0.471, 0.316, respectively, binomial test, Bonferroni corrected P-values < 0.001)"
- [abstract] the orientation of the binding sites of 45% of human transcription factors analyzed have a significant DNA strand bias: "the orientation of the binding sites of 45% of human transcription factors analyzed have a significant DNA strand bias in transcribed regions, that is also corroborated in ChIP-seq analyses"
