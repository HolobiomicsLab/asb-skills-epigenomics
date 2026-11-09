---
name: bayesian-epiallele-model-fitting
description: Use when you have an N-reads × d-CpGs binary methylation read matrix
  Y (0/1, RRBS-derived) for one locus and need to infer how many distinct epialleles
  are present, their methylation patterns, the noise level ε, and which epiallele
  each read originated from.
when_to_use_negative:
- Reads cannot be phased between disjoint loci — do not apply per-locus epiallele
  calls expecting linkage information across distant CpG sites
- Loci falling below the study filters (median read depth < 100, fewer than 6 CpGs,
  more than 25% missing data, or chromosomes X/Y) were excluded in the paper's workflow
- You need a single global methylation summary (e.g. mean methylation level) rather
  than resolving distinct epiallele species per locus
edam_operation: http://edamontology.org/operation_3893
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0091
tools:
- name: bed (bed_fit)
  role: 'Core fitting function: runs hierarchical-clustering initialisation of w,
    MAP updates of w and X, ε estimation, and AIC-based selection of Q'
  repo: github.com/james-e-barrett/bed
- name: bed (bed_marginal)
  role: Marginalises over the assignment parameter w to quantify per-read epiallele
    attribution uncertainty
  repo: github.com/james-e-barrett/bed
- name: devtools
  role: Used to install the bed package from GitHub via install_github
  repo: github.com/james-e-barrett/bed
- name: R
  role: Language in which the BED package and full analysis pipeline are implemented
  repo: github.com/james-e-barrett
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_006
  source_papers:
  - doi: 10.1186/s12859-017-1753-2
    title: Quantification of tumour evolution and heterogeneity via Bayesian epiallele
      detection
  source_content_hash: sha256:60188d9288bfeaa4d70e7cee0d7f336f2018513b83fed16758fbbd1fbc10b3b6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - bed (bed_fit)
  - bed (bed_marginal)
  - bed_fit
  - bed_marginal
  - devtools
  edam_operation: http://edamontology.org/operation_3893
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 18
---

# bayesian-epiallele-model-fitting

## Summary

Fit the Bayesian Epiallele Detection (BED) model to bisulfite sequencing reads at a single locus to automatically infer the number of epialleles Q, their binary DNAm patterns X, the noise hyperparameter ε, and per-read epiallele assignments, with read-level attribution uncertainty quantified by marginalising over w via bed_marginal.

## When to use

Use when you have an N-reads × d-CpGs binary methylation read matrix Y (0/1, RRBS-derived) for one locus and need to infer how many distinct epialleles are present, their methylation patterns, the noise level ε, and which epiallele each read originated from.

## When NOT to use

- Reads cannot be phased between disjoint loci — do not apply per-locus epiallele calls expecting linkage information across distant CpG sites
- Loci falling below the study filters (median read depth < 100, fewer than 6 CpGs, more than 25% missing data, or chromosomes X/Y) were excluded in the paper's workflow
- You need a single global methylation summary (e.g. mean methylation level) rather than resolving distinct epiallele species per locus

## Inputs

- Binary sequencing-read matrix Y (N reads by d CpG sites, 0 = unmethylated, 1 = methylated) for a single locus, e.g. from RRBS data
- Pooled reads from multiple tissue samples (tumour regions plus matched normal) for the same locus when the model borrows information across regions
- Pre-filtered loci: median read depth ≥ 100, ≥ 6 CpGs, ≤ 25% missing data, chromosomes X/Y discarded

## Outputs

- Inferred number of epialleles Q_hat (selected by minimum AIC)
- Epiallele methylation pattern matrix X (Q binary patterns of length d)
- Inferred noise hyperparameter ε (proportion of mismatched CpGs)
- Per-read epiallele assignments w and marginal posterior densities over w from bed_marginal
- Per-locus epiallele frequency vectors φ_s (after discarding epialleles < 5% of observed reads)

## How to apply

Install the 'bed' R package with devtools::install_github("james-e-barrett/bed") and pass the binary read matrix Y to bed_fit(Y). For each candidate number of epialleles Q, initialise read assignments w by hierarchical clustering of reads using Hamming distance, then alternate MAP updates of w (attributing each read y_i to the most similar epiallele, Eq. 4) and X (Eq. 5) until convergence, estimating ε as the proportion of mismatched CpGs (Eq. 6). Select the optimal number of epialleles by minimising AIC(Q) = -2 log L + 2*Q*d, counting the Q*d entries of X as free parameters so a larger Q is retained only if its likelihood gain overcomes the complexity penalty. Pool sequencing reads from all tissue samples at a locus to determine the number of distinct epialleles present, borrowing information across regions for greater statistical power. After fitting, call bed_marginal(Y, X, W) to obtain the marginal posterior density over w_i for each read, incorporating attribution uncertainty into downstream φ_s epiallele frequency distributions.

## Related tools

- **bed (bed_fit)** (Core fitting function: runs hierarchical-clustering initialisation of w, MAP updates of w and X, ε estimation, and AIC-based selection of Q) — github.com/james-e-barrett/bed
- **bed (bed_marginal)** (Marginalises over the assignment parameter w to quantify per-read epiallele attribution uncertainty) — github.com/james-e-barrett/bed
- **devtools** (Used to install the bed package from GitHub via install_github) — github.com/james-e-barrett/bed
- **R** (Language in which the BED package and full analysis pipeline are implemented) — github.com/james-e-barrett

## Examples

```
library(devtools); install_github("james-e-barrett/bed"); fit <- bed_fit(Y); marg <- bed_marginal(Y, fit$X, fit$W)
```

## Evaluation signals

- The AIC(Q) table across candidate Q values exhibits a single clear minimum identifying Q_hat; report the full table
- In simulation recovery (N=100 reads, d=6 CpGs, three epialleles, ε=0.05), the model recovers all three epialleles, their patterns, and the noise level, with approximately 95% of reads attributed to their correct underlying epiallele
- Estimated ε should be close to the true corruption rate (e.g. ~0.05 in the paper's simulations) and consistent with the proportion of mismatched CpGs
- Per-read marginal posteriors from bed_marginal should concentrate near 0/1 for unambiguous reads; low-confidence reads indicate overlapping epiallele patterns
- Output epiallele patterns X are binary vectors of length d, and the selected Q*d parameter count matches the AIC penalty term

## Limitations

- Reads cannot be phased between disjoint loci, so epiallele calls are per-locus; only correlation between close loci may be observed
- The model was validated on a single patient (seven regions of one lung tumour, CRUK0062, plus one matched normal); larger multi-patient studies were deferred to future work
- The number of epialleles Q is chosen via AIC rather than full Bayesian model averaging, so it depends on the 2*Q*d penalty balancing likelihood gain
- Reads overlapping epiallele patterns carry irreducible attribution uncertainty, handled only by marginalising over w rather than resolved outright

## Evidence

- [methods] Initialise w by using hierarchical clustering to group the observed reads Y into Q groups. The hamming distance: "Initialise w by using hierarchical clustering to group the observed reads Y into Q groups. The hamming distance"
- [methods] For fixed X,ε, and Q, the maximum a posteriori (MAP) estimate for w is given by attributing each read y: "For fixed X,ε, and Q, the maximum a posteriori (MAP) estimate for w is given by attributing each read y"
- [methods] Instead we use the Akaike information criterion (AIC) [18] in order to select the optimal number of epialleles: "Instead we use the Akaike information criterion (AIC) [18] in order to select the optimal number of epialleles"
- [methods] Only loci with a median read depth ≥100 across normal and tumour tissue samples and six or more CpGs were considered.: "Only loci with a median read depth ≥100 across normal and tumour tissue samples and six or more CpGs were considered."
- [abstract] The total number of epialleles, the epiallele DNAm patterns, and a noise hyperparameter are all automatically inferred from the data.: "The total number of epialleles, the epiallele DNAm patterns, and a noise hyperparameter are all automatically inferred from the data."
- [readme] library(devtools)
    install_github("james-e-barrett/bed"): "library(devtools)
    install_github("james-e-barrett/bed")"
