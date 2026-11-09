---
name: empirical-null-hypothesis-testing-by-random-bisection
description: Use when you need an empirical P-value for a clustering-agreement metric
  (VI) between a hierarchical partition and benchmark annotations, and a parametric
  null is unavailable.
when_to_use_negative:
- When a parametric or analytic null distribution for the agreement metric is available
  and appropriate
- When the alternative clustering does not assume a nested hierarchy — the null model
  specifically mimics BHi-Cect's recursive bisection structure
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: BHi-Cect
  role: Produces the enclave partition whose VI against TAD/compartment benchmarks
    is tested against the random-bisection null
  repo: https://github.com/princeps091-binf/BHi-Cect
- name: R
  role: Environment in which the VI computation, null partition generation, and P-value
    derivation are implemented
- name: dplyr
  role: Used for data manipulation when summarizing null distributions and writing
    results
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkaa004
    title: 'BHi-Cect: a top-down algorithm for identifying the multi-scale hierarchical
      structure of chromosomes'
  source_content_hash: sha256:e01c1bce046e2446466d3aaafeffeb702aabfadd602d7ea71fd64a7181908ca0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/princeps091-binf/BHi-Cect
  tools:
  - BHi-Cect
  - R
  - RSpectra
  - caret
  - dplyr
  - igraph
  - readr
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# Empirical null hypothesis testing by random recursive bisection

## Summary

Assesses whether an observed clustering agreement statistic (e.g. variation of information between BHi-Cect enclaves and TAD/compartment calls) is significant by building an empirical null distribution from 500 random recursive bisection partitions that preserve the nested-hierarchy assumption of BHi-Cect.

## When to use

Use when you need an empirical P-value for a clustering-agreement metric (VI) between a hierarchical partition and benchmark annotations, and a parametric null is unavailable.

## When NOT to use

- When a parametric or analytic null distribution for the agreement metric is available and appropriate
- When the alternative clustering does not assume a nested hierarchy — the null model specifically mimics BHi-Cect's recursive bisection structure

## Inputs

- BHi-Cect enclave partition of chromosome loci (named list of loci per cluster)
- Rao et al. TAD and compartment benchmark cluster annotations for the same loci
- 500 random recursive bisection null partitions of the loci

## Outputs

- Observed VI values between enclave partition and TAD/compartment clusterings
- Empirical null distribution of VI values from 500 random recursive bisections
- Empirical P-value for the observed VI
- vi_significance_results.csv containing observed VI, null distribution summary, and empirical P-value

## How to apply

First compute the observed variation of information (VI) between the BHi-Cect enclave partition of loci and the Rao et al. TAD/compartment benchmark clusterings. Then generate 500 random partitions that, like BHi-Cect, assume a nested hierarchy of smaller clusters within larger clusters (random recursive bisections of the loci), computing VI for each against the same benchmarks. Build the empirical null distribution from these 500 VI values and derive the empirical P-value for the observed VI. Accept the result if the empirical P-value falls below the 0.01 significance threshold, confirming that enclave clustering agrees with TAD/compartment calls significantly better than the hierarchy-preserving null. Write the observed VI values, null distribution summary, and empirical P-value to vi_significance_results.csv.

## Related tools

- **BHi-Cect** (Produces the enclave partition whose VI against TAD/compartment benchmarks is tested against the random-bisection null) — https://github.com/princeps091-binf/BHi-Cect
- **R** (Environment in which the VI computation, null partition generation, and P-value derivation are implemented)
- **dplyr** (Used for data manipulation when summarizing null distributions and writing results)

## Examples

```
chr_spec_res <- spec_bipart(chr_pow, g_chr1); observed_vi <- vi(chr_spec_res$clusters, rao_tads); p_emp <- mean(null_vi_500 >= observed_vi)
```

## Evaluation signals

- Empirical P-value from the 500 random recursive bisections is below the 0.01 significance threshold
- The null partitions preserve the nested-hierarchy assumption (smaller clusters nested within larger clusters) as BHi-Cect does
- The observed VI is consistently lower (better agreement) than the bulk of the 500 null VI values
- vi_significance_results.csv contains observed VI values, null distribution summary, and empirical P-value

## Limitations

- The null model is specific to hierarchically nested partitions; it tests significance relative to random recursive bisections, not to arbitrary clusterings
- The P-value is empirical and bounded by the number of null replicates (500), so resolution below ~0.01 is not achievable without more nulls

## Evidence

- [results] The agreement between the enclave clustering and TAD/compartment benchmark clusterings was statistically significant by the variation of information metric, with an empirical P-value below 0.01.: [evidence span withheld — anchor status: not_found]
- [methods] Generate 500 random recursive bisection partitions that, like BHi-Cect, assume a nested hierarchy of smaller clusters within larger clusters, and compute VI for each null partition.: [evidence span withheld — anchor status: not_found]
- [readme] BHi-Cect only assumes that chromosome architecture consists of a nested hierarchy where we expect to find smaller-sized clusters nested within larger-sized clusters.: "It only assumes that chromosome architecture consists of a nested hierarchy where we expect to find smaller-sized clusters nested within larger-sized clusters."
- [methods] Write the observed VI values, null distribution summary, and empirical P-value to vi_significance_results.csv.: [evidence span withheld — anchor status: not_found]
