---
name: gaussian-copula-bivariate-emission-construction
description: Use when comparing two ChIP-seq samples of broad histone marks (e.g.
  H3K27me3/H3K9me3) and you need the bivariate emission density behind histoneHMM's
  three-state differential classifications from binned bivariate read counts.
when_to_use_negative:
- The analysis targets peak-like histone marks rather than broad diffuse footprints
  like H3K27me3 — the article notes methods searching for peak-like features generate
  many false positive or false negative calls on broad marks, and this emission model
  is built for the broad-footprint setting
- You only have a single sample's ChIP-seq profile — the copula construction requires
  two samples' bivariate read counts; for single-sample calling use the univariate
  HMM (hmm.posterior / run.univariate.hmm) instead
- Read counts have not yet been binned over larger regions (e.g. 1000 bp bins) and
  deduplicated/truncated — the copula fit takes aggregated bivariate bin counts, not
  raw reads
edam_operation: http://edamontology.org/operation_0415
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: histoneHMM
  role: R package providing fitzinba, fit.zinba.copula, and bivariate.zinba.hmm.posterior
    used to fit the ZINB marginals and construct the Gaussian-copula four-component
    emission density
  repo: github.com/matthiasheinig/histoneHMM
- name: R
  role: Computing environment in which histoneHMM is compiled and executed; installed
    via devtools::install_github("matthiasheinig/histoneHMM")
- name: samtools
  role: Used upstream to remove duplicated reads before binned read counts are produced
    for the emission model fit
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-015-0491-6
    title: 'histoneHMM: Differential analysis of histone modifications with broad
      genomic footprints'
  source_content_hash: sha256:83c37542b42c4f95002093cd230567418bd311d6627a6c50b1f742c2a73120ed
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - histoneHMM
  - samtools
  edam_operation: http://edamontology.org/operation_0415
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# gaussian-copula-bivariate-emission-construction

## Summary

Reconstruct the bivariate emission model of histoneHMM by fitting zero-inflated negative-binomial marginals to each sample's binned ChIP-seq read counts and coupling them through a Gaussian copula, yielding the four-component emission density used to probabilistically classify regions as modified in both, unmodified in both, or differentially modified between two samples.

## When to use

Use when comparing two ChIP-seq samples of broad histone marks (e.g. H3K27me3/H3K9me3) and you need the bivariate emission density behind histoneHMM's three-state differential classifications from binned bivariate read counts.

## When NOT to use

- The analysis targets peak-like histone marks rather than broad diffuse footprints like H3K27me3 — the article notes methods searching for peak-like features generate many false positive or false negative calls on broad marks, and this emission model is built for the broad-footprint setting
- You only have a single sample's ChIP-seq profile — the copula construction requires two samples' bivariate read counts; for single-sample calling use the univariate HMM (hmm.posterior / run.univariate.hmm) instead
- Read counts have not yet been binned over larger regions (e.g. 1000 bp bins) and deduplicated/truncated — the copula fit takes aggregated bivariate bin counts, not raw reads

## Inputs

- Preprocessed binned bivariate read-count tables for the two samples (e.g. BN.txt and SHR.txt from http://histonehmm.molgen.mpg.de/data/), counts truncated at the upper 0.1 percentile
- Optionally raw BAM files (BN.bam, SHR.bam) with chromosome-length table chroms.txt and gene annotation ensembl59-genes.gff for preprocessing via preprocess.for.hmm

## Outputs

- Fitted four-component Gaussian-copula ZINB emission model saved as zinba_copula_fit.rds
- Parameter summary table (per-sample size, mu, beta and copula correlation) as zinba_copula_params.csv
- Four mixture components mapping onto modified-in-both, unmodified-in-both, and differentially-modified region classes

## How to apply

Load the preprocessed binned bivariate read-count tables (BN.txt and SHR.txt from http://histonehmm.molgen.mpg.de/data/) where the genome has been partitioned into 1000 bp bins and extreme counts truncated at the upper 0.1 percentile. Fit a zero-inflated negative-binomial marginal per sample individually with fitzinba(x, weight, start, fix.beta) from the histoneHMM R package, then couple the two fitted marginals via fit.zinba.copula(x, y, weight, marginal.x, marginal.y) to form the four-component bivariate emission density; the equivalent low-level entry point is bivariate.zinba.hmm.posterior with marginal.mu, marginal.size, marginal.beta, mu.coef, size.coef, beta0. The Gaussian copula captures the dependence between the two samples' read counts, so the four mixture components should map onto the modified-in-both, unmodified-in-both, and differentially-modified state classes that histoneHMM reports. Extract the per-sample size, mu, and beta parameters plus the copula covariance/correlation, save the model object as zinba_copula_fit.rds and a parameter table as zinba_copula_params.csv.

## Related tools

- **histoneHMM** (R package providing fitzinba, fit.zinba.copula, and bivariate.zinba.hmm.posterior used to fit the ZINB marginals and construct the Gaussian-copula four-component emission density) — github.com/matthiasheinig/histoneHMM
- **R** (Computing environment in which histoneHMM is compiled and executed; installed via devtools::install_github("matthiasheinig/histoneHMM"))
- **samtools** (Used upstream to remove duplicated reads before binned read counts are produced for the emission model fit)

## Examples

```
install.packages("devtools"); devtools::install_github("matthiasheinig/histoneHMM")
```

## Evaluation signals

- zinba_copula_fit.rds exists and is a valid RDS model object
- The fitted model contains exactly four mixture components
- All marginal parameters (size, mu, beta) and the copula correlation are finite, with correlations within [-1, 1]
- The four components partition bins into the expected modified-in-both / unmodified-in-both / differentially-modified classes
- Marginal fits reproduce each sample's zero-inflated binned count distribution (zeros plus overdispersed positive counts)

## Limitations

- Extreme read counts cause numerical problems with very small emission probabilities; counts must be truncated at the upper 0.1 percentile before fitting
- The method is designed for broad genomic footprints such as H3K27me3/H3K9me3 and may not suit peak-like modifications
- No version number or archival repository (e.g. Zenodo DOI) is stated for histoneHMM in the article; version 1.7 is needed for compatibility with Rcpp > 0.12 per the README

## Evidence

- [abstract] histoneHMM aggregates short-reads over larger regions and takes the resulting bivariate read counts as inputs for an unsupervised classification procedure: "histoneHMM aggregates short-reads over larger regions and takes the resulting bivariate read counts as inputs for an unsupervised classification procedure"
- [methods] devtools::install_github("matthiasheinig/histoneHMM"): "install.packages("devtools")
devtools::install_github("matthiasheinig/histoneHMM")"
- [methods] preprocess.for.hmm(signal.bam, genes, bin.size, genome, chr, expr, offset, n.expr.bins, plot): "preprocess.for.hmm(signal.bam, genes, bin.size, genome, chr, expr, offset, n.expr.bins, plot)"
- [discussion] truncated read counts at the upper 0.1 percentile: "we truncated read counts at the upper 0.1 percentile"
- [discussion] partition the genome into m equally sized bins (1000 bp by default): "we partition the genome into m equally sized bins (1000 bp by default)"
- [readme] README test data set BN.txt and SHR.txt: "We provide a small test data set to try out the package:"
