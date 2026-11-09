---
name: hardy-weinberg-genotype-simulation
description: Use when a power or association analysis needs simulated genotype covariates
  for a reference sample set — e.g. scPower's simulation eQTL power for small-mean
  genes requires HWE-drawn SNP genotypes paired with negative binomial pseudobulk
  counts.
when_to_use_negative:
- Analytic power estimation is sufficient — scPower's own analytic eQTL power is the
  default; simulation (and hence HWE genotype sampling) is reserved for genes with
  small pseudobulk mean expression where the analytic linear-model approximation is
  inaccurate
- The target cohort has known population stratification, relatedness, or non-random
  mating that violates Hardy-Weinberg assumptions
- Extremely large eQTL cohorts (hundreds of samples) are needed — the article notes
  such data sets 'will be very difficult to simulate' due to runtime and memory costs
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scPower
  role: R package whose simulation-based eQTL power routine consumes HWE-simulated
    SNP genotypes as regression covariates for small-mean genes
  repo: https://github.com/heiniglab/scPower
- name: HardyWeinberg
  role: CRAN dependency installed alongside scPower, used for Hardy-Weinberg equilibrium
    genotype generation/analysis
- name: R
  role: Runtime environment in which the genotype simulation and downstream scPower
    power calculations are executed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-26779-7
    title: scPower accelerates and optimizes the design of multi-sample single cell
      transcriptomic studies
  source_content_hash: sha256:f179260904103069b818954dacf01f86edfb4b9443967d70e8ae974860c22b42
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/heiniglab/scPower
  tools:
  - HardyWeinberg
  - R
  - scPower
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# hardy-weinberg-genotype-simulation

## Summary

Generates synthetic SNP genotype data under Hardy-Weinberg equilibrium for reference samples, as used by scPower to supply genotypes for its simulation-based eQTL power estimation in genes whose pseudobulk mean expression is too small for the analytic linear-model approximation.

## When to use

Use when a power or association analysis needs simulated genotype covariates for a reference sample set — e.g. scPower's simulation eQTL power for small-mean genes requires HWE-drawn SNP genotypes paired with negative binomial pseudobulk counts.

## When NOT to use

- Analytic power estimation is sufficient — scPower's own analytic eQTL power is the default; simulation (and hence HWE genotype sampling) is reserved for genes with small pseudobulk mean expression where the analytic linear-model approximation is inaccurate
- The target cohort has known population stratification, relatedness, or non-random mating that violates Hardy-Weinberg assumptions
- Extremely large eQTL cohorts (hundreds of samples) are needed — the article notes such data sets 'will be very difficult to simulate' due to runtime and memory costs

## Inputs

- SNP allele frequencies or genotype distributions for the reference sample (e.g. from eqtl.ref.study, Blueprint Monocytes)
- Sample size of the in silico cohort
- Number of SNPs/variants to simulate

## Outputs

- Simulated genotype matrix (individuals x SNPs) with counts following Hardy-Weinberg proportions (0/1/2 minor alleles)
- Genotype vector used per gene as covariate in the linear regression of log-transformed pseudobulk counts

## How to apply

Draw SNP genotypes for the reference sample under Hardy-Weinberg equilibrium (the scPower R setup installs the 'HardyWeinberg' CRAN package for this), so that genotype frequencies follow p^2:2pq:q^2 at each SNP given an allele frequency. These HWE genotypes serve as the independent variable when, per eQTL gene, negative binomial pseudobulk counts are drawn from scPower's mean/dispersion priors (e.g. eqtl.ref.study from Blueprint Monocytes, read.umi.fit '10X_PBMC_1', gamma.mixed.fits, disp.fun.param), with the dispersion numerically optimized so the simulated data match the target R2. In each of B=100 repeats, fit a linear regression of log-transformed counts on genotype and call significance at the multiple-testing-corrected threshold to obtain empirical power. HWE sampling is appropriate here because genotype realism matters only through its effect size relationship, not through family structure or population stratification.

## Related tools

- **scPower** (R package whose simulation-based eQTL power routine consumes HWE-simulated SNP genotypes as regression covariates for small-mean genes) — https://github.com/heiniglab/scPower
- **HardyWeinberg** (CRAN dependency installed alongside scPower, used for Hardy-Weinberg equilibrium genotype generation/analysis)
- **R** (Runtime environment in which the genotype simulation and downstream scPower power calculations are executed)

## Examples

```
install.packages(c("pwr","MKmisc","reshape2","HardyWeinberg","plotly", "shiny"))
```

## Evaluation signals

- Simulated genotype counts at each SNP match Hardy-Weinberg proportions (p^2, 2pq, q^2) within sampling error (e.g. chi-square HWE test does not reject)
- Genotype matrix dimensions equal the reference sample size x number of simulated SNPs, with values in {0,1,2}
- Downstream empirical power values from the B=100 regression repeats fall within [0,1] and agree with the analytic scPower power within expected simulation uncertainty
- Result table (simulation_vs_analytic_power.csv) contains one row per gene/design setting and exists with non-empty power columns
- Varying the allele frequency changes simulated genotype frequencies in the expected direction (more heterozygotes near p=0.5)

## Limitations

- HWE sampling assumes random mating and no population structure; it cannot represent stratified or related cohorts
- The article notes that larger data sets with hundreds of samples, as required for eQTL studies, will be very difficult to simulate (runtime/memory)
- Simulation-based power is stochastic — agreement with analytic power is only expected within simulation uncertainty across B=100 repeats

## Evidence

- [readme] install.packages(c("pwr","MKmisc","reshape2","HardyWeinberg","plotly", "shiny")): [evidence span withheld — anchor status: external_doc]
- [discussion] larger data sets with hundreds of samples, as required for eQTL studies, will be very difficult to simulate.: "larger data sets with hundreds of samples, as required for eQTL studies, will be very difficult to simulate."
- [methods] We model the overall detection power as the product of the expression probability of a gene and the eQTL power of a gene.: "We model the overall detection power as the product of the expression probability of a gene and the eQTL power of a gene."
