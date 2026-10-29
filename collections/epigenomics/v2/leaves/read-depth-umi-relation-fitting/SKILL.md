---
name: read-depth-umi-relation-fitting
description: 'Use when fitting scPower expression priors from pilot data: you have
  per-matrix mean UMI counts and corresponding cellranger mapped-read counts (e.g.
  23130, 17422, 11666, 5859 from full and subsampled PBMC matrices) and need a read-depth-to-UMI
  curve for power estimation.'
when_to_use_negative:
- The analysis goal is DEG/eQTL power estimation with pre-existing expression priors
  (e.g. the package's 10X_PBMC_1 reference study) — no new read-depth-to-UMI fit is
  needed
- Input data are full-length read counts rather than UMI counts, where the UMI-based
  depth relationship does not apply without re-estimating priors
- You only need pseudobulk reformating or gamma mixed-distribution fitting (create.pseudobulk
  / mixed.gamma.estimation) — those are separate upstream steps
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scPower
  role: R package providing meanUMI.calculation and umi.read.relation to compute per-matrix
    mean UMI counts and fit the logarithmic read-depth-to-UMI relationship stored
    in the expression prior
  repo: https://github.com/heiniglab/scPower
- name: R
  role: Runtime environment in which the scPower fitting workflow is executed and
    the prior is serialized as expression_prior.RData
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-26779-7
    title: scPower accelerates and optimizes the design of multi-sample single cell
      transcriptomic studies
  source_content_hash: sha256:c930978908bac8c9bd5cd015f84427bdde2c73ffc4a303800fbe0399b2d4ad49
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/heiniglab/scPower
  tools:
  - DESeq
  - R
  - estimate.exp.prob.param
  - scPower
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# read-depth-umi-relation-fitting

## Summary

Fits the logarithmic relationship between sequencing depth (mapped reads per cell) and mean UMI counts per cell using scPower, so that a planned experiment's read depth can be translated into the mean UMI input required by the expression probability model.

## When to use

Use when fitting scPower expression priors from pilot data: you have per-matrix mean UMI counts and corresponding cellranger mapped-read counts (e.g. 23130, 17422, 11666, 5859 from full and subsampled PBMC matrices) and need a read-depth-to-UMI curve for power estimation.

## When NOT to use

- The analysis goal is DEG/eQTL power estimation with pre-existing expression priors (e.g. the package's 10X_PBMC_1 reference study) — no new read-depth-to-UMI fit is needed
- Input data are full-length read counts rather than UMI counts, where the UMI-based depth relationship does not apply without re-estimating priors
- You only need pseudobulk reformating or gamma mixed-distribution fitting (create.pseudobulk / mixed.gamma.estimation) — those are separate upstream steps

## Inputs

- Original and read-depth subsampled UMI count matrices (e.g. complete, 75%, 50%, 25% subsamples sampled from GSE185714)
- Cellranger mapped-read counts per matrix (e.g. 23130, 17422, 11666, 5859)
- Cell annotation data frame with cell type and donor columns aligned to count-matrix column ordering

## Outputs

- read.umi.fit.new: fitted logarithmic read-depth-to-UMI regression object
- Expression prior (expression_prior.RData) containing read.umi.fit.new together with gamma.linear.fit.new and disp.fun.general.new

## How to apply

First compute the mean UMI count per cell for each pilot count matrix with scPower's meanUMI.calculation, using the original UMI count matrices at each sequencing depth. Then regress the log mean UMI counts on the log mapped-read counts with umi.read.relation, pairing each matrix's mean UMI value with its cellranger-mapped read total (e.g. 23130, 17422, 11666, 5859 reads for the complete, 75%, 50% and 25% read-depth subsamples of GSE185714). The fitted logarithmic curve (read.umi.fit) annotates how many UMIs per cell are expected at any planned read depth and is serialized alongside gamma.linear.fit.new and disp.fun.general.new in the expression prior. Validate the prior end-to-end by predicting expressed gene numbers with estimate.exp.prob.param (min.counts=3, perc.indiv=0.5) against observed expressed.genes.df counts.

## Related tools

- **scPower** (R package providing meanUMI.calculation and umi.read.relation to compute per-matrix mean UMI counts and fit the logarithmic read-depth-to-UMI relationship stored in the expression prior) — https://github.com/heiniglab/scPower
- **R** (Runtime environment in which the scPower fitting workflow is executed and the prior is serialized as expression_prior.RData)

## Examples

```
read.umi.fit.new <- scPower::umi.read.relation(meanUMI, mapped.reads = c(23130, 17422, 11666, 5859))
```

## Evaluation signals

- The fitted read.umi.fit object reproduces a monotonic logarithmic increase of mean UMI counts with mapped reads across the four sequencing depths (23130, 17422, 11666, 5859)
- End-to-end validation: predicted expressed gene numbers from estimate.exp.prob.param (min.counts=3, perc.indiv=0.5) match observed expressed.genes.df counts on the pilot data
- On the independent PBMC validation set (25,000 reads per cell; batches of 4, 4 and 8 individuals), predicted vs observed expressed gene counts reach r2 > 0.9 across all three batches
- read.umi.fit.new is successfully serialized alongside gamma.linear.fit.new and disp.fun.general.new in expression_prior.RData

## Limitations

- Data driven priors only allow reliable assessment of overall power for sample sizes smaller or roughly equal to the pilot data set
- The read-UMI relationship is technology- and depth-specific: it must be re-fit per platform (droplet-based UMI data here, sampled from GSE185714) rather than reused across technologies
- Actual experiments might result in higher doublet rates and lower numbers of usable cells than the default modelled assumptions

## Evidence

- [methods] calculated from the original count matrices using the function "meanUMI.calculation": "calculated from the original count matrices using the function "meanUMI.calculation""
- [methods] A small toy data set with a UMI count matrix is provided in the package to show the fitting of the expression priors for droplet-based methods (sampled from GSE185714): "A small toy data set with a UMI count matrix is provided in the package to show the fitting of the expression priors"
- [methods] reformating the 2d count matrix into a 3d pseudobulk matrix, using the function "create.pseudobulk": "reformating the 2d count matrix into a 3d pseudobulk matrix, using the function "create.pseudobulk""
- [readme] The second one, called [reproduce-paper-plots](vignettes/reproduce-paper-plots.pdf), shows how all plots in the manuscript were created with the package.: [evidence span withheld — anchor status: external_doc]
