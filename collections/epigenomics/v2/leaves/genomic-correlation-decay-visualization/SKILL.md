---
name: genomic-correlation-decay-visualization
description: Use when integrating epigenomic p-values (e.g., DNA methylation) whose
  assay sites are spatially non-independent, and you need to quantify or visualize
  how correlation between adjacent CpG p-values decays with genomic distance before
  combining p-values.
when_to_use_negative:
- The input data are independent p-values with no spatial/genomic adjacency structure
  — the correlation decay estimate has no meaning
- The goal is module identification or network scoring rather than characterizing
  the correlation structure of the input p-values
- Methylation data have already been processed into a correlation-corrected combined
  score by SMITE (annotateModification with mod_corr=TRUE already run)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
tools:
- name: SMITE
  role: R/Bioconductor package whose makePvalueAnnotation and annotateModification
    (mod_corr=TRUE, Stouffer weighting, MCM resampling) estimate the correlation structure
    among methylation p-values
  repo: https://github.com/GreallyLab/SMITE
- name: R
  role: Environment for data curation, binning the estimated correlations by genomic
    distance, and base-R plotting of the correlation-vs-distance curve
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-017-1477-3
    title: 'SMITE: an R/Bioconductor package that identifies network modules by integrating
      genomic and epigenomic information'
  source_content_hash: sha256:a758643606ac5e4b6e006f5440f57405ac6c698828887b5f1a03ca3e3ec54e15
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GreallyLab/SMITE
  tools:
  - R
  - SMITE
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# genomic-correlation-decay-visualization

## Summary

Reproduces SMITE's Monte Carlo estimation of the average Pearson correlation between adjacent DNA methylation p-values, tabulated in genomic distance bins, and visualizes how that correlation decays with increasing CpG-to-CpG distance in the T. gondii HFF dataset.

## When to use

Use when integrating epigenomic p-values (e.g., DNA methylation) whose assay sites are spatially non-independent, and you need to quantify or visualize how correlation between adjacent CpG p-values decays with genomic distance before combining p-values.

## When NOT to use

- The input data are independent p-values with no spatial/genomic adjacency structure — the correlation decay estimate has no meaning
- The goal is module identification or network scoring rather than characterizing the correlation structure of the input p-values
- Methylation data have already been processed into a correlation-corrected combined score by SMITE (annotateModification with mod_corr=TRUE already run)

## Inputs

- DNA methylation p-value dataset from the SMITE package (data(methylation)) with genomic coordinates and p-values (NA and zero p-values filtered out)
- hg19 gene annotation BED file (data(hg19_genes_bed)) for mapping CpGs to promoter/gene-body contexts via makePvalueAnnotation

## Outputs

- methylation_correlation_by_distance.csv — per-bin average Pearson correlation between adjacent CpG p-values across 500 genomic distance bins
- methylation_correlation_by_distance.png — base R plot of average Pearson correlation vs. genomic distance

## How to apply

Load the methylation dataset from the SMITE package with data(methylation) and curate it by removing CpGs with NA p-values and replacing p-values of zero with the minimum non-zero p-value. Build a PvalueAnnotation with makePvalueAnnotation using the hg19_genes BED annotation, then run annotateModification on the methylation data with mod_corr=TRUE and weight_by_method="Stouffer" so SMITE estimates the correlation structure. Use the Monte Carlo Method (MCM) resampling of combined scores (500 draws, sampling with replacement) to obtain FDR-like p-values; from the resampled combined-score replicates, compute the average Pearson correlation between adjacent CpG p-values in each of 500 genomic distance bins. Tabulate the per-bin average correlations to methylation_correlation_by_distance.csv and plot average Pearson correlation against genomic distance with base R graphics, saving as methylation_correlation_by_distance.png.

## Related tools

- **SMITE** (R/Bioconductor package whose makePvalueAnnotation and annotateModification (mod_corr=TRUE, Stouffer weighting, MCM resampling) estimate the correlation structure among methylation p-values) — https://github.com/GreallyLab/SMITE
- **R** (Environment for data curation, binning the estimated correlations by genomic distance, and base-R plotting of the correlation-vs-distance curve)

## Examples

```
data(methylation); methylation <- methylation[-which(is.na(methylation[,5])), ]; annot <- makePvalueAnnotation(data=hg19_genes, other_data=1); annot <- annotateModification(annot, methylation, weight_by_method="Stouffer", mod_corr=TRUE)
```

## Evaluation signals

- The plotted binned correlation shows a trend of decreasing correlation with increasing CpG distance, consistent with the correlation structure SMITE reports in the companion paper's Figure 2
- The CSV contains one average Pearson correlation value for each of the 500 genomic distance bins
- After curation, no rows retain NA or zero p-values (zeros were replaced with the minimum non-zero p-value)
- The correlation estimate derives from the MCM resampled combined scores (500 draws with replacement), not from raw p-values alone

## Limitations

- The correlation estimate depends on the chosen resampling parameters (500 Monte Carlo draws, sampling with replacement); different settings change the binned estimates
- The skill is grounded in the T. gondii HFF methylation dataset; correlation-by-distance behavior may differ for other assays or organisms
- Replacing zero p-values with the minimum non-zero p-value is an imputation choice that can affect downstream correlation estimates

## Evidence

- [abstract] SMITE combines p-values by accounting for the correlation between non-independent values within data sets: "SMITE combines p-values by accounting for the correlation between non-independent values within data sets"
- [methods] Remove NAs from p-value column: "## Remove NAs from p-value column ##
methylation <- methylation[-which(is.na(methylation[, 5])), ]"
- [methods] Replace zeros with minimum non-zero p-value: "## Replace zeros with minimum non-zero p-value ##
methylation[,5] <- replace(methylation[,5], methylation[,5]==0,"
- [methods] annotateModification call with Stouffer weighting: "test_annotation <- annotateModification(test_annotation, methylation, weight_by_method="Stouffer","
- [methods] Load methylation data: "## Load methylation data ##
data(methylation)"
