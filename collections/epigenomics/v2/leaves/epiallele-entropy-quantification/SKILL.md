---
name: epiallele-entropy-quantification
description: Use when you have per-locus epiallele proportions from multi-region tumour
  RRBS data and need a scalar measure of epigenetic heterogeneity (disorder) to compare
  tumour regions against matched normal tissue.
when_to_use_negative:
- You need locus-level differential epiallele composition between sample groups rather
  than a global per-sample heterogeneity summary
- Loci fail the article's quality filters (median read depth < 100, fewer than six
  CpGs, > 25% missing data, or chromosomes X/Y), since entropy estimates there are
  not comparable
- Reads cannot be reasonably assigned at a locus and epialleles have not been inferred
  via bed_fit/bed_marginal — entropy of raw read patterns is not the paper's statistic
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0625
tools:
- name: R
  role: Language in which the bed package and entropy computation/plotting are implemented;
    R base graphics used for the Fig. 5 box plots
  repo: https://github.com/james-e-barrett/bed
- name: bed_fit
  role: Fits the Bayesian epiallele detection model to the read data at each locus
    to infer epialleles prior to entropy computation
  repo: https://github.com/james-e-barrett/bed
- name: bed_marginal
  role: Marginalises read assignments over w to produce the epiallele distributions
    phi_s used in the entropy calculation
  repo: https://github.com/james-e-barrett/bed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-017-1753-2
    title: Quantification of tumour evolution and heterogeneity via Bayesian epiallele
      detection
  source_content_hash: sha256:5bc403898ddaa07865d15eb2b18fe0a0cfa9751f9b38ba19ef528cf70ac3cda6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/james-e-barrett/bed
  tools:
  - R
  - bed_fit
  - bed_marginal
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# epiallele-entropy-quantification

## Summary

Quantifies global epigenetic disorder per sample by computing the normalised Shannon entropy of epiallele distributions (phi_s) at each methylation locus, using the bed R package's bed_fit/bed_marginal outputs from multi-region RRBS data, as applied to the CRUK0062 lung tumour (R1-R7 plus matched normal N) in Fig. 5.

## When to use

Use when you have per-locus epiallele proportions from multi-region tumour RRBS data and need a scalar measure of epigenetic heterogeneity (disorder) to compare tumour regions against matched normal tissue.

## When NOT to use

- You need locus-level differential epiallele composition between sample groups rather than a global per-sample heterogeneity summary
- Loci fail the article's quality filters (median read depth < 100, fewer than six CpGs, > 25% missing data, or chromosomes X/Y), since entropy estimates there are not comparable
- Reads cannot be reasonably assigned at a locus and epialleles have not been inferred via bed_fit/bed_marginal — entropy of raw read patterns is not the paper's statistic

## Inputs

- Per-locus epiallele distributions phi_s for each sample (seven tumour regions R1-R7 plus matched normal N), obtained by marginalising read assignments over w with bed_marginal after bed_fit model fitting
- RRBS data from multi-region tumour sampling (e.g. TRACERx CRUK0062 lung tumour) at loci passing the article's filters (median read depth >= 100, >= 6 CpGs, <= 25% missing, autosomes only)

## Outputs

- Normalised Shannon entropy values per locus per sample, bounded in [0,1]
- Summary table of per-sample entropy distributions (e.g. median, quartiles), e.g. epiallele_entropy.csv
- Box plots of entropy distributions across normal (N) and tumour regions (R1-R7), reproducing the Fig. 5 comparison

## How to apply

Fit the Bayesian epiallele detection model per locus with bed_fit, then marginalise read assignments over w using bed_marginal to obtain per-sample epiallele distributions phi_s for each region (R1-R7 and normal N). Discard any epiallele accounting for less than 5% of observed reads at a locus before computing phi_s, as the authors did prior to entropy computation. At each locus compute the normalised Shannon entropy -(1/d) * sum_q phi_q * log2(phi_q), where d is the number of CpGs at that locus, which bounds the statistic in [0,1]. Aggregate locus-level entropies into per-sample distributions (e.g. median and quartiles, written to epiallele_entropy.csv) and draw box plots across normal and tumour regions with R base graphics, reproducing the Fig. 5 comparison. Validation: tumour regions should show a substantially elevated entropy distribution relative to the matched normal sample.

## Related tools

- **R** (Language in which the bed package and entropy computation/plotting are implemented; R base graphics used for the Fig. 5 box plots) — https://github.com/james-e-barrett/bed
- **bed_fit** (Fits the Bayesian epiallele detection model to the read data at each locus to infer epialleles prior to entropy computation) — https://github.com/james-e-barrett/bed
- **bed_marginal** (Marginalises read assignments over w to produce the epiallele distributions phi_s used in the entropy calculation) — https://github.com/james-e-barrett/bed

## Examples

```
library(devtools); install_github("james-e-barrett/bed"); fit <- bed_fit(Y); phi <- bed_marginal(Y, X, W)
```

## Evaluation signals

- All entropy values lie in [0,1], since the phi_q sum to 1 and normalisation by the CpG count d bounds the statistic
- Tumour region entropy distributions are substantially elevated relative to the matched normal sample, as reported for Fig. 5
- Epialleles accounting for less than 5% of observed reads were discarded at each locus before phi_s computation
- Only loci with median read depth >= 100, six or more CpGs, no more than 25% missing data, and non-X/Y chromosomes contribute entropy values
- Per-sample summary (median, quartiles) written to epiallele_entropy.csv covers all eight samples (R1-R7 and N)

## Limitations

- Reads cannot be phased between disjoint loci, so entropy may be correlated between close loci rather than independent
- Whether epiallele-level disorder quantification provides a more refined measure of tumour aggressiveness remains unverified
- Demonstrated on a single patient (one tumour with seven regions plus one matched normal); multi-patient validation deferred to future work

## Evidence

- [methods] the entropy of the epiallele distribution ϕ s quantifies how disordered or heterogeneous each locus is in sample s: "the entropy of the epiallele distribution ϕ s quantifies how disordered or heterogeneous each locus is in sample s"
- [methods] any epialleles that accounted for less than 5% of observed reads were discarded prior to the computation of ϕ s: "any epialleles that accounted for less than 5% of observed reads were discarded prior to the computation of ϕ s"
- [methods] Only loci with a median read depth ≥100 across normal and tumour tissue samples and six or more CpGs were considered.: "Only loci with a median read depth ≥100 across normal and tumour tissue samples and six or more CpGs were considered."
- [abstract] We have developed a Bayesian model to infer which epialleles are present in multiple regions of the same tumour.: "We have developed a Bayesian model to infer which epialleles are present in multiple regions of the same tumour."
- [abstract] R code is available at github.com/james-e-barrett: "R code is available at github.com/james-e-barrett"
