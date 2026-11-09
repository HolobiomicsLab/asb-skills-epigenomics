---
name: shannon-entropy-computation-from-probability-distributions
description: Use when RRBS-derived epiallele proportions phi_s exist per locus per
  sample (e.g. tumour regions R1-R7 plus normal N) and the goal is to quantify how
  disordered or heterogeneous each locus/sample is via entropy.
when_to_use_negative:
- The locus has fewer than six CpGs, median read depth below 100, more than 25% missing
  data, or lies on chromosomes X/Y — these loci are excluded before entropy computation
- Epiallele proportions are unavailable (e.g. reads were not pooled across tissue
  samples and modelled with bed_fit/bed_marginal); raw read counts alone cannot be
  used directly
- The goal is inter-sample differential comparison of epiallele composition at specific
  loci rather than a per-sample global disorder measure
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_2640
tools:
- name: R
  role: Implementation language for the bed package and for entropy computation and
    Fig. 5 box plots using base graphics
  repo: github.com/james-e-barrett
- name: bed_fit
  role: Fits the Bayesian epiallele detection model to pooled reads to infer epialleles
    before marginalisation
  repo: github.com/james-e-barrett/bed
- name: bed_marginal
  role: Marginalises read assignments over w to produce the epiallele distribution
    phi_s used in the entropy formula
  repo: github.com/james-e-barrett/bed
- name: devtools
  role: Used to install the bed package from GitHub via install_github
  repo: github.com/james-e-barrett/bed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-017-1753-2
    title: Quantification of tumour evolution and heterogeneity via Bayesian epiallele
      detection
  source_content_hash: sha256:2573b00411b205e3991482945bdf91cf7ae9c5273916a5191b5cb1671cc873e6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/github.com/james-e-barrett
  tools:
  - R
  - bed_fit
  - bed_marginal
  - devtools
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# shannon-entropy-computation-from-probability-distributions

## Summary

Quantifies global epigenetic disorder in multi-region tumour samples by computing the normalised Shannon entropy of epiallele distributions phi_s at each locus, after Bayesian epiallele inference with the bed R package (bed_fit / bed_marginal).

## When to use

Use when RRBS-derived epiallele proportions phi_s exist per locus per sample (e.g. tumour regions R1-R7 plus normal N) and the goal is to quantify how disordered or heterogeneous each locus/sample is via entropy.

## When NOT to use

- The locus has fewer than six CpGs, median read depth below 100, more than 25% missing data, or lies on chromosomes X/Y — these loci are excluded before entropy computation
- Epiallele proportions are unavailable (e.g. reads were not pooled across tissue samples and modelled with bed_fit/bed_marginal); raw read counts alone cannot be used directly
- The goal is inter-sample differential comparison of epiallele composition at specific loci rather than a per-sample global disorder measure

## Inputs

- Per-locus epiallele distributions phi_s for each sample (seven tumour regions R1-R7 plus matched normal N) obtained via bed_fit + bed_marginal
- Number of CpGs d per locus (loci required to have >=6 CpGs, median read depth >=100, <=25% missing data, autosomes only)

## Outputs

- Locus-level normalised Shannon entropy values in [0,1] for each sample
- Per-sample entropy distribution summary table (e.g. epiallele_entropy.csv with median, quartiles)
- Box plots of entropy distributions across normal (N) and tumour regions (R1-R7), reproducing the Fig. 5 comparison

## How to apply

First obtain per-locus epiallele distributions phi_s by fitting the Bayesian model with bed_fit and marginalising read assignments over w using bed_marginal; before computing phi_s, discard any epialleles accounting for less than 5% of observed reads. For each locus and sample compute the normalised Shannon entropy -(1/d) * sum_q phi_q * log2(phi_q), where d is the number of CpGs at that locus (loci considered had median read depth >=100 and six or more CpGs). Aggregate locus-level entropies into per-sample distributions (e.g. median and quartiles) for the seven tumour regions CRUK0062 R1-R7 and the matched normal, writing a summary table such as epiallele_entropy.csv. Finally compare distributions across normal versus tumour regions with box plots (e.g. R base graphics reproducing Fig. 5), since the sum of phi_q equals 1 and normalisation by d bounds the entropy in [0,1].

## Related tools

- **R** (Implementation language for the bed package and for entropy computation and Fig. 5 box plots using base graphics) — github.com/james-e-barrett
- **bed_fit** (Fits the Bayesian epiallele detection model to pooled reads to infer epialleles before marginalisation) — github.com/james-e-barrett/bed
- **bed_marginal** (Marginalises read assignments over w to produce the epiallele distribution phi_s used in the entropy formula) — github.com/james-e-barrett/bed
- **devtools** (Used to install the bed package from GitHub via install_github) — github.com/james-e-barrett/bed

## Examples

```
library(devtools); install_github("james-e-barrett/bed"); ?bed_fit
```

## Evaluation signals

- All entropy values lie in [0,1] since sum of phi_q equals 1 and the statistic is normalised by d (number of CpGs)
- Tumour tissue samples (R1-R7) show a substantially elevated entropy distribution relative to the matched normal sample N, as reported for Fig. 5
- Epialleles contributing less than 5% of observed reads at a locus were discarded before phi_s was computed
- Summary table epiallele_entropy.csv contains per-sample medians and quartiles for the normal and all seven tumour regions
- Only loci with median read depth >=100, >=6 CpGs, <=25% missing data, and no chromosomes X/Y are represented

## Limitations

- Reads cannot be phased between disjoint loci, so the number of epialleles and entropy may be correlated between close loci
- Demonstrated only on a single patient (CRUK0062, seven tumour regions plus matched normal); larger multi-patient validation deferred to future work
- Whether epiallele-level disorder quantification provides a more refined measure of tumour aggressiveness remains unverified

## Evidence

- [methods] the entropy of the epiallele distribution ϕ_s quantifies how disordered or heterogeneous each locus is in sample s: "the entropy of the epiallele distribution ϕ
s quantifies how disordered or heterogeneous each locus is in sample s"
- [methods] any epialleles that accounted for less than 5% of observed reads were discarded prior to the computation of ϕ_s: "any epialleles that accounted for less than 5% of observed reads were discarded prior to the computation of ϕ
s"
- [methods] Only loci with a median read depth ≥100 across normal and tumour tissue samples and six or more CpGs were considered.: "Only loci with a median read depth ≥100 across normal and tumour tissue samples and six or more CpGs were considered."
- [abstract] define a measure of global epigenetic disorder by tracing epiallele distribution throughout the tumour: "By tracing the distribution of epialleles throughout the tumour we can infer the phylogenetic history of the tumour, identify epialleles that differ between normal and cancer tissue, and define a"
- [readme] To install this R package use the devtools install_github command: install_github("james-e-barrett/bed"): "To install this R package use the devtools install_github command:

    library(devtools)
    install_github("james-e-barrett/bed")"
