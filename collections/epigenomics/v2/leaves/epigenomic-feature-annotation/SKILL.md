---
name: epigenomic-feature-annotation
description: Use when you have BED-like interval data (methylation, histone marks)
  plus a gene annotation and need to map each interval to promoter/body/other-TSS
  categories before combining p-values into gene scores.
when_to_use_negative:
- Your data are already gene-level (e.g. a gene x sample expression matrix with gene
  symbols/RefSeq IDs) and no interval-to-gene mapping is needed — go straight to annotateExpression/scorePval.
- Intervals cannot be assigned to genes (no genome annotation or coordinates) — the
  promoter/body/other_tss_distance categorization requires genomic positions.
- You need de novo peak calling or differential analysis of the raw sequencing data
  — this skill only annotates already-processed interval tables.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3169
tools:
- name: SMITE
  role: provides makePvalueAnnotation, annotateExpression, annotateModification, and
    extractModSummary to map intervals to genes and attach p-values
  repo: https://github.com/GreallyLab/SMITE
- name: R
  role: environment in which the SMITE annotation workflow is run
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-017-1477-3
    title: 'SMITE: an R/Bioconductor package that identifies network modules by integrating
      genomic and epigenomic information'
  source_content_hash: sha256:61f347d788c87b884515e610d336376a362046bae968735d2bca088187a0c670
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GreallyLab/SMITE
  tools:
  - R
  - SMITE
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# epigenomic-feature-annotation

## Summary

Annotate genomic intervals (promoters, gene bodies, other regulatory features such as H3K4me1 peaks) onto genes with SMITE's makePvalueAnnotation, then attach expression and modification p-values (annotateExpression, annotateModification) so interval-level evidence can be aggregated to gene-level scores.

## When to use

Use when you have BED-like interval data (methylation, histone marks) plus a gene annotation and need to map each interval to promoter/body/other-TSS categories before combining p-values into gene scores.

## When NOT to use

- Your data are already gene-level (e.g. a gene x sample expression matrix with gene symbols/RefSeq IDs) and no interval-to-gene mapping is needed — go straight to annotateExpression/scorePval.
- Intervals cannot be assigned to genes (no genome annotation or coordinates) — the promoter/body/other_tss_distance categorization requires genomic positions.
- You need de novo peak calling or differential analysis of the raw sequencing data — this skill only annotates already-processed interval tables.

## Inputs

- methylation interval data (BED-like table with genomic coordinates, effect and p-value columns, e.g. SMITE methylationdata)
- hg19 gene annotation BED file (hg19_genes_bed)
- histone modification peaks (histone_h3k4me1)
- curated expression data with effect and p-value columns (curated_expressiondata)

## Outputs

- SMITE PvalueAnnotation object with expression and modification p-values attached per gene
- extractModSummary data frame of combined per-gene p-values for each modification category (e.g. methylation_promoter, methylation_body, h3k4me1)
- optionally a CSV of paired weighted/unweighted gene scores for downstream comparison

## How to apply

Load the SMITE example data (methylationdata, hg19_genes_bed, histone_h3k4me1) and first clean the modification table: replace p-values of zero with the minimum non-zero p-value and drop rows with NA p-values. Build a PvalueAnnotation with makePvalueAnnotation, specifying promoter_upstream_distance=1000, promoter_downstream_distance=1000, and other_tss_distance=5000 for the h3k4me1 track supplied via other_data. Attach expression p-values with annotateExpression (effect_col=1, pval_col=2), then attach each modification with annotateModification, choosing a weight_by_method such as 'Stouffer' and optional weight_by distances (e.g., promoter='distance', body='distance', h3k4me1='distance') or running unweighted. The rationale, per the paper, is that multiple events observed at the same locus increase confidence that regulatory activity is genuine, so intervals must be assigned to genes before SMITE combines p-values accounting for correlation between non-independent values. Extract the per-gene annotation and combined p-value summaries with extractModSummary to confirm every interval landed in the intended category.

## Related tools

- **SMITE** (provides makePvalueAnnotation, annotateExpression, annotateModification, and extractModSummary to map intervals to genes and attach p-values) — https://github.com/GreallyLab/SMITE
- **R** (environment in which the SMITE annotation workflow is run)

## Examples

```
test_annotation <- makePvalueAnnotation(data=hg19_genes, other_data=list(h3k4me1=h3k4me1), promoter_upstream_distance=1000, promoter_downstream_distance=1000, other_tss_distance=5000)
```

## Evaluation signals

- Every interval row survives with a valid p-value after zero-replacement (min non-zero) and NA removal; row counts match expectations.
- annotateModification returns a PvalueAnnotation with modification slots for promoter, body, and each other_data track (e.g. h3k4me1) populated.
- extractModSummary yields per-gene combined p-values whose rows correspond to genes in hg19_genes_bed, not to individual intervals.
- For the weighted Stouffer variant, a regression of distance-weighted scores on unweighted scores gives R-squared ≈ 0.99 (Additional file 1: Figure S2), with nearly identical final modules and annotations.
- Gene IDs in the annotation match the expected type (RefSeq/symbol) after convertGeneIds conversion.

## Limitations

- P-values of exactly zero must be replaced with the minimum non-zero p-value and NA rows dropped, otherwise aggregation fails or is biased.
- SMITE is robust to weighting choice (R2 ≈ 0.99), so weighting by distance changes results minimally; do not expect weighting to materially alter module discovery.
- Promoter/other-TSS distance windows are user-set (here 1000 bp up/down for promoters, 5000 bp for other_tss_distance); different thresholds change which intervals are assigned to which genes.

## Evidence

- [abstract] SMITE combines p-values by accounting for the correlation between non-independent values within data sets, allowing genes and gene modules in an interaction network to be assigned significance values: "SMITE combines p-values by accounting for the correlation between non-independent values within data"
- [methods] makePvalueAnnotation call: "test_annotation <- makePvalueAnnotation(data=hg19_genes, other_data=list(h3k4me1=h3k4me1),"
- [methods] annotateModification call with Stouffer weighting: "test_annotation <- annotateModification(test_annotation, methylation, weight_by_method="Stouffer","
- [methods] NA removal filter step: "## Remove NAs from p-value column ##
methylation <- methylation[-which(is.na(methylation[, 5])), ]"
- [intro] multiple events at a locus rationale: "they also suggest that multiple types of events observed at the same locus increase confidence that regulatory activity is genuinely occurring at that locus"
