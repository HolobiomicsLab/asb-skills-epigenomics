---
name: z-score-significance-assessment
description: Use when an observed overlap between query ranges (e.g., caQTL or SNP
  sets) and target ranges (e.g., ATAC-seq peaks) must be tested against block-bootstrapped
  null ranges, and a z-score of the overlap rate is needed.
when_to_use_negative:
- The comparison requires overlap counts of DEGs vs differential accessibility peaks
  with spline-based logFC threshold optimization (macrophage analysis uses macrophage.R/fit_splines.R
  instead)
- The query statistic is a correlation across single-cell multiome modalities rather
  than a range overlap rate (use multiomicsCor.R)
- A conventional shuffling/permutation null is intentionally desired (bootstrap.R
  path), though this risks overly narrow null distributions
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: bootRanges
  role: Generates the block-bootstrapped null genomic ranges whose overlap rates form
    the null distribution for the z-score
  repo: https://bioconductor.org/packages/nullranges
- name: nullranges
  role: R/Bioconductor package hosting bootRanges; used with plyranges to handle GRanges
    overlap computations
  repo: https://bioconductor.org/packages/nullranges
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad190
    title: 'bootRanges: flexible generation of null sets of genomic ranges for hypothesis
      testing'
  source_content_hash: sha256:5493ba22192309e73660bfc9856d2137fc4fb1355bf3b8fd80ceed4769153fc5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - bootRanges
  - nullranges
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# z-score-significance-assessment

## Summary

Quantifies significance of an observed genomic range overlap by standardizing the observed statistic (e.g., overlap rate, the proportion of SNPs with peaks within 10 kb) against the bootRanges block-bootstrap null distribution via a z-score, computed in the var.R step.

## When to use

Use when an observed overlap between query ranges (e.g., caQTL or SNP sets) and target ranges (e.g., ATAC-seq peaks) must be tested against block-bootstrapped null ranges, and a z-score of the overlap rate is needed.

## When NOT to use

- The comparison requires overlap counts of DEGs vs differential accessibility peaks with spline-based logFC threshold optimization (macrophage analysis uses macrophage.R/fit_splines.R instead)
- The query statistic is a correlation across single-cell multiome modalities rather than a range overlap rate (use multiomicsCor.R)
- A conventional shuffling/permutation null is intentionally desired (bootstrap.R path), though this risks overly narrow null distributions

## Inputs

- Block-bootstrapped null GRanges sets from bootRanges (output of segBootranges.R)
- Observed SNP ranges (e.g., caQTLs / total-cholesterol-associated SNPs) as GRanges
- ATAC-seq peak ranges as GRanges

## Outputs

- Per-iteration overlap rates across Lb bootstrap replicates
- Across-replicate variance of the overlap rate
- z-score of the observed overlap rate, written to overlap_rate_zscore.tsv

## How to apply

Load the block-bootstrapped null range sets produced by segBootranges.R (bootRanges) together with the observed SNP and peak GRanges using nullranges/plyranges. For each bootstrap iteration b, count SNPs whose range overlaps a peak within a 10 kb window and divide by the total number of SNPs to get the per-iteration overlap rate. Pool the Lb per-iteration rates and compute their variance to characterize the spread of the null distribution. Compute the z-score as (observed rate − mean of the Lb null rates) / standard deviation of the null rates, and write the per-iteration rates, variance, and z-score to overlap_rate_zscore.tsv. The block bootstrap is used because shuffling or permutation can yield overly narrow null distributions and over-estimated significance, whereas block bootstrapping preserves local genomic correlation structure.

## Related tools

- **bootRanges** (Generates the block-bootstrapped null genomic ranges whose overlap rates form the null distribution for the z-score) — https://bioconductor.org/packages/nullranges
- **nullranges** (R/Bioconductor package hosting bootRanges; used with plyranges to handle GRanges overlap computations) — https://bioconductor.org/packages/nullranges

## Examples

```
Rscript var.R  # after segBootranges.R: compute per-iteration overlap rates (SNPs with peaks within 10 kb), variance, and z-score -> overlap_rate_zscore.tsv
```

## Evaluation signals

- overlap_rate_zscore.tsv exists and contains per-iteration overlap rates, the across-replicate variance, and the z-score columns
- Per-iteration overlap rate equals (number of SNPs overlapping a peak within 10 kb) / (total SNPs) and lies in [0, 1]
- Z-score equals (observed rate − mean of Lb null rates) / standard deviation of null rates; recomputing from the stored per-iteration rates reproduces it
- Variance of pooled null rates is non-negative and consistent with the spread of the Lb per-iteration rates
- z-score computed from bootRanges nulls is compared against the conventional shuffling null (bootstrap.R) to confirm the permutation scheme does not over-estimate significance

## Limitations

- Shuffling or permutation-based nulls can produce overly narrow test statistic null distributions and over-estimate statistical significance, so conventional shuffling (bootstrap.R) results should be interpreted cautiously
- The overlap rate definition is tied to a fixed 10 kb window; other window choices are not covered by this card
- The null distribution spread depends on the block length; variance is computed across block lengths, so results vary with segmentation/block-length choices

## Evidence

- [abstract] bootRanges provides fast functions for generation of block bootstrapped genomic ranges representing the null hypothesis in enrichment analysis.: "bootRanges provides fast functions for generation of block bootstrapped genomic ranges representing the null hypothesis in enrichment analysis."
- [abstract] Shuffling or permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance: "We show that shuffling or permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance"
- [abstract] while creating new range sets with a block bootstrap preserves local genomic correlation structure and generates more reliable null distributions.: "while creating new range sets with a block bootstrap preserves local genomic correlation structure and generates more reliable null distributions."
- [readme] var.R (Variance of the rate of overlaps and z score): "*var.R*(Variance of the rate of overlaps and $z$ score)"
- [readme] Derive overlap rate of  caQTLs detected in human liver tissue-SNPs associated with total cholesterol: "Derive overlap rate of  caQTLs detected in human liver tissue-SNPs associated with total cholesterol:"
- [abstract] bootRanges is freely available in the R/Bioconductor package nullranges hosted at https://bioconductor.org/packages/nullranges.: "bootRanges is freely available in the R/Bioconductor package nullranges hosted at https://bioconductor.org/packages/nullranges."
