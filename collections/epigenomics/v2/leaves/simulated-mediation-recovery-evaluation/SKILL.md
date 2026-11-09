---
name: simulated-mediation-recovery-evaluation
description: Use when you need to verify that a mediation-based cis driver ranking
  (e.g., WFTM/Sobel-based) actually recovers true underlying cis-trans mediations
  before trusting it on real SCNA/expression data.
when_to_use_negative:
- When real TCGA-like data are being analyzed for discovery — simulation is a validation
  step, not a driver-discovery method
- When sample sizes are small or correlation effects are weak — the Sobel test's high
  AUC was established under adequate sample size and strong correlation assumptions,
  so recovery metrics would be unreliable
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: iEDGE
  role: R package providing the pipeline (construct_iEDGE, run_iEDGE, calc_sobel,
    pruning) applied to simulated data with known mediations
  repo: https://github.com/montilab/iEDGE
- name: R
  role: Runtime environment for the simulation and evaluation scripts (rmarkdown notebook)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-019-52886-z
    title: Identification of candidate cancer drivers by integrative Epi-DNA and Gene
      Expression (iEDGE) data analysis
  source_content_hash: sha256:470cb6bdb30fde11f50a8458b31e41509f2c602e867928ddb7db8eda5361bd5e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/montilab/iEDGE
  tools:
  - Biobase
  - R
  - iEDGE
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# Simulated Mediation Recovery Evaluation

## Summary

Validates an iEDGE-style mediation pipeline by simulating input gene expression data with cis and trans genes having known mediation effects, then checking that the Sobel-test-based recovery reproduces those planted connections and matches the pruning step's significant results.

## When to use

Use when you need to verify that a mediation-based cis driver ranking (e.g., WFTM/Sobel-based) actually recovers true underlying cis-trans mediations before trusting it on real SCNA/expression data.

## When NOT to use

- When real TCGA-like data are being analyzed for discovery — simulation is a validation step, not a driver-discovery method
- When sample sizes are small or correlation effects are weak — the Sobel test's high AUC was established under adequate sample size and strong correlation assumptions, so recovery metrics would be unreliable

## Inputs

- Simulated gene expression matrix with known cis and trans gene mediations
- Binary epi-DNA alteration vector (e.g., copy-number presence/absence)
- iEDGE data object built via construct_iEDGE from the simulated matrices

## Outputs

- Sobel mediation statistics and FDR values per cn-cis-trans candidate connection
- Set of recovered significant cn-cis-trans connections comparable to planted ground truth
- Sobel test performance measures (e.g., AUC) for mediation recovery

## How to apply

Simulate the input gene expression matrix together with designated cis and trans genes that carry known mediation relationships with a binary epi-DNA alteration (copy number) vector, so the ground-truth cn-cis-trans connections are known by construction. Run the full iEDGE pipeline on the simulated object — differential expression (iEDGE_DE), Sobel mediation testing via calc_sobel(x, y, z, y.names, z.names), and pruning — then compute Sobel-test FDR statistics for each candidate mediator. Compare the set of recovered significant connections against the planted mediations, treating the simulations also as a benchmark for Sobel-test performance (AUC under adequate sample size and strong correlation assumptions). Finally, cross-check that recovered significant connections reproduce the pruning step's significant output (res$pruning$sig) so that in-silico recovery is consistent with the pipeline's own pruning criterion.

## Related tools

- **iEDGE** (R package providing the pipeline (construct_iEDGE, run_iEDGE, calc_sobel, pruning) applied to simulated data with known mediations) — https://github.com/montilab/iEDGE
- **R** (Runtime environment for the simulation and evaluation scripts (rmarkdown notebook))

## Examples

```
dat <- construct_iEDGE(cn, gep, cisgenes, transgenes = NA, cn.fdat, gep.fdat); res <- run_iEDGE(dat = dat, header = "testrun", outdir = ".")
```

## Evaluation signals

- Recovered significant cn-cis-trans connections reproduce the known simulated mediations planted in the synthetic data
- Recovered significant connections match the significant connections reported by the pruning step (res$pruning$sig)
- Sobel test achieves high AUC for uncovering true mediation effects under adequate sample size and strong correlation conditions
- Each SCNA-trans pair selects a single most significant cis mediator by Sobel test p-value, consistent with expected mediation structure

## Limitations

- Sobel test validity depends on adequate sample size and strong correlation effects, conditions not met in all datasets
- Assumption that the number of mediated trans genes proxies driver importance may not hold for specific use cases
- Simulation-based validation reflects the simulated effect strengths; real-world weaker effects may not be recovered as well

## Evidence

- [methods] simulate input gene expression, cis and trans genes with known mediations: "we will simulate the input gene expression data, the cis and trans genes with known mediations"
- [discussion] Simulation-based validation of Sobel test performance: "simulations were performed to validate the performance of the Sobel test of mediation"
- [discussion] Sobel AUC finding: "the Sobel test of mediation achieved high AUC for uncovering true mediation effects"
- [methods] calc_sobel invocation: "calc_sobel(x, y, z, y.names, z.names)"
