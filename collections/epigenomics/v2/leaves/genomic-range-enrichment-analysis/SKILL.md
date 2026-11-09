---
name: genomic-range-enrichment-analysis
description: Use when testing enrichment of overlaps between genomic range sets (e.g.
  caQTL SNPs vs chromatin accessibility peaks, or DEGs vs DA peaks) and when permutation-based
  nulls seem too narrow or thresholds need optimization.
when_to_use_negative:
- When the analysis goal is a simple single overlap P-value rather than a null-distribution
  comparison or threshold optimization — a plain permutation test may suffice, though
  it risks over-estimating significance.
- When range sets lack genomic coordinates or cannot be represented as GRanges objects.
- When testing mean correlations across cell types from Multiome ATAC + Gene Expression
  data (multiomicsCor.R), which is a distinct analysis in the same paper.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0749
tools:
- name: bootRanges
  role: Generates block-bootstrapped genomic ranges representing the null hypothesis
    for enrichment analysis and supports spline-based threshold optimization of logFC
  repo: https://bioconductor.org/packages/nullranges
- name: nullranges
  role: R/Bioconductor package hosting bootRanges functions for null range generation
  repo: https://bioconductor.org/packages/nullranges
- name: GSC
  role: Conventional genomic shuffle approach compared against bootRanges for running
    time on genome-scale statistics
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad190
    title: 'bootRanges: flexible generation of null sets of genomic ranges for hypothesis
      testing'
  source_content_hash: sha256:e1f680d08acf2fa6dd2f440aaa23dd54d4cfa86f447dd71e117f0e6ad5bc90e7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GSC
  - bootRanges
  - nullranges
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# genomic-range-enrichment-analysis

## Summary

Applies bootRanges block bootstrapping of genomic ranges to compute null distributions of overlap statistics for enrichment testing, and fits penalized splines to recover optimized significance thresholds (e.g. SNP -log10(P) > 8, gene |logFC| > 2).

## When to use

Use when testing enrichment of overlaps between genomic range sets (e.g. caQTL SNPs vs chromatin accessibility peaks, or DEGs vs DA peaks) and when permutation-based nulls seem too narrow or thresholds need optimization.

## When NOT to use

- When the analysis goal is a simple single overlap P-value rather than a null-distribution comparison or threshold optimization — a plain permutation test may suffice, though it risks over-estimating significance.
- When range sets lack genomic coordinates or cannot be represented as GRanges objects.
- When testing mean correlations across cell types from Multiome ATAC + Gene Expression data (multiomicsCor.R), which is a distinct analysis in the same paper.

## Inputs

- GRanges objects for observed query and subject range sets (e.g. caQTL SNPs, chromatin accessibility peaks, differential expression genes with logFC)
- Overlap count tables linking differential expression genes (logFC) and differential accessibility peaks (Alasoo 2018 macrophage dataset)
- Observed and bootstrapped overlap count/statistic tables produced by bootRanges

## Outputs

- Null distributions of overlap rate and z scores from block-bootstrapped range sets
- Penalized-spline (conditional density) fits of observed vs bootstrap overlap statistics over logFC
- Optimized significance thresholds (e.g. SNPs -log10(P) > 8, genes |logFC| > 2) written to fitted_thresholds.csv

## How to apply

Construct GRanges objects for the observed range sets (GRanges_construct.R), then run the segmented block bootstrap (segBootranges.R) to generate null range sets that preserve local genomic correlation structure. Compute the rate of overlaps and its variance along with z scores (var.R), then fit generalized penalized-spline (P-spline) GLMs to observed overlap counts — e.g. smoothing over logFC for the Alasoo 2018 macrophage DEG/DA peak data — and repeat the fit on each bootstrap null set (fit_splines.R, macrophage.R). Compare the observed spline against the null spline envelope to identify logFC (or SNP -log10(P)) values where the observed statistic separates strongly from the bootstrap distribution, recovering optimized thresholds. For the liver caQTL analysis, this workflow identified SNPs with -log10(P-value) > 8 as strongly separated, while the macrophage analysis recovered genes with |logFC| > 2; write recovered thresholds to fitted_thresholds.csv.

## Related tools

- **bootRanges** (Generates block-bootstrapped genomic ranges representing the null hypothesis for enrichment analysis and supports spline-based threshold optimization of logFC) — https://bioconductor.org/packages/nullranges
- **nullranges** (R/Bioconductor package hosting bootRanges functions for null range generation) — https://bioconductor.org/packages/nullranges
- **GSC** (Conventional genomic shuffle approach compared against bootRanges for running time on genome-scale statistics)

## Examples

```
Rscript macrophage.R  # derive DEG/DA peak overlap counts and fit generalized splines to optimize DEG logFC thresholds
```

## Evaluation signals

- Recovered thresholds reproduce the known values: SNPs with -log10(P-value) > 8 and genes with |logFC| > 2.
- Observed spline fit diverges from the bootstrap null spline envelope at the reported threshold, with a large z score separating observed from null overlap statistics.
- Null distributions from block bootstrap are wider (less over-estimated significance) than those from conventional shuffling/permutation.
- bootRanges running time on genome-scale statistics is competitive with (faster than) GSC.
- fitted_thresholds.csv is produced containing the optimized logFC and SNP -log10(P) cutoffs.

## Limitations

- Shuffling or permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance — a key reason to prefer block bootstrap.
- Block bootstrap quality depends on preserving local genomic correlation structure; segment length choices affect null reliability.
- bootRanges vs GSC running-time differences matter on genome-scale statistics and should be considered for large analyses.

## Evidence

- [abstract] bootRanges provides fast functions for generation of block bootstrapped genomic ranges representing the null hypothesis in enrichment analysis.: "bootRanges provides fast functions for generation of block bootstrapped genomic ranges representing the null hypothesis in enrichment analysis."
- [abstract] Shuffling or permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance: "We show that shuffling or permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance"
- [abstract] Block bootstrap creates new range sets that preserve local genomic correlation structure and generate more reliable null distributions: "while creating new range sets with a block bootstrap preserves local genomic correlation structure and generates more reliable null distributions."
- [abstract] It can also be used in more complex analyses, such as accessing correlations between cis-regulatory elements (CREs) and genes across cell types or providing optimized thresholds, e.g. log fold change: "It can also be used in more complex analyses, such as accessing correlations between cis-regulatory elements (CREs) and genes across cell types or providing optimized thresholds, e.g. log fold change"
- [readme] Derive overlap counts of differential expression genes and differential accessibility peaks. And fitted peneralized splines to optimize DEG logFC thresholds.: "Derive overlap counts of differential expression genes and differential accessibility peaks.
And fitted peneralized splines to optimize DEG logFC thresholds. "
- [readme] *var.R*(Variance of the rate of overlaps and $z$ score) -> *fit_splines.R*(conditional density plot): "*var.R*(Variance of the rate of overlaps and $z$ score) -> *fit_splines.R*(conditional density plot)"
