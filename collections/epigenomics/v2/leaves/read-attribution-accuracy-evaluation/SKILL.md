---
name: read-attribution-accuracy-evaluation
description: Use when validating BED (Bayesian Epiallele Detection) on simulated bisulfite-sequencing
  reads with known epialleles, to measure the fraction of reads attributed to the
  correct underlying epiallele before trusting model output on real RRBS tumour data.
when_to_use_negative:
- Real RRBS data where the true generating epiallele of each read is unknown — no
  correctness proportion can be computed, only posterior marginals
- Loci with fewer than six CpGs or median read depth below 100, which the article's
  filtering excludes from consideration
- Phasing epialleles across disjoint loci — the article states it is not possible
  to phase reads between disjoint loci
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: bed_fit
  role: Fits the Bayesian epiallele model to the simulated read matrix Y, inferring
    the number of epialleles, their methylation patterns, and the noise level via
    MAP estimation and AIC model selection
  repo: https://github.com/james-e-barrett/bed
- name: bed_marginal
  role: Computes the marginal density over the assignment parameter w to attribute
    each read to an epiallele with quantified uncertainty
  repo: https://github.com/james-e-barrett/bed
- name: devtools
  role: Used to install the BED R package via install_github("james-e-barrett/bed")
  repo: https://github.com/james-e-barrett/bed
- name: R
  role: Language in which the simulation, fitting, marginalisation, and success-rate
    computation are implemented
  repo: https://github.com/james-e-barrett
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-017-1753-2
    title: Quantification of tumour evolution and heterogeneity via Bayesian epiallele
      detection
  source_content_hash: sha256:9d4dbbd10de087598521d510385146e9572c5c69b76aa5890c9e978b3302c3b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/james-e-barrett/bed
  tools:
  - R
  - bed_fit
  - bed_marginal
  - devtools
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# read-attribution-accuracy-evaluation

## Summary

A validation skill for the BED Bayesian epiallele detection model that quantifies, via repeated simulations with known ground-truth epialleles, the proportion of observed sequencing reads correctly attributed to their underlying epiallele using bed_marginal posterior attribution.

## When to use

Use when validating BED (Bayesian Epiallele Detection) on simulated bisulfite-sequencing reads with known epialleles, to measure the fraction of reads attributed to the correct underlying epiallele before trusting model output on real RRBS tumour data.

## When NOT to use

- Real RRBS data where the true generating epiallele of each read is unknown — no correctness proportion can be computed, only posterior marginals
- Loci with fewer than six CpGs or median read depth below 100, which the article's filtering excludes from consideration
- Phasing epialleles across disjoint loci — the article states it is not possible to phase reads between disjoint loci

## Inputs

- Simulated read matrix Y (100 reads × 6 CpG binary methylation states) with known ground-truth epialleles and mixture weights (~1/3 each)
- Ground-truth noise level ε = 0.05 used to corrupt CpG statuses
- BED R package installed from github.com/james-e-barrett/bed

## Outputs

- Mean read-attribution success rate across simulations (proportion of reads assigned to correct epiallele)
- Inferred number of epialleles (expected to recover Q=3)
- Inferred binary methylation patterns of each epiallele
- Inferred noise hyperparameter ε
- Per-read posterior marginal attribution uncertainties from bed_marginal(Y, X, W)
- Results file (e.g. simulation_recovery_results.)

## How to apply

Simulate a read matrix Y of 100 sequencing reads over a locus of d=6 CpG sites drawn from three randomly generated binary epialleles, each contributing about one third of reads, and corrupt each observed CpG status with noise probability ε=0.05; repeat over ~100 simulations as in the paper. For each simulation, run bed_fit(Y) — which initialises read assignments by hierarchical clustering with Hamming distance and selects the number of epialleles via AIC(Q) = -2 log L + 2Qd — then call bed_marginal(Y, X, W) to marginalise over the assignment parameter w and quantify attribution uncertainty per read. Attribute each read to the epiallele with the highest posterior marginal, count the proportion of reads assigned to their correct generating epiallele, and average this success rate across simulations. Record the recovered number of epialleles, their inferred methylation patterns, the inferred noise level, and the mean attribution success rate as correctness evidence.

## Related tools

- **bed_fit** (Fits the Bayesian epiallele model to the simulated read matrix Y, inferring the number of epialleles, their methylation patterns, and the noise level via MAP estimation and AIC model selection) — https://github.com/james-e-barrett/bed
- **bed_marginal** (Computes the marginal density over the assignment parameter w to attribute each read to an epiallele with quantified uncertainty) — https://github.com/james-e-barrett/bed
- **devtools** (Used to install the BED R package via install_github("james-e-barrett/bed")) — https://github.com/james-e-barrett/bed
- **R** (Language in which the simulation, fitting, marginalisation, and success-rate computation are implemented) — https://github.com/james-e-barrett

## Examples

```
library(devtools); install_github("james-e-barrett/bed"); fit <- bed_fit(Y); marg <- bed_marginal(Y, fit$X, fit$W)
```

## Evaluation signals

- The inferred number of epialleles equals the known simulated value (Q=3) across simulations, i.e. AIC-based selection recovers the true Q
- The inferred noise hyperparameter ε is close to the simulated truth of 0.05
- Mean read-attribution success rate is high (reads attributed to their correct underlying epiallele the majority of the time) when averaged over 100 simulations
- Inferred epiallele binary methylation patterns match the three generating patterns (allowing read-level noise corruption)
- Output JSON contains the four required records: inferred number of epialleles, inferred patterns, inferred noise level, and mean success rate

## Limitations

- Uncertainty as to which epiallele a read originated from is inherent; misattribution of noisy reads is expected and only marginalised over, not eliminated
- Simulation is restricted to a single locus with a small number of epialleles (three) and fixed noise (ε=0.05); performance at other depths, loci sizes, or noise levels is not established here
- Epialleles accounting for less than 5% of observed reads may be discarded, biasing recovery of rare epialleles

## Evidence

- [abstract] The total number of epialleles, the epiallele DNAm patterns, and a noise hyperparameter are all automatically inferred from the data.: "The total number of epialleles, the epiallele DNAm patterns, and a noise hyperparameter are all automatically inferred from the data."
- [abstract] Uncertainty as to which epiallele a read originated from is explicitly incorporated by marginalising over the appropriate posterior densities.: "Uncertainty as to which epiallele an observed sequencing read originated from is explicitly incorporated by marginalising over the appropriate posterior densities."
- [methods] MAP estimation attributes each read to the most similar epiallele.: "the maximum a posteriori (MAP) estimate for w is given by attributing each read y
i to the epiallele that is most similar to it."
- [methods] Hierarchical clustering with Hamming distance initialises w.: "Initialise w by using hierarchical clustering to group the observed reads Y into Q groups. The hamming distance"
- [methods] AIC is used to select the optimal number of epialleles.: "Instead we use the Akaike information criterion (AIC) [18] in order to select the optimal number of epialleles"
- [methods] bed_marginal computes the marginal density over w.: "The marginal density over w
i is given by fixing all other parameters to their MAP values"
- [readme] README gives the install command for the BED package.: "install_github("james-e-barrett/bed")"
