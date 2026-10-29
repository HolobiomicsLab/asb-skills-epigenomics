---
name: rrbs-epiallele-inference
description: Use when reduced representation bisulfite sequencing (RRBS) read matrices
  are available for two or more samples (e.g. multi-region tumour regions plus matched
  normal) and the goal is to infer which epialleles are present at each eligible CpG
  locus.
when_to_use_negative:
- Reads cannot be phased between disjoint loci, so this skill cannot reconstruct multi-locus
  haplotypes from single-locus RRBS data
- The study design is a single patient's tumour with multi-region sampling; it is
  not validated for larger multi-patient cohorts (deferred to future work in the article)
- Loci failing the filters (chromosomes X/Y, >25% missing data, fewer than six CpGs,
  median read depth <100) are excluded and must not be fitted
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
tools:
- name: bed (R package)
  role: Provides bed_fit(Y) for MAP model fitting and AIC-based selection of Q, and
    bed_marginal(Y, X, W) for posterior marginalisation over read assignments w
  repo: github.com/james-e-barrett/bed
- name: devtools
  role: Used to install the bed R package from GitHub via install_github("james-e-barrett/bed")
  repo: github.com/james-e-barrett/bed
- name: R
  role: Language in which the BED model and analysis scripts are implemented
  repo: github.com/james-e-barrett
provenance:
  source_task_ids:
  - task_003
  - task_007
  source_papers:
  - doi: 10.1186/s12859-017-1753-2
    title: Quantification of tumour evolution and heterogeneity via Bayesian epiallele
      detection
  source_content_hash: sha256:142aa460eae54d89229733f1a6f92cbc03c7cba88ba378d4ea8e11f86a3040cd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - bed (R package)
  - bed_fit
  - bed_marginal
  - devtools
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# rrbs-epiallele-inference

## Summary

A Bayesian Epiallele Detection (BED) workflow that pools RRBS reads from multiple tissue regions of a tumour plus matched normal and infers the number of epialleles (Q), epiallele DNAm patterns (X), per-read assignments (w), and a noise hyperparameter ε directly from the data.

## When to use

Use when reduced representation bisulfite sequencing (RRBS) read matrices are available for two or more samples (e.g. multi-region tumour regions plus matched normal) and the goal is to infer which epialleles are present at each eligible CpG locus.

## When NOT to use

- Reads cannot be phased between disjoint loci, so this skill cannot reconstruct multi-locus haplotypes from single-locus RRBS data
- The study design is a single patient's tumour with multi-region sampling; it is not validated for larger multi-patient cohorts (deferred to future work in the article)
- Loci failing the filters (chromosomes X/Y, >25% missing data, fewer than six CpGs, median read depth <100) are excluded and must not be fitted

## Inputs

- RRBS read matrices Y at each eligible CpG locus (one matrix per sample: seven tumour regions R1-R7 plus matched normal N of lung tumour CRUK0062 in the case study)
- Locus-level filtering already applied: no chromosomes X/Y, ≤25% missing data, ≥6 CpGs, median read depth ≥100

## Outputs

- Inferred epiallele DNAm patterns X
- Inferred total number of epialleles Q (AIC-optimal)
- MAP noise hyperparameter estimate ε = α0/(α0+α1)
- Per-sample epiallele distributions φ_s (proportion of reads per sample attributed to each retained epiallele), e.g. saved as epiallele_inference_results.rds

## How to apply

First filter loci: discard chromosomes X and Y, require no more than 25% missing data per locus, six or more CpGs per locus, and a median read depth ≥100 across normal and tumour samples; also identify loci occurring in two or more samples. For each eligible locus, pool sequencing reads from all tissue samples to borrow information and boost statistical power, then run bed_fit(Y), which initialises read assignments w with hierarchical clustering using Hamming distance, iterates MAP updates of the epiallele assignments w and epiallele patterns X until convergence, and selects the number of epialleles Q by minimising AIC(Q) = -2 log p(Y|X,w,ε,Q) + 2Qd. Compute the MAP noise estimate ε = α0/(α0+α1), discard any epialleles accounting for less than 5% of observed reads, then run bed_marginal(Y, X, W) to marginalise over posterior densities of w and compute per-sample epiallele distributions φ_s. Save the epiallele patterns, inferred Q, ε, and per-sample φ_s vectors (e.g. to an .rds file) for downstream tasks such as entropy-based epigenetic disorder quantification.

## Related tools

- **bed (R package)** (Provides bed_fit(Y) for MAP model fitting and AIC-based selection of Q, and bed_marginal(Y, X, W) for posterior marginalisation over read assignments w) — github.com/james-e-barrett/bed
- **devtools** (Used to install the bed R package from GitHub via install_github("james-e-barrett/bed")) — github.com/james-e-barrett/bed
- **R** (Language in which the BED model and analysis scripts are implemented) — github.com/james-e-barrett

## Examples

```
library(devtools); install_github("james-e-barrett/bed"); fit <- bed_fit(Y); marg <- bed_marginal(Y, fit$X, fit$W)
```

## Evaluation signals

- The iterative MAP procedure converges to a steady solution, typically within two or three iterations
- The retained model corresponds to the minimum AIC score across candidate Q values
- Discarded epialleles are those accounting for less than 5% of observed reads before computing φ_s
- Per-sample φ_s vectors are valid distributions (sum of φ_q equals 1), so downstream entropies lie in [0,1] after normalisation by d
- The MAP noise estimate satisfies ε = α0/(α0+α1) and lies in [0,1]

## Limitations

- Reads cannot be phased between disjoint loci, although the number of epialleles and the entropy may be correlated between close loci
- The case study is limited to a single patient (one tumour with seven regions plus one matched normal); larger multi-patient studies were deferred to future work
- Whether epiallele-level disorder quantification provides a more refined measure of tumour aggressiveness remains unverified

## Evidence

- [methods] Initialise w by using hierarchical clustering to group the observed reads Y into Q groups. The hamming distance: "Initialise w by using hierarchical clustering to group the observed reads Y into Q groups. The hamming distance"
- [methods] Instead we use the Akaike information criterion (AIC) [18] in order to select the optimal number of epialleles: "Instead we use the Akaike information criterion (AIC) [18] in order to select the optimal number of epialleles"
- [methods] Only loci with a median read depth ≥100 across normal and tumour tissue samples and six or more CpGs were considered.: "Only loci with a median read depth ≥100 across normal and tumour tissue samples and six or more CpGs were considered."
- [abstract] The total number of epialleles, the epiallele DNAm patterns, and a noise hyperparameter are all automatically inferred from the data.: "The total number of epialleles, the epiallele DNAm patterns, and a noise hyperparameter are all automatically inferred from the data."
- [readme] To install this R package use the devtools install_github command: library(devtools) install_github("james-e-barrett/bed"): "To install this R package use the devtools install_github command"
