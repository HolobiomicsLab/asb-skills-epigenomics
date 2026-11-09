---
name: spline-glm-fitting-overlap-counts
description: Use when bootRanges overlap counts between differential expression genes
  (logFC) and differential accessibility peaks must be converted into an optimized,
  data-driven logFC or SNP -log10(P) significance threshold rather than an arbitrary
  cutoff.
when_to_use_negative:
- When a fixed a priori cutoff (e.g. standard FDR threshold) is acceptable and no
  data-driven optimized threshold is needed
- When overlap counts come from conventional shuffling/permutation nulls rather than
  bootRanges block-bootstrap nulls, since the narrower null distributions undermine
  the divergence-based threshold recovery
- When the input lacks matched differential statistics (logFC for genes, -log10(P)
  for SNPs) to use as the spline smooth-term variable
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_2269
tools:
- name: bootRanges
  role: Generates block-bootstrapped genomic ranges (null sets) whose overlap counts
    provide the null spline distribution against which the observed spline is compared
    to recover thresholds
  repo: https://bioconductor.org/packages/nullranges
- name: nullranges
  role: R/Bioconductor package hosting bootRanges functions used to build the null
    sets feeding the spline GLM fitting
  repo: https://bioconductor.org/packages/nullranges
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad190
    title: 'bootRanges: flexible generation of null sets of genomic ranges for hypothesis
      testing'
  source_content_hash: sha256:959aaf2d54e56b982f16adf090ec093448badb3dc68d24ec875b1b31e218bdbb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - bootRanges
  - nullranges
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# spline-glm-fitting-overlap-counts

## Summary

Fits penalized-spline (P-spline) generalized linear models to observed and bootRanges block-bootstrap null overlap counts between differential expression genes and differential accessibility peaks, to identify data-driven significance thresholds (SNPs -log10(P) > 8, genes |logFC| > 2) where the observed statistic separates from the bootstrap null distribution.

## When to use

Use when bootRanges overlap counts between differential expression genes (logFC) and differential accessibility peaks must be converted into an optimized, data-driven logFC or SNP -log10(P) significance threshold rather than an arbitrary cutoff.

## When NOT to use

- When a fixed a priori cutoff (e.g. standard FDR threshold) is acceptable and no data-driven optimized threshold is needed
- When overlap counts come from conventional shuffling/permutation nulls rather than bootRanges block-bootstrap nulls, since the narrower null distributions undermine the divergence-based threshold recovery
- When the input lacks matched differential statistics (logFC for genes, -log10(P) for SNPs) to use as the spline smooth-term variable

## Inputs

- Observed overlap count table of differential expression genes (with logFC) and differential accessibility peaks (Alasoo 2018 macrophage dataset)
- BootRanges block-bootstrap null overlap count tables (multiple null sets from segBootranges.R)
- SNP -log10(P-value) statistics paired with gene log fold change statistics

## Outputs

- Observed penalized-spline GLM fit over logFC (conditional density plot from fit_splines.R)
- Null distribution of spline fits across bootRanges null sets
- fitted_thresholds.csv containing recovered thresholds (SNPs -log10(P) > 8, genes |logFC| > 2)

## How to apply

Load the observed and bootstrapped overlap count tables linking differential expression genes (logFC) and differential accessibility peaks from the Alasoo 2018 macrophage dataset produced with bootRanges. Fit a generalized linear model with a penalized-spline (P-spline) smooth term on logFC to the observed overlap counts, mirroring the fit_splines.R step that produces the conditional density plot. Fit the same spline GLMs to each bootRanges null set to build a null distribution of spline fits over logFC, then locate the logFC (and corresponding SNP -log10(P)) values where the observed spline diverges from the null spline envelope. Rationale: the block bootstrap preserves local genomic correlation structure, giving a more reliable null envelope than shuffling, so divergence of the observed spline from that envelope marks sets where the z score reflects strong separation; the recovered thresholds should equal SNPs -log10(P) > 8 and genes |logFC| > 2, written to fitted_thresholds.csv.

## Related tools

- **bootRanges** (Generates block-bootstrapped genomic ranges (null sets) whose overlap counts provide the null spline distribution against which the observed spline is compared to recover thresholds) — https://bioconductor.org/packages/nullranges
- **nullranges** (R/Bioconductor package hosting bootRanges functions used to build the null sets feeding the spline GLM fitting) — https://bioconductor.org/packages/nullranges

## Examples

```
Rscript script/macrophage.R
```

## Evaluation signals

- Recovered thresholds from the spline divergence match SNPs -log10(P) > 8 and genes |logFC| > 2
- Observed spline diverges from the bootRanges null spline envelope exactly in the threshold region reported in fitted_thresholds.csv
- z score at the selected thresholds reflects strong separation of the observed statistic from the bootstrap statistic distribution (var.R variance of the rate of overlaps)
- Conditional density plot from fit_splines.R reproduces the paper's macrophage.R overlap-count spline fit
- Overlap counts used as GLM input are consistent between observed and bootstrap tables (same peak-gene linkage definition)

## Limitations

- Shuffling or permutation nulls may produce overly narrow test statistic null distributions and over-estimated statistical significance, so threshold recovery requires block-bootstrap nulls that preserve local genomic correlation structure
- Thresholds are specific to the Alasoo 2018 macrophage differential expression/accessibility data; the values (SNPs -log10(P) > 8, genes |logFC| > 2) do not transfer directly to other datasets

## Evidence

- [abstract] bootRanges can be used in more complex analyses, such as accessing correlations between CREs and genes across cell types or providing optimized thresholds, e.g. logFC from differential analysis: [evidence span withheld — anchor status: not_found]
- [abstract] Block bootstrap preserves local genomic correlation structure and generates more reliable null distributions: "while creating new range sets with a block bootstrap preserves local genomic correlation structure and generates more reliable null distributions."
- [intro] Derive overlap counts of differential expression genes and differential accessibility peaks, and fit generalized splines to optimize DEG logFC thresholds: [evidence span withheld — anchor status: not_found]
- [intro] fit_splines.R (conditional density plot) workflow step from article intro: "fit_splines.R (conditional density plot)"
- [readme] README macrophage.R description: "Derive overlap counts of differential expression genes and differential accessibility peaks.
And fitted peneralized splines to optimize DEG logFC thresholds."
- [readme] README var.R description: "*var.R*(Variance of the rate of overlaps and $z$ score)"
