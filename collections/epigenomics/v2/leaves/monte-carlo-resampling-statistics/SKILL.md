---
name: monte-carlo-resampling-statistics
description: Use when combining p-values from correlated genomic assays (e.g., methylation
  p-values from multiple CpGs mapped to one gene) and you need SMITE's MCM correlation-corrected
  resampling, including characterizing how CpG correlation decays with genomic distance.
when_to_use_negative:
- P-value columns are independent across features (no repeated CpGs per gene or correlated
  assays), so the correlation correction and Monte Carlo resampling add no value
- Input p-values are already combined or FDR-adjusted per gene, so re-running MCM
  resampling would double-correct
- The goal is module detection or network scoring in SMITE (spinglass/igraph step)
  rather than estimating CpG correlation structure
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3365
- http://edamontology.org/topic_0203
tools:
- name: SMITE
  role: Provides makePvalueAnnotation and annotateModification with weight_by_method="Stouffer"
    and mod_corr=TRUE, plus the Monte Carlo Method resampling of combined scores to
    estimate correlation-corrected significance
  repo: https://github.com/GreallyLab/SMITE
- name: R
  role: Language in which the methylation curation (NA removal, zero replacement),
    distance binning, Pearson correlation computation, and plotting are performed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-017-1477-3
    title: 'SMITE: an R/Bioconductor package that identifies network modules by integrating
      genomic and epigenomic information'
  source_content_hash: sha256:875cd6510f32901c39dedaf34ade5249fbc59a4e7d5048fd91ae14937b7f1155
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GreallyLab/SMITE
  tools:
  - R
  - SMITE
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3365
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# monte-carlo-resampling-statistics

## Summary

Reproduces SMITE's Monte Carlo Method (MCM) for combining p-values under an estimated correlation structure, and specifically the distance-binned Monte Carlo correlation matrix for adjacent methylation p-values in the T. gondii HFF dataset, so that gene-level significance correctly reflects non-independent CpG observations.

## When to use

Use when combining p-values from correlated genomic assays (e.g., methylation p-values from multiple CpGs mapped to one gene) and you need SMITE's MCM correlation-corrected resampling, including characterizing how CpG correlation decays with genomic distance.

## When NOT to use

- P-value columns are independent across features (no repeated CpGs per gene or correlated assays), so the correlation correction and Monte Carlo resampling add no value
- Input p-values are already combined or FDR-adjusted per gene, so re-running MCM resampling would double-correct
- The goal is module detection or network scoring in SMITE (spinglass/igraph step) rather than estimating CpG correlation structure

## Inputs

- methylation dataset from SMITE (data(methylation)) with CpG coordinates and p-values (column 5)
- hg19 gene annotation BED file (data(hg19_genes_bed)) for makePvalueAnnotation

## Outputs

- PvalueAnnotation object with correlation-corrected methylation p-values (annotateModification, Stouffer weighting, mod_corr=TRUE)
- Per-bin average Pearson correlation table for 500 genomic distance bins (methylation_correlation_by_distance.csv)
- Plot of average Pearson correlation vs. genomic distance (methylation_correlation_by_distance.png)

## How to apply

Load the methylation data from the SMITE package (data(methylation)) and curate it: drop CpGs with NA p-values (methylation[-which(is.na(methylation[, 5])), ]) and replace zero p-values with the minimum non-zero p-value so resampling statistics remain defined. Map curated CpGs to gene contexts with makePvalueAnnotation using the hg19_genes BED annotation, then call annotateModification with weight_by_method="Stouffer" and mod_corr=TRUE so SMITE estimates the correlation structure rather than treating CpGs as independent. SMITE's Monte Carlo Method resamples combined scores (e.g., 500 draws sampled with replacement) to derive FDR-like significance; from these replicates, compute the average Pearson correlation between adjacent CpG p-values in each of 500 genomic distance bins. Rationale: the paper stresses that SMITE combines p-values by accounting for correlation between non-independent values, and the T. gondii HFF analysis shows estimated correlation is higher for nearby sites and decreases with distance (paper's Figure 2).

## Related tools

- **SMITE** (Provides makePvalueAnnotation and annotateModification with weight_by_method="Stouffer" and mod_corr=TRUE, plus the Monte Carlo Method resampling of combined scores to estimate correlation-corrected significance) — https://github.com/GreallyLab/SMITE
- **R** (Language in which the methylation curation (NA removal, zero replacement), distance binning, Pearson correlation computation, and plotting are performed)

## Examples

```
library(SMITE); data(methylation); methylation <- methylation[-which(is.na(methylation[,5])),]; test_annotation <- annotateModification(test_annotation, methylation, weight_by_method="Stouffer")
```

## Evaluation signals

- The curated methylation table contains no NA values and no zeros in the p-value column (column 5)
- The distance-binned plot of average Pearson correlation shows a monotone-decreasing trend: correlation is highest between nearby CpGs and declines with genomic distance
- The binned correlation trend matches the correlation structure reported in the companion paper's Figure 2 for the T. gondii HFF methylation data
- Exactly 500 Monte Carlo resampling draws and 500 genomic distance bins are used and tabulated in methylation_correlation_by_distance.csv
- annotateModification is invoked with weight_by_method="Stouffer" and mod_corr=TRUE, confirming correlation correction is active rather than assuming independent p-values

## Limitations

- P-values of exactly zero break resampling statistics and must be replaced with the dataset's minimum non-zero p-value before analysis
- CpGs with NA p-values must be removed, reducing the set of assayed sites that contribute to each gene's combined score
- The correlation decay pattern is empirical for the T. gondii HFF methylation dataset and is estimated in distance bins, so within-bin variation is averaged out

## Evidence

- [abstract] Abstract statement that correlation correction is core to SMITE's p-value combination: "SMITE combines p-values by accounting for the correlation between non-independent values within data sets"
- [intro] Method-level rationale that repeated events at one locus carry corroborating evidence (why CpG correlation is modeled): "they also suggest that multiple types of events observed at the same locus increase confidence that regulatory activity is genuinely occurring at that locus"
- [methods] Filter step: remove CpGs with NA p-values before correlation estimation: "## Remove NAs from p-value column ##
methylation <- methylation[-which(is.na(methylation[, 5])), ]"
- [methods] Filter step: replace zero p-values with the minimum non-zero p-value: "## Replace zeros with minimum non-zero p-value ##
methylation[,5] <- replace(methylation[,5], methylation[,5]==0,"
- [methods] Core call enabling correlation-corrected Stouffer combination of methylation p-values: "test_annotation <- annotateModification(test_annotation, methylation, weight_by_method="Stouffer","
