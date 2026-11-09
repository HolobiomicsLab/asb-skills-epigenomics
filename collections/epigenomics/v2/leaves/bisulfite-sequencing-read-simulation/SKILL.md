---
name: bisulfite-sequencing-read-simulation
description: Use when validating or benchmarking an epiallele inference method on
  RRBS-style data and you need synthetic reads at a single locus with known ground-truth
  epialleles, mixing proportions, and a specified noise probability.
when_to_use_negative:
- The goal is analysis of real RRBS tumour data rather than controlled validation
  with known ground-truth epialleles
- The locus has fewer CpGs than the model's simulated setting (e.g. <6 CpGs) or the
  reads are already phased and error-free, so noise-corrupted simulation adds no benchmarking
  value
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3674
tools:
- name: R
  role: Environment in which the simulation of binary epialleles, read generation,
    and noise corruption is implemented
  repo: github.com/james-e-barrett
- name: devtools
  role: Used to install the bed R package (install_github) that both generates the
    benchmark context and fits the model to the simulated reads
  repo: github.com/james-e-barrett
- name: bed_fit
  role: Fitted to each simulated read matrix to assess whether the Bayesian model
    recovers the true number of epialleles, patterns, and noise level
  repo: github.com/james-e-barrett/bed
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1186/s12859-017-1753-2
    title: Quantification of tumour evolution and heterogeneity via Bayesian epiallele
      detection
  source_content_hash: sha256:7d1e208fb1915fbc7051a77c2b0ebd1ac024024979eaad9f24c67806756f1f62
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
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3674
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# bisulfite-sequencing-read-simulation

## Summary

Simulates binary bisulfite sequencing reads over a single locus to benchmark Bayesian epiallele detection: generating a read matrix from known underlying epialleles with controlled noise (e.g. ε=0.05) so that recovery of epiallele number, patterns, and noise level can be quantified.

## When to use

Use when validating or benchmarking an epiallele inference method on RRBS-style data and you need synthetic reads at a single locus with known ground-truth epialleles, mixing proportions, and a specified noise probability.

## When NOT to use

- The goal is analysis of real RRBS tumour data rather than controlled validation with known ground-truth epialleles
- The locus has fewer CpGs than the model's simulated setting (e.g. <6 CpGs) or the reads are already phased and error-free, so noise-corrupted simulation adds no benchmarking value

## Inputs

- Randomly generated binary epiallele definitions (Q=3 distinct DNAm patterns over d=6 CpG sites)
- Number of simulated reads per locus (N=100) and per-epiallele mixing proportions (~1/3 each)
- Noise/corruption probability ε=0.05 applied per CpG observation
- Number of simulation replicates (100)

## Outputs

- Binary N-by-d read matrices Y (100 reads x 6 CpGs) per replicate, 0=unmethylated, 1=methylated
- Ground-truth mapping of each read to its generating epiallele
- Per-replicate recovery records: inferred number of epialleles, inferred patterns, inferred noise level, and mean read-attribution success rate (e.g. simulation_recovery_results.)

## How to apply

In R, simulate N=100 sequencing reads over a locus of d=6 CpG sites by drawing from three distinct randomly generated binary epialleles (0 = unmethylated, 1 = methylated), each contributing on average one third of the reads. Corrupt each observed CpG status independently with noise probability ε=0.05 to mimic bisulfite sequencing errors, and repeat the simulation 100 times to characterise variability as in the paper. Pass each simulated N-by-d binary read matrix Y to the inference routine, then compare the inferred number of epialleles (AIC-selected), the inferred methylation patterns, and the inferred noise hyperparameter against the known generative values. The rationale is that a controlled simulation with known epiallele count, patterns, and noise separates model-recovery failure from data limitation. Quantify attribution accuracy by counting the proportion of reads assigned to their correct generating epiallele and averaging over simulations.

## Related tools

- **R** (Environment in which the simulation of binary epialleles, read generation, and noise corruption is implemented) — github.com/james-e-barrett
- **devtools** (Used to install the bed R package (install_github) that both generates the benchmark context and fits the model to the simulated reads) — github.com/james-e-barrett
- **bed_fit** (Fitted to each simulated read matrix to assess whether the Bayesian model recovers the true number of epialleles, patterns, and noise level) — github.com/james-e-barrett/bed

## Examples

```
library(devtools); install_github("james-e-barrett/bed"); ?bed_fit
```

## Evaluation signals

- The fitted model recovers Q=3 epialleles as the AIC minimum in (most) simulated replicates at N=100 reads and d=6 CpGs
- The inferred methylation patterns match the generating epialleles (e.g. via Hamming distance to truth)
- The inferred noise hyperparameter is close to the simulation value ε=0.05
- Approximately 95% of simulated reads are attributed to their correct underlying epiallele when averaged over the 100 simulations

## Limitations

- Recovery rates are established only for the simulated regime of N=100 reads, d=6 CpGs, three epialleles, and 5% noise; other regimes are not characterised here
- The study is limited to a single patient/tumour context, so simulated benchmarks do not address multi-patient variability

## Evidence

- [abstract] The total number of epialleles, the epiallele DNAm patterns, and a noise hyperparameter are all automatically inferred from the data.: "The total number of epialleles, the epiallele DNAm patterns, and a noise hyperparameter are all automatically inferred from the data."
- [methods] Initialise w by hierarchical clustering on Hamming distance: "Initialise w by using hierarchical clustering to group the observed reads Y into Q groups. The hamming distance"
- [methods] Instead we use the Akaike information criterion (AIC) in order to select the optimal number of epialleles: "Instead we use the Akaike information criterion (AIC) [18] in order to select the optimal number of epialleles"
- [readme] README installation instructions for the bed package: "To install this R package use the devtools install_github command: library(devtools) install_github("james-e-barrett/bed")"
