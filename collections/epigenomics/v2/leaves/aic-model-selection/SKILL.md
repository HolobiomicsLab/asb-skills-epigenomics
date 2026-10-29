---
name: aic-model-selection
description: Use when binary RRBS read matrices Y (reads x CpGs) at a locus must be
  partitioned into an unknown number of epialleles and Q must be chosen without manual
  specification or overfitting.
when_to_use_negative:
- Loci with fewer than six CpGs, median read depth below 100, more than 25% missing
  data, or on chromosomes X and Y
- Loci observed in fewer than two samples, since pooling reads across tissue samples
  is required to determine the number of distinct epialleles
- Analyses requiring phasing of reads between disjoint loci, which the approach cannot
  do
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
tools:
- name: R
  role: Language in which the BED package implementing bed_fit and the AIC selection
    loop is used
  repo: github.com/james-e-barrett
- name: devtools
  role: Used to install the bed package from GitHub via install_github before running
    model selection
- name: bed (bed_fit)
  role: Fits the Bayesian epiallele model at each candidate Q via MAP updates of w
    and X and noise estimation, yielding the log-likelihood for AIC computation
  repo: github.com/james-e-barrett/bed
provenance:
  source_task_ids:
  - task_003
  - task_001
  - task_002
  source_papers:
  - doi: 10.1186/s12859-017-1753-2
    title: Quantification of tumour evolution and heterogeneity via Bayesian epiallele
      detection
  source_content_hash: sha256:3bc3894bebc9f5cbde02d78fcb7ddc1f75c4bb18acd314b603f6169304fa4f76
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/github.com/james-e-barrett
  tools:
  - R
  - bed (bed_fit)
  - bed_fit
  - bed_marginal
  - devtools
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# aic-model-selection

## Summary

Selects the optimal number of Bayesian epialleles (Q) present at an RRBS locus by fitting the BED model across candidate Q values and minimising AIC(Q) = -2 log p(Y|X_hat, w_hat, epsilon_hat, Q) + 2*Q*d, where the Q*d epiallele matrix entries are counted as free parameters. This avoids overfitting while allowing the likelihood gain to justify extra epialleles.

## When to use

Use when binary RRBS read matrices Y (reads x CpGs) at a locus must be partitioned into an unknown number of epialleles and Q must be chosen without manual specification or overfitting.

## When NOT to use

- Loci with fewer than six CpGs, median read depth below 100, more than 25% missing data, or on chromosomes X and Y
- Loci observed in fewer than two samples, since pooling reads across tissue samples is required to determine the number of distinct epialleles
- Analyses requiring phasing of reads between disjoint loci, which the approach cannot do

## Inputs

- Binary sequencing read matrix Y (N reads by d CpG sites, 0 = unmethylated, 1 = methylated) per locus
- RRBS data from multiple tumour regions pooled per locus (e.g. seven CRUK0062 tumour samples plus one matched normal)
- Loci passing filters: six or more CpGs, median read depth >= 100, at most 25% missing data, chromosomes X/Y excluded

## Outputs

- Optimal number of epialleles Q_hat at each locus
- Full AIC(Q) table across candidate Q values (e.g. aic_model_selection.csv)
- MAP epiallele patterns X, read assignments w, and noise estimate epsilon = alpha0/(alpha0+alpha1)

## How to apply

Load the N-by-d binary read matrix Y for a locus (e.g. RRBS loci with six or more CpGs and median read depth >= 100 across the seven CRUK0062 tumour regions plus matched normal). For each candidate number of epialleles Q, run bed_fit(Y), which initialises read assignments w by hierarchical clustering with Hamming distance, alternates MAP updates of w (Eq. 4) and epiallele patterns X (Eq. 5) until convergence (typically two or three iterations), and estimates the noise epsilon as the proportion of mismatched CpGs (Eq. 6). For each fitted model compute AIC(Q) = -2 log p(Y | X_hat, w_hat, epsilon_hat, Q) + 2*Q*d, counting the Q*d entries of X as free parameters so a larger Q is retained only if the likelihood gain overcomes the complexity penalty. Select Q_hat = argmin_Q AIC(Q) and report the full AIC-versus-Q table to confirm a single clear minimum. Pool reads from all tissue samples at each locus before fitting, since the BED model borrows information across regions for greater statistical power.

## Related tools

- **R** (Language in which the BED package implementing bed_fit and the AIC selection loop is used) — github.com/james-e-barrett
- **devtools** (Used to install the bed package from GitHub via install_github before running model selection)
- **bed (bed_fit)** (Fits the Bayesian epiallele model at each candidate Q via MAP updates of w and X and noise estimation, yielding the log-likelihood for AIC computation) — github.com/james-e-barrett/bed

## Examples

```
library(devtools); install_github("james-e-barrett/bed"); fit <- bed_fit(Y); Q_hat <- which.min(sapply(Q_candidates, function(q) -2 * fit$loglik[q] + 2 * q * d))
```

## Evaluation signals

- The minimum of AIC(Q) identifies a single optimal Q_hat; report the full AIC-versus-Q table and confirm no ambiguous tie
- The iterative MAP procedure converges to a steady solution within a small number of iterations (typically two or three)
- At N=100 reads, d=6 CpGs, and 5% noise, the fitted model should attribute approximately 95% of reads to their correct epiallele
- Retained epialleles account for at least 5% of observed reads (smaller ones are discarded prior to computing phi_s)
- Inferred quantities (Q, epiallele DNAm patterns, noise hyperparameter) are recovered from data alone with no manual specification

## Limitations

- Reads cannot be phased between disjoint loci, though epiallele number and entropy may be correlated between close loci
- Case study was limited to a single patient (one tumour with seven regions plus one matched normal); larger multi-patient validation deferred to future work
- A larger Q is accepted only if the likelihood gain exceeds the 2*Q*d penalty, so weakly supported subclonal epialleles may be under-counted; epialleles below 5% read support are discarded

## Evidence

- [methods] AIC is used to select the optimal number of epialleles and thereby avoid overfitting the model.: "Instead we use the Akaike information criterion (AIC) [18] in order to select the optimal number of epialleles"
- [methods] MAP estimation alternates updates of w and X with hierarchical clustering initialisation by Hamming distance.: "Initialise w by using hierarchical clustering to group the observed reads Y into Q groups. The hamming distance"
- [methods] Noise epsilon is estimated as the proportion of mismatched CpGs during fitting.: [evidence span withheld — anchor status: not_found]
- [methods] The number of distinct epialleles is determined by pooling reads across all tissue samples.: "The number of distinct epialleles present at a particular locus is determined by pooling sequencing reads from all tissue samples"
- [readme] The bed R package is installed from GitHub with devtools.: "install_github("james-e-barrett/bed")"
- [abstract] The total number of epialleles, DNAm patterns, and noise hyperparameter are inferred automatically from the data.: "The total number of epialleles, the epiallele DNAm patterns, and a noise hyperparameter are all automatically inferred from the data."
