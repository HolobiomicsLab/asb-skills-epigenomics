---
name: cpg-distance-binned-correlation-analysis
description: Use when you need to characterize how correlation between adjacent DNA
  methylation p-values decays with genomic distance, e.g. to justify correlation-aware
  p-value combination in SMITE or reproduce its Figure 2 correlation analysis.
when_to_use_negative:
- The p-values are independent across sites, so a correlation correction is unnecessary
- The data are not methylation p-values assayed at genomic coordinates (no distance
  binning possible)
- You only need combined gene-level p-values without characterizing the underlying
  CpG correlation structure
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
tools:
- name: SMITE
  role: R/Bioconductor package whose makePvalueAnnotation and annotateModification
    (mod_corr=TRUE, weight_by_method='Stouffer') estimate the CpG p-value correlation
    structure via Monte Carlo resampling
  repo: https://github.com/GreallyLab/SMITE
- name: R
  role: Language used to run the SMITE workflow and compute/plot binned correlations
    with base R graphics
- name: SMITEdatasets
  role: Curated datasets repository providing processed data used in the SMITE methods
    vignette
  repo: https://github.com/GreallyLab/SMITEdatasets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-017-1477-3
    title: 'SMITE: an R/Bioconductor package that identifies network modules by integrating
      genomic and epigenomic information'
  source_content_hash: sha256:27812d8e102059a887eba4bdea3c40bf0c2793e6c2ba95bd828f4cc1494436d7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GreallyLab/SMITE
  tools:
  - R
  - SMITE
  - SMITEdatasets
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# cpg-distance-binned-correlation-analysis

## Summary

Estimates the distance-dependent correlation structure of CpG methylation p-values using SMITE's annotateModification with mod_corr=TRUE and Monte Carlo resampling of combined scores, then tabulates and plots average Pearson correlation across 500 genomic distance bins for the T. gondii HFF dataset.

## When to use

Use when you need to characterize how correlation between adjacent DNA methylation p-values decays with genomic distance, e.g. to justify correlation-aware p-value combination in SMITE or reproduce its Figure 2 correlation analysis.

## When NOT to use

- The p-values are independent across sites, so a correlation correction is unnecessary
- The data are not methylation p-values assayed at genomic coordinates (no distance binning possible)
- You only need combined gene-level p-values without characterizing the underlying CpG correlation structure

## Inputs

- methylation dataset from the SMITE package (data(methylation)) with CpG coordinates and p-values
- hg19_genes BED gene annotation (data(hg19_genes_bed))

## Outputs

- methylation_correlation_by_distance.csv — per-bin average Pearson correlations for 500 genomic distance bins
- methylation_correlation_by_distance.png — plot of average Pearson correlation vs. genomic distance (base R graphics)

## How to apply

Load the methylation dataset from the SMITE package (data(methylation)) and curate it by removing CpGs with NA p-values and replacing p-values of zero with the minimum non-zero p-value. Map the curated CpGs to gene contexts by building a PvalueAnnotation with makePvalueAnnotation using the hg19_genes BED annotation. Run annotateModification with weight_by_method='Stouffer' and mod_corr=TRUE so SMITE estimates the correlation structure, using the Monte Carlo Method (MCM) — 500 draws of combined scores sampled with replacement — to obtain FDR-like p-values. From the resampled combined-score replicates, compute the average Pearson correlation between adjacent CpG p-values in each of 500 genomic distance bins, tabulate the per-bin averages, and plot average Pearson correlation against genomic distance. The rationale is that SMITE combines p-values accounting for correlation between non-independent values, and the empirical binned correlation quantifies that dependence structure.

## Related tools

- **SMITE** (R/Bioconductor package whose makePvalueAnnotation and annotateModification (mod_corr=TRUE, weight_by_method='Stouffer') estimate the CpG p-value correlation structure via Monte Carlo resampling) — https://github.com/GreallyLab/SMITE
- **R** (Language used to run the SMITE workflow and compute/plot binned correlations with base R graphics)
- **SMITEdatasets** (Curated datasets repository providing processed data used in the SMITE methods vignette) — https://github.com/GreallyLab/SMITEdatasets

## Examples

```
data(methylation); data(hg19_genes_bed); annotation <- makePvalueAnnotation(data=hg19_genes_bed); annotation <- annotateModification(annotation, methylation, weight_by_method="Stouffer", mod_corr=TRUE)
```

## Evaluation signals

- The binned average Pearson correlation is higher between p-values close to one another and decreases monotonically (trend) as distance between assayed sites increases, matching SMITE's companion paper Figure 2
- The output table contains one average correlation per each of the 500 genomic distance bins
- Zero p-values are absent from the curated input (all replaced with the minimum non-zero p-value) and no NA p-values remain
- annotateModification was called with mod_corr=TRUE and weight_by_method='Stouffer', with 500 Monte Carlo resampling draws sampled with replacement
- The plotted correlation was produced with base R graphics and saved as methylation_correlation_by_distance.png

## Limitations

- The estimated correlation is empirical and depends on the resampling scheme (500 draws, with replacement); conclusions are about average Pearson correlation per distance bin, not pairwise CpG correlations
- The workflow assumes methylation p-values mapped into gene contexts via the hg19 gene annotation; sites outside those contexts are not represented in the binned estimates
- The binned correlation is a trend/descriptive estimate (no per-bin significance thresholds are specified in the article)

## Evidence

- [abstract] SMITE combines p-values by accounting for the correlation between non-independent values within data sets, allowing genes and gene modules in an interaction network to be assigned significance values: "SMITE combines p-values by accounting for the correlation between non-independent values within data sets, allowing genes and gene modules in an interaction network to be assigned significance values"
- [methods] ## Remove NAs from p-value column ##: "## Remove NAs from p-value column ##
methylation <- methylation[-which(is.na(methylation[, 5])), ]"
- [methods] ## Replace zeros with minimum non-zero p-value ##: "## Replace zeros with minimum non-zero p-value ##
methylation[,5] <- replace(methylation[,5], methylation[,5]==0,"
- [methods] annotateModification call in methods: "test_annotation <- annotateModification(test_annotation, methylation, weight_by_method="Stouffer","
- [methods] The package can be found on [Github](https://github.com/GreallyLab/SMITE): "The package can be found on [Github](https://github.com/GreallyLab/SMITE)"
