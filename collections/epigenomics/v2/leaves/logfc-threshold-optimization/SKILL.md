---
name: logfc-threshold-optimization
description: Use when differential accessibility peaks and differential expression
  gene sets need objective significance thresholds for enrichment analysis, rather
  than arbitrary logFC or P-value cutoffs.
when_to_use_negative:
- The analysis only needs a fixed a priori cutoff (e.g. p < 0.05); this skill is for
  data-driven threshold selection, not hypothesis testing at a preset alpha
- The genomic ranges come from a design where local genomic correlation structure
  must not be preserved by block bootstrapping, or a conventional shuffling null (bootstrap.R)
  is explicitly required
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: bootRanges
  role: Generates block bootstrapped genomic ranges forming the null hypothesis distribution
    used to calibrate the spline-fit divergence that defines optimized logFC thresholds
  repo: https://bioconductor.org/packages/nullranges
- name: nullranges
  role: R/Bioconductor package hosting bootRanges functions for generating null sets
    of genomic ranges
  repo: https://bioconductor.org/packages/nullranges
- name: macrophage.R
  role: Derives overlap counts of differential expression genes and differential accessibility
    peaks and fits generalized splines to optimize DEG logFC thresholds
  repo: https://github.com/Wancen/bootRangespaper/blob/master/script/macrophage.R
- name: fit_splines.R
  role: Fits the penalized-spline GLMs to observed and null overlap counts and produces
    the conditional density plot used to identify threshold divergence
  repo: https://github.com/Wancen/bootRangespaper/blob/master/script/liver/README.md
- name: segBootranges.R
  role: Performs the bootRanges block bootstrap step that builds the null distribution
  repo: https://github.com/Wancen/bootRangespaper/blob/master/script/liver/README.md
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad190
    title: 'bootRanges: flexible generation of null sets of genomic ranges for hypothesis
      testing'
  source_content_hash: sha256:9d4d4b3634fe4e506d9481624d4fda213c57153b04e7941979c54109fdb89558
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - bootRanges
  - fit_splines.R
  - macrophage.R
  - nullranges
  - segBootranges.R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# logfc-threshold-optimization

## Summary

A bootRanges-based spline method that identifies data-driven significance cutoffs for differential analysis — SNPs with -log10(P-value) > 8 and genes with |logFC| > 2 — by finding where the observed penalized-spline GLM fit of overlap counts diverges from the block-bootstrap null envelope.

## When to use

Use when differential accessibility peaks and differential expression gene sets need objective significance thresholds for enrichment analysis, rather than arbitrary logFC or P-value cutoffs.

## When NOT to use

- The analysis only needs a fixed a priori cutoff (e.g. p < 0.05); this skill is for data-driven threshold selection, not hypothesis testing at a preset alpha
- The genomic ranges come from a design where local genomic correlation structure must not be preserved by block bootstrapping, or a conventional shuffling null (bootstrap.R) is explicitly required

## Inputs

- Observed overlap count table linking differential expression genes (logFC) and differential accessibility peaks from the Alasoo 2018 macrophage dataset
- bootRanges block-bootstrap null overlap count tables over logFC
- GRanges objects of differential SNP (-log10 P) and gene (logFC) statistics

## Outputs

- fitted_thresholds.csv containing recovered cutoffs (SNPs -log10(P) > 8; genes |logFC| > 2)
- Conditional density plot of penalized-spline GLM fits over logFC with null envelope
- z scores and variances of the rate of overlaps comparing observed vs bootstrap statistics

## How to apply

Build GRanges of differential results for the Alasoo 2018 macrophage dataset, then run bootRanges block bootstrap (segBootranges.R) to generate null sets that preserve local genomic correlation structure while shuffling ranges. Count overlaps between differential expression genes and differential accessibility peaks for the observed and each bootstrap null set (macrophage.R). Fit a generalized linear model with a penalized-spline (P-spline) smooth term on logFC to the observed overlap counts and to each null set (fit_splines.R, conditional density plot), and compute the variance of the overlap rate and z score (var.R). Recover the thresholds where the observed spline separates from the bootstrap null spline distribution, yielding SNPs with -log10(P-value) > 8 and genes with |logFC| > 2 as sets where the z score reflects strong separation of the observed statistic from the bootstrap distribution. Write the recovered cutoffs to fitted_thresholds.csv.

## Related tools

- **bootRanges** (Generates block bootstrapped genomic ranges forming the null hypothesis distribution used to calibrate the spline-fit divergence that defines optimized logFC thresholds) — https://bioconductor.org/packages/nullranges
- **nullranges** (R/Bioconductor package hosting bootRanges functions for generating null sets of genomic ranges) — https://bioconductor.org/packages/nullranges
- **macrophage.R** (Derives overlap counts of differential expression genes and differential accessibility peaks and fits generalized splines to optimize DEG logFC thresholds) — https://github.com/Wancen/bootRangespaper/blob/master/script/macrophage.R
- **fit_splines.R** (Fits the penalized-spline GLMs to observed and null overlap counts and produces the conditional density plot used to identify threshold divergence) — https://github.com/Wancen/bootRangespaper/blob/master/script/liver/README.md
- **segBootranges.R** (Performs the bootRanges block bootstrap step that builds the null distribution) — https://github.com/Wancen/bootRangespaper/blob/master/script/liver/README.md

## Examples

```
Rscript https://raw.githubusercontent.com/Wancen/bootRangespaper/master/script/macrophage.R
```

## Evaluation signals

- Recovered thresholds equal SNPs -log10(P-value) > 8 and genes with |logFC| > 2
- The observed penalized-spline fit clearly diverges from the block-bootstrap null spline envelope at the reported logFC and -log10(P) values
- z scores at the recovered thresholds show strong separation of the observed statistic from the bootstrap statistic distribution
- fitted_thresholds.csv is produced and contains both the SNP and gene cutoffs
- Null sets are block bootstrapped (preserving local genomic correlation structure), not simple permutations, to avoid overly narrow null distributions

## Limitations

- Shuffling or permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance, so thresholds are only trustworthy with block-bootstrap nulls
- Thresholds were derived on the Alasoo 2018 macrophage differential expression/differential accessibility dataset; values (|logFC| > 2, -log10(P) > 8) are dataset-specific and must be re-derived for new data

## Evidence

- [abstract] It can also be used in more complex analyses, such as accessing correlations between cis-regulatory elements (CREs) and genes across cell types or providing optimized thresholds: "It can also be used in more complex analyses, such as accessing correlations between cis-regulatory elements (CREs) and genes across cell types or providing optimized thresholds"
- [abstract] bootRanges provides fast functions for generation of block bootstrapped genomic ranges representing the null hypothesis in enrichment analysis.: "bootRanges provides fast functions for generation of block bootstrapped genomic ranges representing the null hypothesis in enrichment analysis."
- [readme] Derive overlap counts of differential expression genes and differential accessibility peaks. And fitted peneralized splines to optimize DEG logFC thresholds.: "Derive overlap counts of differential expression genes and differential accessibility peaks. And fitted peneralized splines to optimize DEG logFC thresholds."
- [abstract] Shuffling or permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance: [evidence span withheld — anchor status: not_found]
- [intro] *fit_splines.R*(conditional density plot): "*fit_splines.R*(conditional density plot)"
