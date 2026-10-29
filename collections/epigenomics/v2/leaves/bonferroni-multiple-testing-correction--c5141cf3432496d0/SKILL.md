---
name: bonferroni-multiple-testing-correction
description: Use when a strand asymmetry analysis produces many P-values from repeated
  two-tailed binomial tests (p = 0.5 expected asymmetry) and you need Bonferroni-corrected
  significance for each feature (TF PWM, motif, repeat class, ChIP-seq peak set).
when_to_use_negative:
- Only a single asymmetry comparison is being run — Bonferroni is unnecessary and
  needlessly conservative (Asymmetron applies it 'in case of multiple tests')
- The analysis calls for an empirical/permutation-based null instead — Asymmetron
  separately supports N = 1000 random-strand simulations for empirical P-values, which
  is a distinct procedure
- P-values are already adjusted with a different multiple-testing method; re-correcting
  would double-penalize
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Asymmetron
  role: Toolkit whose asymmetry functions (contained_asymmetries, consecutive_patterns)
    generate the per-feature binomial P-values that are subsequently Bonferroni-adjusted
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: scipy
  role: Computes the two-tailed binomial test P-values (p = 0.5) for each asymmetry
    comparison before correction
- name: pandas
  role: Tabulates strand asymmetry scores, raw and Bonferroni-corrected P-values per
    transcription factor/experiment in Python
- name: numpy
  role: Numerical support for the Python statistical analysis alongside math, scipy
    and pandas
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_006
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkaa1052
    title: 'Asymmetron: a toolkit for the identification of strand asymmetry patterns
      in biological sequences'
  source_content_hash: sha256:d8aa07f3a0b50d521b06d91613be8a977755e2e41016769358fb058ea1ab9694
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
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# bonferroni-multiple-testing-correction

## Summary

Bonferroni adjustment of two-tailed binomial test P-values across the many strand-asymmetry comparisons performed by Asymmetron (e.g., dozens of TF PWMs, ChIP-seq experiments, or repeat classes), so that reported orientation biases like the 45% of TFs with significant DNA strand bias survive multiple-testing correction.

## When to use

Use when a strand asymmetry analysis produces many P-values from repeated two-tailed binomial tests (p = 0.5 expected asymmetry) and you need Bonferroni-corrected significance for each feature (TF PWM, motif, repeat class, ChIP-seq peak set).

## When NOT to use

- Only a single asymmetry comparison is being run — Bonferroni is unnecessary and needlessly conservative (Asymmetron applies it 'in case of multiple tests')
- The analysis calls for an empirical/permutation-based null instead — Asymmetron separately supports N = 1000 random-strand simulations for empirical P-values, which is a distinct procedure
- P-values are already adjusted with a different multiple-testing method; re-correcting would double-penalize

## Inputs

- Table of raw two-tailed binomial test P-values, one per feature (TF PWM, PolyASite cluster set, repeat class, or ChIP-seq experiment), from Asymmetron asymmetry comparisons
- Same/opposite strand orientation counts per feature (computed via Pybedtools interval overlap with expected_asym = 0.5)
- Number of simultaneous tests (count of PWMs, experiments, or region sets)

## Outputs

- Asymmetry table (e.g., tfbs_chipseq_strand_asymmetries.tsv) with strand asymmetry score, raw P-value and Bonferroni-corrected P-value per feature
- Count/fraction of features with significant corrected bias (e.g., 248/551 = 45% of TFs; 194/202 ChIP-seq experiments)
- Bonferroni-corrected significance calls at a stated threshold (P < 0.05 or P < 0.001)

## How to apply

After running Asymmetron's contained_asymmetries or consecutive_patterns functions, compute the same/opposite strand asymmetry ratio (same occurrences divided by same plus opposite occurrences) for each feature and calculate its two-tailed binomial test P-value with the scipy package in Python, assuming P = 0.5. Multiply each raw P-value by the number of tests (e.g., 551 TF PWMs for JASPAR-derived TFBSs, 202 CTCF ChIP-seq experiments, or the number of repeat classes) and cap at 1.0 — this is how Asymmetron adjusts 'with Bonferroni correction in case of multiple tests'. Perform the statistical analysis in Python with 'math', 'scipy', 'pandas' and 'numpy' as in the paper. Declare significance using a corrected threshold (the paper uses Bonferroni-corrected P < 0.001 for repeat classes and P < 0.05 for TF PWMs). Report, per feature, the strand asymmetry ratio, raw P-value and Bonferroni-corrected P-value so the fraction of significant features (e.g., 248 of 551 TFs, ~45%) can be reproduced.

## Related tools

- **Asymmetron** (Toolkit whose asymmetry functions (contained_asymmetries, consecutive_patterns) generate the per-feature binomial P-values that are subsequently Bonferroni-adjusted) — https://github.com/Ahituv-lab/Asymmetron
- **scipy** (Computes the two-tailed binomial test P-values (p = 0.5) for each asymmetry comparison before correction)
- **pandas** (Tabulates strand asymmetry scores, raw and Bonferroni-corrected P-values per transcription factor/experiment in Python)
- **numpy** (Numerical support for the Python statistical analysis alongside math, scipy and pandas)

## Examples

```
from scipy import stats; p = stats.binom_test(same, same + opposite, 0.5); p_bonf = min(p * n_tests, 1.0)
```

## Evaluation signals

- Corrected P-values equal min(raw P-value × number_of_tests, 1.0) and are monotonically consistent with raw P-values
- Significance counts match reported anchors: 45% (248/551) of TF PWMs significant at Bonferroni-corrected P < 0.05; 194/202 CTCF ChIP-seq experiments significant; SINE ratio 0.471 significant at corrected P < 0.001
- Reported strand asymmetry ratios are unchanged by correction (Bonferroni affects only P-values, e.g., 0.72 for polyadenylation signals, 0.471 for SINEs)
- Output table contains, for every feature, exactly one raw and one corrected P-value with no missing entries

## Limitations

- Bonferroni is conservative; features with moderate raw P-values can lose significance when the test count is large (e.g., 551 PWMs), so corrected and uncorrected significance fractions may diverge
- The correction applies to the family of binomial tests actually run; results depend on which features were retained after filtering (e.g., PWMs filtered out for having no matches meeting the minimum P-value < 1e-6 threshold, leaving 551 PWMs)
- Correction does not address the p = 0.5 null assumption itself — the expected same-strand probability must still be justified for each comparison

## Evidence

- [methods] Bonferroni correction for multiple tests is applied to P-values: "and are adjusted with Bonferroni correction in case of multiple tests."
- [methods] P-values calculated with scipy in Python: "The corresponding P-values are calculated using the ‘scipy’ package in Python"
- [methods] Statistical analysis performed in Python with specific packages: "Statistical analysis was performed in Python with the packages ‘math’, ‘scipy’, ‘pandas’ and ‘numpy’"
- [results] Repeat class asymmetries reported with Bonferroni-corrected P-values: "ratios 0.392, 0.471, 0.316, respectively, binomial test, Bonferroni corrected P-values < 0.001"
- [abstract] 45% of transcription factors show significant strand bias after multiple testing correction: "the orientation of the binding sites of 45% of human transcription factors analyzed have a significant DNA strand bias in transcribed regions"
