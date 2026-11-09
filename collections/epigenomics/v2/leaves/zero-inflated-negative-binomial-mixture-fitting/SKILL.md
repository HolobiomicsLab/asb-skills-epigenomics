---
name: zero-inflated-negative-binomial-mixture-fitting
description: Use when you have preprocessed binned bivariate read-count tables (e.g.
  histoneHMM BN.txt/SHR.txt) and need to reconstruct or fit the emission densities
  of a bivariate HMM for differential histone modification analysis.
when_to_use_negative:
- Input is raw BAM/bed data not yet aggregated into binned read counts — preprocess
  first (e.g. with preprocess.for.hmm and 1000 bp bins) before fitting marginals
- The data are peak-like ChIP-seq marks rather than broad diffuse histone modifications,
  where methods searching for peak-like features are more appropriate
- You need a supervised or tuning-parameter-driven classification — histoneHMM's emission
  fitting is explicitly unsupervised with no further tuning parameters
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: histoneHMM
  role: R package providing fitzinba, fit.zinba.copula, and bivariate.zinba.hmm.posterior
    for ZINB marginal fitting and Gaussian-copula coupling of bivariate emission densities
  repo: https://matthiasheinig.github.io/histoneHMM/
- name: R
  role: Computing environment in which histoneHMM is compiled and the fitting workflow
    is executed
- name: samtools
  role: Used upstream to remove duplicated reads before binning counts for mixture
    fitting
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-015-0491-6
    title: 'histoneHMM: Differential analysis of histone modifications with broad
      genomic footprints'
  source_content_hash: sha256:6fbd373addec8e4727f5b4028958675b8acdba02d868740bbc0f2cdd58475ed8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - histoneHMM
  - samtools
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# zero-inflated-negative-binomial-mixture-fitting

## Summary

Fit zero-inflated negative-binomial (ZINB) marginal distributions to binned bivariate ChIP-seq read counts and couple them through a Gaussian copula, reproducing the four-component bivariate emission model that powers histoneHMM's three-state differential classification.

## When to use

Use when you have preprocessed binned bivariate read-count tables (e.g. histoneHMM BN.txt/SHR.txt) and need to reconstruct or fit the emission densities of a bivariate HMM for differential histone modification analysis.

## When NOT to use

- Input is raw BAM/bed data not yet aggregated into binned read counts — preprocess first (e.g. with preprocess.for.hmm and 1000 bp bins) before fitting marginals
- The data are peak-like ChIP-seq marks rather than broad diffuse histone modifications, where methods searching for peak-like features are more appropriate
- You need a supervised or tuning-parameter-driven classification — histoneHMM's emission fitting is explicitly unsupervised with no further tuning parameters

## Inputs

- Preprocessed binned bivariate read-count tables for two samples (e.g. BN.txt and SHR.txt from histoneHMM test data)
- Genome-partitioned bin definitions (1000 bp bins by default)
- Optionally BAM/bed files and annotation (chroms.txt, ensembl59-genes.gff, expression.txt) for preprocessing via preprocess.for.hmm

## Outputs

- Fitted copula model object (e.g. zinba_copula_fit.rds) containing exactly four mixture components
- Parameter summary table (e.g. zinba_copula_params.csv) with per-sample marginal size, mu, beta and copula correlation
- Per-bin posterior state probabilities feeding modified/unmodified/differential classification

## How to apply

Load the preprocessed binned read-count tables for the two samples (e.g. BN.txt and SHR.txt from http://histonehmm.molgen.mpg.de/data/), where the genome has been partitioned into 1000 bp bins and read counts are truncated at the upper 0.1 percentile. For each sample individually, fit the zero-inflated negative-binomial marginal with fitzinba(x, weight, start, fix.beta) from the histoneHMM R package. Then couple the two fitted ZINB marginals through a Gaussian copula using fit.zinba.copula(x, y, weight, marginal.x, marginal.y) to build the four-component bivariate emission density (equivalently usable via bivariate.zinba.hmm.posterior with marginal.mu, marginal.size, marginal.beta, mu.coef, size.coef, beta0). The rationale is that histoneHMM aggregates short reads over larger regions into binned bivariate counts that feed an unsupervised classification procedure requiring no further tuning parameters, and the EM-estimated mixture components map onto modified-in-both, unmodified-in-both, and differentially-modified states. Save the fitted copula model object and a parameter summary (size, mu, beta per sample plus copula correlation) for downstream HMM posterior or Viterbi computation.

## Related tools

- **histoneHMM** (R package providing fitzinba, fit.zinba.copula, and bivariate.zinba.hmm.posterior for ZINB marginal fitting and Gaussian-copula coupling of bivariate emission densities) — https://matthiasheinig.github.io/histoneHMM/
- **R** (Computing environment in which histoneHMM is compiled and the fitting workflow is executed)
- **samtools** (Used upstream to remove duplicated reads before binning counts for mixture fitting)

## Examples

```
devtools::install_github("matthiasheinig/histoneHMM"); fit <- fit.zinba.copula(x, y, weight, marginal.x, marginal.y)
```

## Evaluation signals

- The fitted copula model contains exactly four mixture components
- All marginal parameters (size, mu, beta) and the copula correlation are finite, with correlations within [-1, 1]
- The four components partition bins into the expected modified-in-both / unmodified-in-both / differentially-modified classes
- Saved artifacts exist: zinba_copula_fit.rds and zinba_copula_params.csv
- Input counts respect the 0.1-percentile truncation so emission probabilities remain numerically stable

## Limitations

- Extreme read counts cause artifacts and numerical problems with very small emission probabilities unless counts are truncated at the upper 0.1 percentile
- No version number or archival DOI is stated for the histoneHMM software on the project website, so reproducibility depends on tracking the GitHub commit
- The pre-binned test data (BN.txt, SHR.txt) must be used as-is if raw BAM files or annotation (chroms.txt, ensembl59-genes.gff) are unavailable

## Evidence

- [abstract] histoneHMM aggregates short-reads over larger regions and takes the resulting bivariate read counts as inputs for an unsupervised classification procedure requiring no further tuning parameters: "histoneHMM aggregates short-reads over larger regions and takes the resulting bivariate read counts as inputs for an unsupervised classification procedure, requiring no further tuning parameters"
- [discussion] Parameter estimates are obtained via the EM algorithm: "Parameter estimates are obtained via the EM algorithm [43]"
- [discussion] Truncation of read counts at upper 0.1 percentile: "we truncated read counts at the upper 0.1 percentile"
- [methods] Installation via devtools from GitHub: "devtools::install_github("matthiasheinig/histoneHMM")"
- [readme] README lists test data files: "We provide a small test data set to try out the package"
