---
name: empirical-permutation-pvalue-calculation
description: Use when testing whether a strand asymmetry ratio (e.g. same vs opposite
  orientation of consecutive motif pairs) is statistically significant and a closed-form
  binomial test needs empirical confirmation via strand-label permutations.
when_to_use_negative:
- When features are palindromic or strand cannot be assigned — no orientation information
  exists to permute
- When the question concerns compositional or mutational strand bias already covered
  by the binomial test rather than orientation of consecutive/paired features
- When a very small N (e.g. 100) is used but fine-grained P-values below ~0.01 are
  required, since resolution is bounded by 1/N
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Asymmetron
  role: Runs the permutation simulations that randomly assign '+'/'-' strand labels
    with fixed +/− counts and derives empirical P-values for strand asymmetries
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: scipy
  role: Computes the two-tailed binomial test P-values that complement the empirical
    permutation P-values
- name: numpy
  role: Supports the statistical analysis of permutation results in Python
- name: pandas
  role: Handles the tabular strand asymmetry results used in significance testing
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkaa1052
    title: 'Asymmetron: a toolkit for the identification of strand asymmetry patterns
      in biological sequences'
  source_content_hash: sha256:c9509f5549dd60b414be1bf7914cc31786416a1f0ad48a2029405fc3f8428f41
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ahituv-lab/Asymmetron
  tools:
  - Asymmetron
  - Pybedtools
  - Python
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# empirical-permutation-pvalue-calculation

## Summary

Asymmetron's permutation-based approach for computing empirical two-tailed P-values of strand asymmetry: N simulations (default 1000, task card used 100) randomly permute '+'/'-' strand labels while keeping the +/− counts fixed, and the observed asymmetry ratio is compared against this null distribution.

## When to use

Use when testing whether a strand asymmetry ratio (e.g. same vs opposite orientation of consecutive motif pairs) is statistically significant and a closed-form binomial test needs empirical confirmation via strand-label permutations.

## When NOT to use

- When features are palindromic or strand cannot be assigned — no orientation information exists to permute
- When the question concerns compositional or mutational strand bias already covered by the binomial test rather than orientation of consecutive/paired features
- When a very small N (e.g. 100) is used but fine-grained P-values below ~0.01 are required, since resolution is bounded by 1/N

## Inputs

- Strand-annotated BED file of genomic features (e.g. CTCF MA0139.1 motif occurrences with chromosome, start, end, strand)
- Observed strand asymmetry ratio r = m/(m+o) from consecutive_patterns, pairwise_asymmetries, or contained_asymmetries
- Number of simulations N (default 1000; --simulations parameter)

## Outputs

- Empirical two-tailed P-value per asymmetry pattern
- Null distribution of asymmetry ratios from N permuted datasets
- Bonferroni-adjusted P-values when multiple tests are performed
- TSV table of strand asymmetries (e.g. ctcf_consecutive_asymmetries.tsv) with significance calls

## How to apply

After Asymmetron computes the observed strand asymmetry ratio r = m/(m+o) for a pattern (e.g. consecutive CTCF motif occurrences from a strand-annotated BED file within --max_distance 100), run N simulations (default N = 1000; the CTCF task used --simulations 100) in which a '+' or '-' value is randomly assigned to every feature while preserving the total +/− counts. Recompute the asymmetry ratio for each permuted dataset to build the null distribution, then derive the empirical two-tailed P-value as the fraction of simulated ratios at least as extreme as the observed one. Complement this with the two-tailed binomial test computed via the scipy package, and apply Bonferroni correction when testing multiple patterns (e.g. many TF PWMs from the JASPAR 2020 non-redundant CORE set). Rationale: permutation preserves the marginal strand composition so the empirical P-value reflects orientation structure rather than strand-count imbalance.

## Related tools

- **Asymmetron** (Runs the permutation simulations that randomly assign '+'/'-' strand labels with fixed +/− counts and derives empirical P-values for strand asymmetries) — https://github.com/Ahituv-lab/Asymmetron
- **scipy** (Computes the two-tailed binomial test P-values that complement the empirical permutation P-values)
- **numpy** (Supports the statistical analysis of permutation results in Python)
- **pandas** (Handles the tabular strand asymmetry results used in significance testing)

## Examples

```
python consecutive_patterns.py --input ctcf_MA0139.1.bed --max_distance 100 --min_distance 0 --simulations 100 --output ctcf_consecutive_asymmetries.tsv
```

## Evaluation signals

- The empirical P-value should agree in direction with the two-tailed binomial test computed via scipy for the same asymmetry ratio
- With default N = 1000 simulations, the minimum attainable empirical P-value is bounded by ~1/1000; P-values should respect this resolution
- For the CTCF consecutive-occurrence validation, the same/opposite strand asymmetry ratio must exceed 0.5 with empirical P-value < 0.001 (using --max_distance 100 on MA0139.1 BED)
- After Bonferroni correction, significant patterns (e.g. LINE/SINE/LTR template-strand ratios 0.392/0.471/0.316) should remain Bonferroni corrected P < 0.001
- Permutation must preserve the total +/− strand counts of the input BED file; verify permuted datasets have identical strand-label marginals

## Limitations

- Empirical P-value resolution is limited by the number of simulations (default 1000); the CTCF task used only 100 simulations, bounding significance at P ≈ 0.01
- Empirical P-values assume the null of random strand assignment with fixed +/− counts and may not capture genomic confounders such as distance-to-TSS effects, which the article showed modulate asymmetry strength for LINEs and SINEs
- Multiple testing across many PWMs (551 tested after filtering) requires Bonferroni correction, which can mask weaker but real asymmetries

## Evidence

- [methods] permutation simulations for empirical P-values: [evidence span withheld — anchor status: not_found]
- [methods] binomial test complement: "We then calculate the P-value for the two-tailed binomial test as follows"
- [methods] Bonferroni correction: "and are adjusted with Bonferroni correction in case of multiple tests."
- [methods] scipy for P-values: "The corresponding P-values are calculated using the ‘scipy’ package in Python"
- [abstract] Asymmetron input/output scope: "Asymmetron takes as input DNA features provided with strand annotation and outputs strand asymmetries for consecutive occurrences of a single DNA feature or between pairs of features."
- [readme] CTCF non-palindromic orientation: "An exemplar case reflecting the contribution of strand asymmetries in biological processes is the transcription factor CCCTC-binding factor (CTCF)."
