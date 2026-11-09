---
name: pharmacogenomic-chipseq-simulation
description: Use when benchmarking a TR–drug response identification method on synthetic
  pharmacogenomic/ChIP-seq data, or when evaluating robustness of such methods to
  added noise or spurious confounder-induced correlations.
when_to_use_negative:
- The goal is identifying TR–drug associations in real CCLE or liver cancer pharmacogenomic
  data rather than benchmarking method performance on synthetic data
- No reliable ChIP-seq target/RP score information is available — RePhine identification
  would be misled and the simulation benchmark does not address this regime
edam_operation: http://edamontology.org/operation_2478
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0204
tools:
- name: RePhine
  role: Method under test; its R package supplies simulation_code.R for generating
    synthetic TR/target expression, noise, and RP score data, and its pipeline (adjusted
    expression, partial correlations, likelihood ratio test uniP/multiP, modified
    GSEA filtering) is benchmarked against baselines
  repo: https://github.com/coexps/RePhine
- name: R
  role: Execution environment for simulation_code.R, the RePhine package, and the
    baseline logistic regression and correlation analyses
  repo: https://github.com/coexps/RePhine
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2019.09.008
    title: '<i>RePhine</i>: An Integrative Method for Identification of Drug Response-Related
      Transcriptional Regulators'
  source_content_hash: sha256:84e7ec5ef75a909bb483c4b08d7d3fbf632efc23b6eb3307c1f6972b79df67a5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/coexps/RePhine
  tools:
  - R
  - RePhine
  edam_operation: http://edamontology.org/operation_2478
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# pharmacogenomic-chipseq-simulation

## Summary

Reproduce the RePhine simulation benchmark that generates synthetic pharmacogenomic (gene expression + drug response) and ChIP-seq regulatory-potential data with added noises or confounders, then compares RePhine against Pearson correlation, logistic regression, and GSEA for identifying drug response-related transcriptional regulators.

## When to use

Use when benchmarking a TR–drug response identification method on synthetic pharmacogenomic/ChIP-seq data, or when evaluating robustness of such methods to added noise or spurious confounder-induced correlations.

## When NOT to use

- The goal is identifying TR–drug associations in real CCLE or liver cancer pharmacogenomic data rather than benchmarking method performance on synthetic data
- No reliable ChIP-seq target/RP score information is available — RePhine identification would be misled and the simulation benchmark does not address this regime

## Inputs

- simulation_code.R simulation script from the RePhine repository
- File S1 simulation scenario parameters (noises and confounders settings)
- Synthetic TR and target expression data, drug response (pharmacological profile) data, and RP scores
- R environment with packages glmnet, MASS, lmtest, ppcor, psych, ROCR

## Outputs

- simulation_benchmark.csv: per-scenario, per-method identification performance comparison table
- Simulated pharmacogenomic and ChIP-seq (RP score) datasets for noise and confounder scenarios

## How to apply

Obtain the RePhine R package (v1.1.0) and its simulation R script (simulation_code.R) from github.com/coexps/RePhine, plus the File S1 scenario parameters. Run simulation_code.R to synthesize TR and target expression data, noise data, and RP (regulatory potential) scores under the added-noise scenario, then re-run under the confounder scenario where spurious correlations not mediated by the TR are introduced. Apply RePhine to each simulated dataset alongside the three baselines (Pearson correlation analysis, logistic regression model, and gene set enrichment analysis) with identical inputs. Score all four methods by identification performance (detection of the true simulated TR–drug relationship versus false positives/negatives) and assemble per-scenario, per-method results into simulation_benchmark.csv.

## Related tools

- **RePhine** (Method under test; its R package supplies simulation_code.R for generating synthetic TR/target expression, noise, and RP score data, and its pipeline (adjusted expression, partial correlations, likelihood ratio test uniP/multiP, modified GSEA filtering) is benchmarked against baselines) — https://github.com/coexps/RePhine
- **R** (Execution environment for simulation_code.R, the RePhine package, and the baseline logistic regression and correlation analyses) — https://github.com/coexps/RePhine

## Examples

```
Rscript simulation_code.R
```

## Evaluation signals

- The benchmark table must show RePhine achieving higher identification performance than Pearson correlation, logistic regression, and GSEA in both the added-noise and added-confounder scenarios
- Simulated data must include TR/target expression, noise data, and RP scores as produced by simulation_code.R
- All four methods must receive identical simulated inputs per scenario to make the comparison fair
- Results must be reproducible from the published script (simulation_code.R) and scenario parameters in File S1

## Limitations

- RePhine identification is restricted to cases where reliable target information is available; lack of ChIP-seq data or targets with too few peaks would mislead the identification
- Correlations detected through targets may not be causal — upstream regulators or kinases influencing drug response and downstream TRs simultaneously can confound the inference
- The simulation does not address drug perturbation effects on genes, as post-treatment data are lacking

## Evidence

- [abstract] RePhine demonstrated an improved performance in comparison with three commonly used methods (including Pearson correlation analysis, logistic regression model, and gene set enrichment analysis): "RePhine demonstrated an improved performance in comparison with three commonly used methods (including Pearson correlation analysis, logistic regression model, and gene set enrichment analysis)"
- [readme] simulation_code.R is the simulation R code. it simulate TR and targets expression data, noise data and RP scores.: [evidence span withheld — anchor status: external_doc]
- [discussion] because RePhine identifies drug response-related TRs through the targets, it is restricted to the cases where reliable target information is available: "because RePhine identifies drug response-related TRs through the targets, it is restricted to the cases where reliable target information is available"
- [discussion] Lack of ChIP-seq data or ChIP-seq data with too few targets would mislead the identification of TRs: "Lack of ChIP-seq data or ChIP-seq data with too few targets would mislead the identification of TRs"
