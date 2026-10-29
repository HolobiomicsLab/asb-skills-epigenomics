---
name: bayesian-latent-variable-model-fitting
description: Use when bisulfite-sequencing reads from multiple samples of the same
  tumour must be decomposed into latent epialleles at a locus, and Q, methylation
  patterns, and noise must be inferred from data rather than fixed a priori.
when_to_use_negative:
- Reads cannot be phased between disjoint loci, so do not use this to phase methylation
  patterns across separate loci — the model is locus-local.
- Do not apply when a locus fails the eligibility filters (fewer than six CpGs, median
  read depth below 100, or more than 25% missing data), as the model was validated
  only on filtered loci.
- 'Do not treat per-locus inference as evidence for multi-patient or multi-tumour
  generalisation: the case study is a single patient (one tumour, seven regions, one
  matched normal).'
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
tools:
- name: R
  role: Implementation language of the bed package for Bayesian epiallele detection
  repo: github.com/james-e-barrett
- name: bed_fit
  role: 'Fits the Bayesian epiallele model to the pooled read matrix Y: hierarchical-clustering
    initialisation of w, iterative MAP updates of w and X, and AIC-based selection
    of Q'
  repo: github.com/james-e-barrett/bed
- name: bed_marginal
  role: Computes the posterior marginal over read assignments w and enables per-sample
    epiallele distributions phi_s
  repo: github.com/james-e-barrett/bed
- name: devtools
  role: Used to install the bed package from GitHub via install_github
  repo: github.com/james-e-barrett/bed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-017-1753-2
    title: Quantification of tumour evolution and heterogeneity via Bayesian epiallele
      detection
  source_content_hash: sha256:efd329f76b5ad454631363786105714d5ffcde88ddb13c439047781bbbde7d5e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/github.com/james-e-barrett
  tools:
  - R
  - bed_fit
  - bed_marginal
  - devtools
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# bayesian-latent-variable-model-fitting

## Summary

Fits a Bayesian epiallele detection (BED) latent-variable model to pooled RRBS read matrices, automatically inferring the number of epialleles Q, epiallele DNAm patterns X, read assignments w, and a noise hyperparameter epsilon. Implemented in the R package 'bed' and applied to multi-region lung tumour (CRUK0062) plus matched normal data.

## When to use

Use when bisulfite-sequencing reads from multiple samples of the same tumour must be decomposed into latent epialleles at a locus, and Q, methylation patterns, and noise must be inferred from data rather than fixed a priori.

## When NOT to use

- Reads cannot be phased between disjoint loci, so do not use this to phase methylation patterns across separate loci — the model is locus-local.
- Do not apply when a locus fails the eligibility filters (fewer than six CpGs, median read depth below 100, or more than 25% missing data), as the model was validated only on filtered loci.
- Do not treat per-locus inference as evidence for multi-patient or multi-tumour generalisation: the case study is a single patient (one tumour, seven regions, one matched normal).

## Inputs

- RRBS read matrices Y (methylation states of sequencing reads) at each eligible locus, pooled across the seven CRUK0062 tumour regions and one matched normal sample
- Locus eligibility filters: >= 6 CpGs per locus, median read depth >= 100 across samples, <= 25% missing data per locus, autosomes only (chromosomes X and Y discarded)

## Outputs

- Inferred epiallele methylation patterns X
- Optimal number of epialleles Q selected by minimum AIC
- MAP noise hyperparameter estimate epsilon = alpha0/(alpha0+alpha1)
- Posterior marginals over read assignments w from bed_marginal
- Per-sample epiallele distributions phi_s saved to epiallele_inference_results.rds

## How to apply

At each eligible RRBS locus (six or more CpGs, median read depth >= 100 across samples, at most 25% missing data, chromosomes X/Y discarded), pool reads from all seven tumour regions and the matched normal sample into a read matrix Y to borrow statistical power across regions. Call bed_fit(Y), which initialises read assignments w via hierarchical clustering using Hamming distance, then iterates MAP updates of w and the epiallele patterns X until convergence (typically two or three iterations), selecting the optimal number of epialleles Q by minimising AIC(Q) = -2 log p(Y|X,w,epsilon,Q) + 2Qd. Compute the MAP noise estimate epsilon = alpha0/(alpha0+alpha1) and discard epialleles accounting for less than 5% of observed reads. Then run bed_marginal(Y, X, W) to obtain the posterior marginal over w and compute per-sample epiallele distributions phi_s, the proportion of reads per sample attributed to each retained epiallele. This marginalisation explicitly incorporates uncertainty as to which epiallele each observed read originated from.

## Related tools

- **R** (Implementation language of the bed package for Bayesian epiallele detection) — github.com/james-e-barrett
- **bed_fit** (Fits the Bayesian epiallele model to the pooled read matrix Y: hierarchical-clustering initialisation of w, iterative MAP updates of w and X, and AIC-based selection of Q) — github.com/james-e-barrett/bed
- **bed_marginal** (Computes the posterior marginal over read assignments w and enables per-sample epiallele distributions phi_s) — github.com/james-e-barrett/bed
- **devtools** (Used to install the bed package from GitHub via install_github) — github.com/james-e-barrett/bed

## Examples

```
library(devtools); install_github("james-e-barrett/bed"); fit <- bed_fit(Y); marg <- bed_marginal(Y, fit$X, fit$W)
```

## Evaluation signals

- The iterative MAP procedure must converge to a steady solution, typically within two or three iterations.
- The retained model must correspond to the minimum AIC score across candidate Q values (AIC(Q) = -2 log p(Y|X,w,epsilon,Q) + 2Qd).
- All epialleles retained in the final output must account for at least 5% of observed reads.
- Per-sample distributions phi_s must be proper proportions (each attributable fraction between 0 and 1, summing to 1 across retained epialleles per sample).
- The noise estimate must equal the MAP value epsilon = alpha0/(alpha0+alpha1).

## Limitations

- It is not possible to phase reads between disjoint loci, although the number of epialleles and the entropy may be correlated between close loci.
- The study is limited to a single patient (one tumour with seven regions plus one matched normal); larger multi-patient studies are deferred to future work.
- Whether epiallele-level disorder quantification provides a more refined measure of tumour aggressiveness remains unverified.
- The discussion section provides no separate software/implementation name or availability statement; the code is documented only in the abstract/methods and README.

## Evidence

- [abstract] The total number of epialleles, the epiallele DNAm patterns, and a noise hyperparameter are all automatically inferred from the data.: "The total number of epialleles, the epiallele DNAm patterns, and a noise hyperparameter are all automatically inferred from the data."
- [methods] AIC model selection for the number of epialleles: "Instead we use the Akaike information criterion (AIC) [18] in order to select the optimal number of epialleles"
- [methods] Hierarchical clustering initialisation with Hamming distance: "Initialise w by using hierarchical clustering to group the observed reads Y into Q groups. The hamming distance"
- [methods] Locus eligibility filters: "Only loci with a median read depth ≥100 across normal and tumour tissue samples and six or more CpGs were considered."
- [methods] Discard low-prevalence epialleles before computing phi_s: [evidence span withheld — anchor status: not_found]
- [readme] Package installation and documented entry point: "To install this R package use the devtools install_github command:

    library(devtools)
    install_github("james-e-barrett/bed")"
