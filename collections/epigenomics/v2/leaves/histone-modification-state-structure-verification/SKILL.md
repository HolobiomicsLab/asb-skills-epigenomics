---
name: histone-modification-state-structure-verification
description: 'Use when reproducing or auditing histoneHMM''s emission model: you have
  binned bivariate ChIP-seq read counts from two samples (e.g. BN vs SHR) and need
  to verify the four-component copula fit yields coherent modified/unmodified/differential
  state classes.'
when_to_use_negative:
- Input is raw BAM/bed reads rather than preprocessed binned read counts — run the
  histoneHMM preprocessing (preprocess.for.hmm, countBamInGRanges, truncation at the
  upper 0.1 percentile) first
- The analysis goal is peak-like modification calling in a single sample with narrow
  marks, where broad-footprint bivariate differential modeling is not the appropriate
  scope
- Only one ChIP-seq sample is available — the Gaussian-copula coupling requires two
  samples to construct the bivariate emission density
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: histoneHMM
  role: Provides fitzinba, fit.zinba.copula, and bivariate.zinba.hmm.posterior to
    fit the zero-inflated negative-binomial marginals and couple them via a Gaussian
    copula into the four-component bivariate emission density
  repo: github.com/matthiasheinig/histoneHMM
- name: R
  role: Computing environment in which the histoneHMM package runs and the copula
    model is fitted and serialized
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-015-0491-6
    title: 'histoneHMM: Differential analysis of histone modifications with broad
      genomic footprints'
  source_content_hash: sha256:05adea3d27404e4869de1cc2ab626d21fb127c568350380f4f525db94b14c24a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - histoneHMM
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# histone-modification-state-structure-verification

## Summary

Reconstructs and verifies the Gaussian-copula bivariate emission model underlying histoneHMM, confirming that the four mixture components of the coupled zero-inflated negative-binomial marginals map onto the modified-in-both, unmodified-in-both, and differentially-modified state classes. This validates the probabilistic state structure that drives histoneHMM's three-state genomic region classifications.

## When to use

Use when reproducing or auditing histoneHMM's emission model: you have binned bivariate ChIP-seq read counts from two samples (e.g. BN vs SHR) and need to verify the four-component copula fit yields coherent modified/unmodified/differential state classes.

## When NOT to use

- Input is raw BAM/bed reads rather than preprocessed binned read counts — run the histoneHMM preprocessing (preprocess.for.hmm, countBamInGRanges, truncation at the upper 0.1 percentile) first
- The analysis goal is peak-like modification calling in a single sample with narrow marks, where broad-footprint bivariate differential modeling is not the appropriate scope
- Only one ChIP-seq sample is available — the Gaussian-copula coupling requires two samples to construct the bivariate emission density

## Inputs

- Preprocessed binned bivariate read count tables for two samples (BN.txt, SHR.txt from http://histonehmm.molgen.mpg.de/data/)
- Fitted zero-inflated negative-binomial marginal objects per sample (from fitzinba)
- histoneHMM R package (installed via devtools::install_github("matthiasheinig/histoneHMM"))

## Outputs

- Fitted Gaussian-copula bivariate emission model object saved as zinba_copula_fit.rds
- Parameter summary table (size, mu, beta per sample plus copula correlation) saved as zinba_copula_params.csv
- Component-to-state mapping showing how the four mixture components partition bins into modified-in-both, unmodified-in-both, and differentially-modified classes

## How to apply

Load preprocessed binned read count tables (BN.txt and SHR.txt from http://histonehmm.molgen.mpg.de/data/) and fit a zero-inflated negative-binomial marginal per sample with fitzinba(x, weight, start, fix.beta). Couple the two ZINB marginals through a Gaussian copula using fit.zinba.copula(x, y, weight, marginal.x, marginal.y), or equivalently bivariate.zinba.hmm.posterior with marginal.mu, marginal.size, marginal.beta, mu.coef, size.coef, beta0, to construct the four-component bivariate emission density. Extract the estimated marginal parameters (size, mu, beta per sample) and the copula covariance/correlation, then check that the four mixture components partition bins into the modified-in-both, unmodified-in-both, and differentially-modified classes reported by histoneHMM. The rationale is that histoneHMM takes binned bivariate read counts as inputs to an unsupervised classification procedure with no further tuning parameters, so the emission model must be structurally correct before downstream HMM inference (EM estimation, Baum-Welch transitions, forward-backward posteriors) can be trusted.

## Related tools

- **histoneHMM** (Provides fitzinba, fit.zinba.copula, and bivariate.zinba.hmm.posterior to fit the zero-inflated negative-binomial marginals and couple them via a Gaussian copula into the four-component bivariate emission density) — github.com/matthiasheinig/histoneHMM
- **R** (Computing environment in which the histoneHMM package runs and the copula model is fitted and serialized)

## Examples

```
library(histoneHMM); bn <- read.table("BN.txt"); shr <- read.table("SHR.txt"); m.x <- fitzinba(bn$x, bn$weight); m.y <- fitzinba(shr$x, shr$weight); fit <- fit.zinba.copula(bn$x, shr$x, bn$weight, m.x, m.y); saveRDS(fit, "zinba_copula_fit.rds")
```

## Evaluation signals

- zinba_copula_fit.rds exists and the fitted copula model contains exactly four components
- All estimated marginal parameters (size, mu, beta) and the copula correlation are finite
- Copula covariance/correlation values fall within [-1, 1]
- The four mixture components partition bins into the expected modified-in-both, unmodified-in-both, and differentially-modified classes consistent with histoneHMM's reported classifications
- Marginal fits (fitzinba) converge without relying on further tuning parameters, consistent with histoneHMM's unsupervised design

## Limitations

- The paper's supplementary material (Additional file 1) describing simulation-based parameter settings is not included, so validation of state-structure choices against simulation is not directly reproducible from the provided text
- histoneHMM is distributed via a project website URL only with no version number or archival DOI stated in the article, complicating exact reproduction
- Read counts are truncated at the upper 0.1 percentile before modeling; skipping this step causes artifacts from extreme read counts and numerical problems with very small emission probabilities

## Evidence

- [abstract] histoneHMM aggregates short-reads over larger regions and takes the resulting bivariate read counts as inputs for an unsupervised classification procedure, requiring no further tuning parameters: "histoneHMM aggregates short-reads over larger regions and takes the resulting bivariate read counts as inputs for an unsupervised classification procedure, requiring no further tuning parameters"
- [discussion] Parameter estimates via EM: "Parameter estimates are obtained via the EM algorithm"
- [discussion] Truncate read counts at upper percentile: "In order to avoid artifacts from regions with extreme read counts [40] and to avoid numerical problems with very small emission probabilities we truncated read counts at the upper 0.1 percentile"
- [methods] Install from GitHub: "devtools::install_github("matthiasheinig/histoneHMM")"
- [readme] Test data set provided: "We provide a small test data set to try out the package:"
