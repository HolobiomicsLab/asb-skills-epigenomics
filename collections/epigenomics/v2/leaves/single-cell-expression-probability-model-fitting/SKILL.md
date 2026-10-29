---
name: single-cell-expression-probability-model-fitting
description: Use when fitting/validating scPower's expression probability priors on
  a new atlas count matrix (e.g., Drop-seq, Smart-seq2, 10X) to verify predicted expressed
  gene numbers match observed counts before power analysis.
when_to_use_negative:
- The goal is simulation-based power analysis rather than analytic expression probability
  fitting (simulation tools like muscat and powsimR have long runtimes and high memory
  requirements, e.g., 35-48 GB for 20 samples x 3000 cells)
- The input is effect size priors for DEG/eQTL power (use scPower's de.ref.study reference
  studies such as Pancreas_alphabeta or 10X_PBMC_1 instead of fitting expression priors)
- The data are not single-cell count matrices (scPower priors require cell-level counts
  with donor and cell-type annotations; continuous annotations like pseudo time would
  need to be discretized first)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scPower
  role: R package providing create.pseudobulk, nbinom.estimation, mixed.gamma.estimation,
    meanUMI.calculation, convert.gamma.parameters, umi.read.relation, estimate.exp.prob.param,
    and calculate.gene.counts for expression probability model fitting and validation
  repo: https://github.com/heiniglab/scPower
- name: R
  role: Implementation language; scPower is an open source R package installed via
    devtools
  repo: https://github.com/heiniglab/scPower
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-26779-7
    title: scPower accelerates and optimizes the design of multi-sample single cell
      transcriptomic studies
  source_content_hash: sha256:595e7617d31038a69f058040e8e6196e80736c8cd755e836b0e69da220abdc62
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/heiniglab/scPower
  tools:
  - R
  - scPower
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# single-cell-expression-probability-model-fitting

## Summary

Fit scPower's expression probability model to single-cell RNA-seq count matrices from droplet-based (UMI) or read-count (Smart-seq2) technologies, then validate predicted expressed gene numbers against observed counts (r2 ~0.995 for Drop-seq lung, ~0.991 for Smart-seq2 pancreas).

## When to use

Use when fitting/validating scPower's expression probability priors on a new atlas count matrix (e.g., Drop-seq, Smart-seq2, 10X) to verify predicted expressed gene numbers match observed counts before power analysis.

## When NOT to use

- The goal is simulation-based power analysis rather than analytic expression probability fitting (simulation tools like muscat and powsimR have long runtimes and high memory requirements, e.g., 35-48 GB for 20 samples x 3000 cells)
- The input is effect size priors for DEG/eQTL power (use scPower's de.ref.study reference studies such as Pancreas_alphabeta or 10X_PBMC_1 instead of fitting expression priors)
- The data are not single-cell count matrices (scPower priors require cell-level counts with donor and cell-type annotations; continuous annotations like pseudo time would need to be discretized first)

## Inputs

- Atlas count matrices for lung (droplet/Drop-seq) and pancreas (Smart-seq2) with donor and cell-type annotation data frames whose row ordering matches count matrix columns
- Technology-specific parameters: mean UMI per cell for droplet data (meanUMI.calculation); gene-length normalized count threshold (min.norm.count=3) and dispersion function for Smart-seq2
- Expression thresholds: min.counts=3, perc.indiv=0.5, num.genes.kept=21000

## Outputs

- Fitted gamma mixed distribution parameters (zero component plus two left-censored gamma components) per read depth
- Per-technology r2 table of predicted vs. observed expressed gene counts (expression_model_validation_r2.csv), targeting r2=0.995 (Drop-seq lung) and r2=0.991 (Smart-seq2 pancreas)

## How to apply

Reformat the 2D count matrix plus donor/cell-type annotation into a 3D pseudobulk matrix with create.pseudobulk (ensuring annotation rows match count matrix column ordering). Estimate per-gene negative binomial mean and dispersion with nbinom.estimation (DESeq-based normalization) per read-depth subsample, then fit a gamma mixed distribution (zero component plus two left-censored gamma components) via mixed.gamma.estimation with num.genes.kept=21000 and censoredPoint=1/num_cells_per_celltype. For droplet (UMI) data, compute mean UMI per cell with meanUMI.calculation, convert gamma parameters to mean-sd parameterization with convert.gamma.parameters, fit the read-UMI relation with umi.read.relation, and model doublets with a constant multiplet rate (multipletRateGrowth="constant"). For Smart-seq2 (read-count) data, parameterize the gamma-mixed distribution directly over read depth, apply the gene-length normalized threshold min.norm.count=3 and the read-depth-dependent median dispersion function (disp.fun.param.smart). Predict expressed gene numbers with estimate.exp.prob.param using min.counts=3 and perc.indiv=0.5, merge with observed counts from calculate.gene.counts, and compute r2 between predicted and observed expressed genes (targets: ~0.995 lung, ~0.991 pancreas, Supplementary Table S3), writing results to expression_model_validation_r2.csv.

## Related tools

- **scPower** (R package providing create.pseudobulk, nbinom.estimation, mixed.gamma.estimation, meanUMI.calculation, convert.gamma.parameters, umi.read.relation, estimate.exp.prob.param, and calculate.gene.counts for expression probability model fitting and validation) — https://github.com/heiniglab/scPower
- **R** (Implementation language; scPower is an open source R package installed via devtools) — https://github.com/heiniglab/scPower

## Examples

```
library(scPower); pseudobulk.lung <- create.pseudobulk(count.matrix, annotation, sample.column="donor", celltype.column="celltype"); params <- nbinom.estimation(pseudobulk.lung, depth.subsamples)
```

## Evaluation signals

- r2 between predicted and observed expressed gene numbers reaches ~0.995 for Drop-seq lung data and ~0.991 for Smart-seq2 pancreas data (Supplementary Table S3)
- Annotation data frame rows match count matrix column ordering before create.pseudobulk is called
- Gamma mixed fit uses num.genes.kept=21000 and censoredPoint=1/num_cells_per_celltype; droplet doublets modeled with multipletRateGrowth="constant"
- Smart-seq2 predictions use gene-length normalized threshold min.norm.count=3 and the read-depth-dependent median dispersion function (disp.fun.param.smart)
- Per-technology r2 values written to expression_model_validation_r2.csv

## Limitations

- Data-driven priors only allow reliable assessment of overall power for sample sizes smaller or roughly equal to the pilot data sets; larger sample sizes may identify new significant DEGs with lower effect sizes not captured
- Actual experiments might result in higher doublet rates and lower numbers of usable cells than the default assumption
- Continuous cell annotations such as pseudo time must be discretized before power analysis

## Evidence

- [methods] reformating the 2d count matrix into a 3d pseudobulk matrix, using the function "create.pseudobulk": "reformating the 2d count matrix into a 3d pseudobulk matrix, using the function "create.pseudobulk""
- [methods] the negative binomial fit for each gene, i.e. the mean and disperion parameter, is estimated with "nbinom.estimation": "the negative binomial fit for each gene, i.e. the mean and disperion parameter, is estimated with "nbinom.estimation""
- [methods] gamma.fit.temp<-mixed.gamma.estimation(norm.mean.values.temp$mean, num.genes.kept = 21000, censoredPoint = censoredPoint): "gamma.fit.temp<-mixed.gamma.estimation(norm.mean.values.temp$mean, num.genes.kept = 21000, censoredPoint = censoredPoint)"
- [methods] A mixed distribution of a zero component and two left zensored gamma components is fitted for each mean value vector: "A mixed distribution of a zero component and two left zensored gamma components is fitted for each mean value vector"
- [readme] The tool was evaluated with data from different tissues and single cell technologies, based on UMI counts and read counts.: "The tool was evaluated with data from different tissues and single cell technologies, based on UMI counts and read counts."
- [methods] the expressed genes are extracted using the function "calculate.gene.counts": "the expressed genes are extracted using the function "calculate.gene.counts""
