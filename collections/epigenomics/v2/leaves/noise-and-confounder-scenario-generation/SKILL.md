---
name: noise-and-confounder-scenario-generation
description: 'Use when stress-testing a TR–drug response identification method under
  controlled synthetic conditions: you need simulated TR/target expression, drug response,
  and RP scores with injected noise or spurious (non-TR-mediated) correlations to
  compare against baseline methods.'
when_to_use_negative:
- Your goal is real-data validation — RePhine's discussion notes that 'TR target analyses
  and TR knockout or activation experiments combined with drug response examination
  are also required to validate the causal relationship', which simulation cannot
  provide
- 'You lack reliable target information: the article states RePhine ''is restricted
  to the cases where reliable target information is available'', so simulated RP-score
  scenarios poorly proxy target-free settings'
- You need to model post-treatment/acquired-resistance dynamics; the article notes
  such effects 'could not be integrated due to lack of post-treatment data', and the
  simulation scripts do not cover them
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0091
tools:
- name: RePhine
  role: R package supplying simulation_code.R that simulates TR/target expression,
    noise data, and RP scores; the method-under-test in the benchmark
  repo: https://github.com/coexps/RePhine
- name: R
  role: Execution environment for the simulation scripts and for running RePhine and
    baseline methods (Pearson correlation, logistic regression, GSEA) on simulated
    data
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2019.09.008
    title: '<i>RePhine</i>: An Integrative Method for Identification of Drug Response-Related
      Transcriptional Regulators'
  source_content_hash: sha256:f700ad3a6b2d6c8184c5a4346ad805cba0d444cd5e47f5ff4bfc7474afc2776f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/coexps/RePhine
  tools:
  - R
  - RePhine
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# noise-and-confounder-scenario-generation

## Summary

Generates synthetic pharmacogenomic benchmark scenarios with added noise or confounding factors, as used to benchmark RePhine against Pearson correlation, logistic regression, and GSEA. Grounded in the simulation R scripts (simulation_code.R) shipped with the RePhine package and the scenario parameters in File S1.

## When to use

Use when stress-testing a TR–drug response identification method under controlled synthetic conditions: you need simulated TR/target expression, drug response, and RP scores with injected noise or spurious (non-TR-mediated) correlations to compare against baseline methods.

## When NOT to use

- Your goal is real-data validation — RePhine's discussion notes that 'TR target analyses and TR knockout or activation experiments combined with drug response examination are also required to validate the causal relationship', which simulation cannot provide
- You lack reliable target information: the article states RePhine 'is restricted to the cases where reliable target information is available', so simulated RP-score scenarios poorly proxy target-free settings
- You need to model post-treatment/acquired-resistance dynamics; the article notes such effects 'could not be integrated due to lack of post-treatment data', and the simulation scripts do not cover them

## Inputs

- Scenario parameters from File S1 (simulation settings for noise and confounder scenarios)
- simulation_code.R script from the coexps/RePhine repository
- R environment with packages glmnet, MASS, lmtest, ppcor, psych, ROCR (per README prerequisites)

## Outputs

- Synthetic TR and target expression matrices with simulated drug response under the added-noise scenario
- Synthetic data with confounder-induced spurious correlations under the added-confounders scenario
- Simulated RP scores (regulatory potential scores)
- Per-scenario, per-method performance comparison table (e.g. simulation_benchmark.csv)

## How to apply

Obtain the RePhine repository (github.com/coexps/RePhine), which provides simulation_code.R — the script that simulates TR and target expression data, noise data, and RP scores — together with the scenario parameters defined in the article's File S1. Run the simulation scripts once under the added-noise scenario to generate synthetic pharmacogenomic data (gene expression plus drug response) and simulated ChIP-seq-derived RP scores. Re-run the scripts under the added-confounders scenario to inject spurious correlations not mediated by the transcriptional regulator. Feed each simulated dataset identically to the method under test (RePhine) and the three baselines (Pearson correlation analysis, logistic regression model, gene set enrichment analysis), then score detection of the true simulated TR–drug relationship versus false positives/negatives and assemble per-scenario, per-method results into a comparison table (e.g. simulation_benchmark.csv). The rationale is that RePhine's integration of ChIP-seq targets and adjustment of confounding factors should preserve identification power where mRNA-only approaches fail.

## Related tools

- **RePhine** (R package supplying simulation_code.R that simulates TR/target expression, noise data, and RP scores; the method-under-test in the benchmark) — https://github.com/coexps/RePhine
- **R** (Execution environment for the simulation scripts and for running RePhine and baseline methods (Pearson correlation, logistic regression, GSEA) on simulated data)

## Examples

```
Rscript simulation_code.R  # run once for the noise scenario, then re-run with File S1 confounder parameters
```

## Evaluation signals

- The benchmark table shows RePhine achieving higher identification performance than Pearson correlation, logistic regression, and GSEA under BOTH the added-noise and added-confounder scenarios
- Simulated data contain the true TR–drug relationship (from simulation_code.R ground truth) so true/false positives and negatives can be counted per method
- The confounder scenario demonstrably introduces spurious correlations not mediated by the TR (baselines degrade, RePhine adjusts them out)
- Simulated RP scores are generated alongside expression data so the ChIP-seq target-inference channel of RePhine is exercised
- Per-scenario, per-method results are assembled into a single comparison file (simulation_benchmark.csv) covering all 4 methods × 2 scenarios

## Limitations

- Simulation validates noise/confounder robustness only; the article states experimental validation via 'TR knockout or activation experiments combined with drug response examination are also required to validate the causal relationship'
- Simulated correlations may not be causal — 'the relationship may not be causal. Some upstream regulators or kinases may exist'
- The simulation assumes RP scores are available; 'Lack of ChIP-seq data or ChIP-seq data with too few targets would mislead the identification of TRs'

## Evidence

- [readme] simulation_code.R is the simulation R code. it simulate TR and targets expression data, noise data and RP scores.: [evidence span withheld — anchor status: external_doc]
- [abstract] RePhine outperformed baselines on noisy/confounded simulated data: "RePhine demonstrated an improved performance in comparison with three commonly used methods (including Pearson correlation analysis, logistic regression model, and gene set enrichment analysis)"
- [discussion] Experimental validation still required to establish causality: "TR target analyses and TR knockout or activation experiments combined with drug response examination are also required to validate the causal relationship"
- [readme] R packages required to run the simulation and analysis pipeline: [evidence span withheld — anchor status: external_doc]
