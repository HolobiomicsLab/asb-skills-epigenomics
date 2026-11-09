---
name: multi-region-tumour-heterogeneity-comparison
description: Use when you have per-locus epiallele distributions from multiple tumour
  regions plus matched normal tissue and need a scalar, per-sample measure of epigenetic
  heterogeneity to compare regions against normal.
when_to_use_negative:
- The goal is localising which individual CpGs or loci differ between sample groups
  — use the differential epiallele-composition analysis instead; entropy is a per-sample
  global disorder statistic
- Reads cannot be phased across disjoint loci in the data, and the question requires
  phasing between distant loci — the article notes this is a stated limitation
- Single-sample data with no multi-region or matched-normal comparison available —
  the disorder comparison requires at least tumour and normal samples to contrast
tools:
- name: bed_fit
  role: Fits the Bayesian epiallele detection model to the pooled multi-sample RRBS
    reads to infer epialleles X and assignments w at each locus
  repo: https://github.com/james-e-barrett/bed
- name: bed_marginal
  role: Marginalises read assignments over w to produce the per-locus epiallele distributions
    phi_s used as input to entropy computation
  repo: https://github.com/james-e-barrett/bed
- name: R
  role: Computes normalised Shannon entropies and draws box plots of per-sample entropy
    distributions using base graphics
  repo: https://github.com/james-e-barrett/bed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-017-1753-2
    title: Quantification of tumour evolution and heterogeneity via Bayesian epiallele
      detection
  source_content_hash: sha256:cba386f1d7f590d92136890892b88a288a719b0f7970c4a50d5d2c74b17065cd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/james-e-barrett/bed
  tools:
  - R
  - bed_fit
  - bed_marginal
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# multi-region-tumour-heterogeneity-comparison

## Summary

Quantifies global epigenetic disorder in a multi-region tumour by computing normalised Shannon entropy of epiallele distributions at each locus in each sample (tumour regions R1-R7 vs matched normal N), then comparing entropy distributions across samples. Grounded in the Bayesian Epiallele Detection (bed) workflow applied to RRBS data from the CRUK0062 lung tumour.

## When to use

Use when you have per-locus epiallele distributions from multiple tumour regions plus matched normal tissue and need a scalar, per-sample measure of epigenetic heterogeneity to compare regions against normal.

## When NOT to use

- The goal is localising which individual CpGs or loci differ between sample groups — use the differential epiallele-composition analysis instead; entropy is a per-sample global disorder statistic
- Reads cannot be phased across disjoint loci in the data, and the question requires phasing between distant loci — the article notes this is a stated limitation
- Single-sample data with no multi-region or matched-normal comparison available — the disorder comparison requires at least tumour and normal samples to contrast

## Inputs

- Per-locus epiallele distributions phi_s for each sample (seven tumour regions R1-R7 plus normal N), obtained by marginalising read assignments over w with bed_marginal following bed_fit model fitting
- RRBS read data from multiple regions of a single tumour plus matched normal, pre-filtered to loci with median read depth >= 100, >= 6 CpGs, <= 25% missing data, autosomes only

## Outputs

- Per-sample distributions of locus-level normalised Shannon entropies (values in [0,1])
- Summary table epiallele_entropy.csv with per-sample median and quartiles
- Box plots of entropy distributions across normal (N) and tumour regions (R1-R7), reproducing the Fig. 5 comparison

## How to apply

Fit the Bayesian epiallele model with bed_fit on pooled multi-sample RRBS reads, then use bed_marginal to marginalise read assignments over w and obtain per-locus epiallele distributions phi_s for each sample. Discard any epiallele accounting for less than 5% of observed reads at a locus before computing phi_s. At each locus compute the normalised Shannon entropy -(1/d) * sum_q phi_q * log2(phi_q), where d is the number of CpGs at that locus (loci considered had median read depth >= 100 and six or more CpGs). Aggregate locus-level entropies into per-sample distributions for each tumour region and the normal, write summary statistics (e.g. median, quartiles) to epiallele_entropy.csv, and draw box plots comparing N against R1-R7 using R base graphics, as in Fig. 5.

## Related tools

- **bed_fit** (Fits the Bayesian epiallele detection model to the pooled multi-sample RRBS reads to infer epialleles X and assignments w at each locus) — https://github.com/james-e-barrett/bed
- **bed_marginal** (Marginalises read assignments over w to produce the per-locus epiallele distributions phi_s used as input to entropy computation) — https://github.com/james-e-barrett/bed
- **R** (Computes normalised Shannon entropies and draws box plots of per-sample entropy distributions using base graphics) — https://github.com/james-e-barrett/bed

## Examples

```
library(devtools); install_github("james-e-barrett/bed"); ?bed_fit
```

## Evaluation signals

- Entropy values must lie in [0,1] since phi_q sums to 1 and normalisation by the CpG count d bounds the statistic
- Tumour regions R1-R7 must exhibit a substantially elevated entropy distribution relative to the matched normal sample N, as reported for Fig. 5
- Confirm the low-frequency filter was applied: epialleles accounting for less than 5% of observed reads were discarded prior to computing phi_s
- Confirm loci entering the analysis satisfy median read depth >= 100 and >= 6 CpGs with no more than 25% missing data
- epiallele_entropy.csv exists with per-sample summary statistics (median, quartiles) for all eight samples

## Limitations

- Reads cannot be phased between disjoint loci; the number of epialleles and entropy may be correlated between close loci
- Case study covers a single patient (one lung tumour, CRUK0062, seven regions plus one matched normal); larger multi-patient studies deferred to future work
- Whether epiallele-level disorder quantification provides a more refined measure of tumour aggressiveness remains unverified

## Evidence

- [methods] Low-frequency filter and phi_s computation: "any epialleles that accounted for less than 5% of observed reads were discarded prior to the computation of ϕ s"
- [methods] Entropy as disorder measure: "the entropy of the epiallele distribution ϕ s quantifies how disordered or heterogeneous each locus is in sample s"
- [abstract] Global epigenetic disorder as a finding: "By tracing the distribution of epialleles throughout the tumour we can infer the phylogenetic history of the tumour"
- [methods] Locus quality thresholds: "Only loci with a median read depth ≥100 across normal and tumour tissue samples and six or more CpGs were considered."
- [readme] Package installation path: "To install this R package use the devtools install_github command:"
