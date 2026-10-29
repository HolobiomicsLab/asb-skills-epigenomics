---
name: chromvar-deviation-score-statistical-testing
description: Use when you have a chromVARDeviations object with bias-corrected deviations/Z-scores
  and cell-type or sample annotations, and you want per-motif tests of differential
  deviation or differential variability between groups.
when_to_use_negative:
- When the goal is clustering or dimensionality reduction of single cells — the article
  notes newer methods such as SnapATAC outperform chromVAR for clustering tasks; chromVAR
  is better for annotating TF motif usage in cells & clusters
- When the input is raw peak counts without bias correction and motif matching — deviation
  testing requires the upstream computeDeviations step on filtered, GC-bias-corrected
  counts
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0203
tools:
- name: chromVAR
  role: Provides differentialDeviations and differentialVariability functions and
    the chromVARDeviations object with bias-corrected deviations and Z-scores tested
    for group differences
  repo: https://github.com/GreenleafLab/chromVAR
- name: R
  role: Runtime environment for the chromVAR package and the statistical testing workflow
  repo: https://github.com/GreenleafLab/chromVAR
- name: SummarizedExperiment
  role: Container for counts and colData (Cell_Type annotations) from which the deviations
    object derives
- name: Matrix
  role: Sparse matrix support for single-cell fragment count data
- name: BiocParallel
  role: Registers the parallel backend (MulticoreParam, SnowParam, or SerialParam)
    used by chromVAR functions
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/nmeth.4401
    title: 'chromVAR: inferring transcription-factor-associated accessibility from
      single-cell epigenomic data'
  source_content_hash: sha256:8f722a0359ed3c8cca11ceacafbf52ac80d771936686a618758f6bc8f277c056
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/GreenleafLab/chromVAR
  tools:
  - BiocParallel
  - Matrix
  - R
  - SummarizedExperiment
  - chromVAR
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# chromvar-deviation-score-statistical-testing

## Summary

Applies chromVAR's differentialDeviations and differentialVariability functions to bias-corrected motif deviation scores to test which motifs or genomic annotations differ between groups of cells or samples in scATAC-seq data.

## When to use

Use when you have a chromVARDeviations object with bias-corrected deviations/Z-scores and cell-type or sample annotations, and you want per-motif tests of differential deviation or differential variability between groups.

## When NOT to use

- When the goal is clustering or dimensionality reduction of single cells — the article notes newer methods such as SnapATAC outperform chromVAR for clustering tasks; chromVAR is better for annotating TF motif usage in cells & clusters
- When the input is raw peak counts without bias correction and motif matching — deviation testing requires the upstream computeDeviations step on filtered, GC-bias-corrected counts

## Inputs

- chromVARDeviations object with bias-corrected deviations and deviation Z-scores for motifs (e.g., JASPAR motifs)
- colData cell-type or sample annotations (e.g., Cell_Type column) attached to the deviations object
- RangedSummarizedExperiment of filtered fragment counts used to compute deviations

## Outputs

- differentialDeviations result table with per-motif statistics and p-values comparing groups
- differentialVariability result table with per-motif variability statistics and p-values
- Combined CSV (e.g., differential_deviations.csv) with rows per tested motif

## How to apply

Load a pre-computed chromVARDeviations object containing bias-corrected deviations and deviation Z-scores for JASPAR motifs across single-cell samples, with colData annotations (e.g., a 'Cell_Type' column). Run differentialDeviations with the group column name to test, per motif, whether bias-corrected deviations differ significantly between groups, and differentialVariability with the same column to test whether motif variability differs between groups. Combine both per-motif statistic and p-value tables and export them (e.g., to differential_deviations.csv). These tests build on chromVAR's upstream workflow (filterSamples/filterPeaks, addGCBias, matchMotifs, computeDeviations), which estimates gain or loss of accessibility within peaks sharing the same motif or annotation while controlling for technical biases. Register a BiocParallel backend before running chromVAR functions for parallelization.

## Related tools

- **chromVAR** (Provides differentialDeviations and differentialVariability functions and the chromVARDeviations object with bias-corrected deviations and Z-scores tested for group differences) — https://github.com/GreenleafLab/chromVAR
- **R** (Runtime environment for the chromVAR package and the statistical testing workflow) — https://github.com/GreenleafLab/chromVAR
- **SummarizedExperiment** (Container for counts and colData (Cell_Type annotations) from which the deviations object derives)
- **Matrix** (Sparse matrix support for single-cell fragment count data)
- **BiocParallel** (Registers the parallel backend (MulticoreParam, SnowParam, or SerialParam) used by chromVAR functions)

## Examples

```
dev <- computeDeviations(object = counts_filtered, annotations = motif_ix); diff_dev <- differentialDeviations(dev, 'Cell_Type'); diff_var <- differentialVariability(dev, 'Cell_Type')
```

## Evaluation signals

- Output CSV exists and contains one row per tested motif
- Result tables include the per-motif differential test statistics and p-value columns produced by differentialDeviations and differentialVariability
- Deviations were computed after filterSamples/filterPeaks and addGCBias, so deviations are bias-corrected for GC content and technical artifacts
- Motif annotations trace to a defined motif set (e.g., JASPAR motifs fetched via getJasparMotifs and matched with matchMotifs)
- A BiocParallel backend was explicitly registered before running chromVAR functions

## Limitations

- differentialDeviations/differentialVariability tests require prior computation of bias-corrected deviations; they do not operate on raw counts
- chromVAR is sparse-data-oriented; sample and peak filtering (min_depth, min_in_peaks, per-peak fragment thresholds) is advisable before computing deviations
- For clustering of single cells, SnapATAC outperforms chromVAR in the evaluated tasks; chromVAR is complementary as a way of annotating TF motif usage

## Evidence

- [intro] The package aims to identify motifs or other genomic annotations associated with variability in chromatin accessibility between individual cells or samples.: "The package aims to identify motifs or other genomic annotations associated with variability in chromatin accessibility between individual cells or samples."
- [abstract] chromVAR estimates gain or loss of accessibility within peaks sharing the same motif or annotation while controlling for technical biases.: "estimating gain or loss of accessibility within peaks sharing the same motif or annotation while controlling for technical biases"
- [readme] Before running chromVAR functions, it is advisable to use the `register` function from BiocParallel to specify your preferred method of parallelization.: [evidence span withheld — anchor status: external_doc]
- [methods] Two parameters are used for filtering -- min_in_peaks and min_depth.: "Two parameters are used for filtering -- min_in_peaks and min_depth."
- [intro] chromVAR is an R package for the analysis of sparse chromatin accessibility data from single cell or bulk ATAC or DNAse-seq data.: "chromVAR is an R package for the analysis of sparse chromatin accessibility data from single cell or bulk ATAC or DNAse-seq data."
