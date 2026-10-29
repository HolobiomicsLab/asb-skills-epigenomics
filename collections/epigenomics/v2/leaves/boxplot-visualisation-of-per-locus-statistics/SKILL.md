---
name: boxplot-visualisation-of-per-locus-statistics
description: Use when per-locus scalar statistics (e.g. Shannon entropy of epiallele
  distributions phi_s) have been computed for multiple tissue samples and you need
  to compare their distributions across normal vs. tumour regions graphically.
when_to_use_negative:
- The goal is to visualise raw read counts or per-CpG methylation fractions rather
  than aggregated per-locus epiallele-level statistics
- Samples do not share loci — the paper required identifying loci occurring in two
  or more samples (median read depth >= 100, >= 6 CpGs, <= 25% missing data) before
  comparing distributions
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0092
tools:
- name: R
  role: Environment in which box plots of per-locus entropy distributions are drawn
    using R base graphics; R code for the bed package is available on GitHub
  repo: https://github.com/james-e-barrett/bed
- name: bed_fit
  role: Fits the Bayesian epiallele detection model to the read data Y upstream of
    marginalisation and entropy computation
  repo: https://github.com/james-e-barrett/bed
- name: bed_marginal
  role: Marginalises read assignments over w to produce the epiallele distribution
    phi_s per sample that the entropy and box plots are based on
  repo: https://github.com/james-e-barrett/bed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-017-1753-2
    title: Quantification of tumour evolution and heterogeneity via Bayesian epiallele
      detection
  source_content_hash: sha256:0ba34d27b45c082bb3375663f040e6d0a498d8b67ab8c257406e2abc810eefc6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/james-e-barrett/bed
  tools:
  - R
  - bed_fit
  - bed_marginal
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# boxplot-visualisation-of-per-locus-statistics

## Summary

Visualises per-locus distributions of a computed statistic — here the normalised epiallele Shannon entropy — as box plots comparing normal (N) and tumour regions (R1–R7) using R base graphics, reproducing the paper's Fig. 5 comparison of global epigenetic disorder.

## When to use

Use when per-locus scalar statistics (e.g. Shannon entropy of epiallele distributions phi_s) have been computed for multiple tissue samples and you need to compare their distributions across normal vs. tumour regions graphically.

## When NOT to use

- The goal is to visualise raw read counts or per-CpG methylation fractions rather than aggregated per-locus epiallele-level statistics
- Samples do not share loci — the paper required identifying loci occurring in two or more samples (median read depth >= 100, >= 6 CpGs, <= 25% missing data) before comparing distributions

## Inputs

- Per-locus epiallele proportions phi_s per sample s (from bed_marginal over bed_fit results), after discarding epialleles accounting for less than 5% of observed reads
- Locus-level normalised Shannon entropy values for each sample (N, R1–R7)
- Locus metadata (number of CpGs d per locus) used for normalisation

## Outputs

- epiallele_entropy.csv — per-sample summary table of the entropy distributions (e.g. median, quartiles)
- Box plot of entropy distributions across normal (N) and tumour regions (R1–R7), reproducing Fig. 5

## How to apply

After computing per-locus normalised Shannon entropies -(1/d) * sum_q phi_q * log2(phi_q) from the marginalised epiallele proportions phi_s (obtained via bed_marginal following bed_fit), aggregate the locus-level entropies into one distribution per sample (seven tumour regions R1–R7 plus matched normal N) and summarise them (e.g. median, quartiles) into a table such as epiallele_entropy.csv. Then draw box plots of the entropy distributions across all eight samples using R base graphics (boxplot()), as in Fig. 5 of the paper. Because phi_q sums to 1 and the statistic is normalised by the number of CpGs d, all entropy values must lie in [0,1], which bounds the plotted range. The visual comparison should show tumour samples with a substantially elevated entropy distribution relative to the normal tissue, confirming global epigenetic disorder in the tumour.

## Related tools

- **R** (Environment in which box plots of per-locus entropy distributions are drawn using R base graphics; R code for the bed package is available on GitHub) — https://github.com/james-e-barrett/bed
- **bed_fit** (Fits the Bayesian epiallele detection model to the read data Y upstream of marginalisation and entropy computation) — https://github.com/james-e-barrett/bed
- **bed_marginal** (Marginalises read assignments over w to produce the epiallele distribution phi_s per sample that the entropy and box plots are based on) — https://github.com/james-e-barrett/bed

## Examples

```
library(bed); phi <- bed_marginal(Y, X, W); ent <- sapply(phi, function(p) -sum(p[p>=0.05]*log2(p[p>=0.05]))/d); boxplot(split(ent, sample_id), names=c('N','R1','R2','R3','R4','R5','R6','R7'))
```

## Evaluation signals

- All plotted entropy values lie within [0,1], since phi_q sums to 1 and normalisation by d bounds the statistic
- Tumour regions R1–R7 show a substantially elevated entropy distribution relative to the matched normal sample N, as reported for Fig. 5
- The summary table epiallele_entropy.csv contains one distribution (median, quartiles) per sample (N plus R1–R7, i.e. eight samples)
- Each entropy distribution is computed only from loci passing the filters: median read depth >= 100, six or more CpGs, no more than 25% missing data, and no chromosomes X or Y
- Low-frequency epialleles (< 5% of observed reads) were discarded before entropy computation, so box plots should not reflect noise-driven entropy inflation

## Limitations

- Reads cannot be phased between disjoint loci, so the number of epialleles and the entropy may be correlated between close loci
- The underlying case study covers a single patient (one lung tumour CRUK0062 with seven regions plus one matched normal), so the visual comparison is not generalisable across patients
- Whether epiallele-level disorder quantification provides a more refined measure of tumour aggressiveness remains unverified

## Evidence

- [methods] the entropy of the epiallele distribution ϕ s quantifies how disordered or heterogeneous each locus is in sample s: "the entropy of the epiallele distribution ϕ s quantifies how disordered or heterogeneous each locus is in sample s"
- [methods] any epialleles that accounted for less than 5% of observed reads were discarded prior to the computation of ϕ s: "any epialleles that accounted for less than 5% of observed reads were discarded prior to the computation of ϕ s"
- [methods] Only loci with a median read depth ≥100 across normal and tumour tissue samples and six or more CpGs were considered.: "Only loci with a median read depth ≥100 across normal and tumour tissue samples and six or more CpGs were considered."
- [abstract] By tracing the distribution of epialleles throughout the tumour we can infer the phylogenetic history of the tumour, identify epialleles that differ between normal and cancer tissue, and define a: "By tracing the distribution of epialleles throughout the tumour we can infer the phylogenetic history of the tumour, identify epialleles that differ between normal and cancer tissue, and define a"
- [methods] Our case study data consisted of seven tissue samples from a single lung tumour (CRUK0062) along with one matched normal tissue sample.: "Our case study data consisted of seven tissue samples from a single lung tumour (CRUK0062) along with one matched normal tissue sample."
- [readme] This code is described in the draft manuscript "Quantification of tumour evolution and heterogeneity via Bayesian epiallele detection" available at https://arxiv.org/abs/1702.00633.: "This code is described in the draft manuscript "Quantification of tumour evolution and heterogeneity via Bayesian epiallele detection" available at https://arxiv.org/abs/1702.00633."
