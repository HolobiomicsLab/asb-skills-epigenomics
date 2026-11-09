---
name: cross-method-loop-set-overlap-analysis
description: Use when comparing loop sets from a supervised predictor (Peakachu) with
  enrichment-based callers (HiCCUPS, Fit-Hi-C) on the same Hi-C matrix, to quantify
  overlap and orthogonal validation.
when_to_use_negative:
- The input matrix is low depth (e.g., 1.5% down-sample) such that Fit-Hi-C retains
  no interactions at q < 1e-5 and HiCCUPS with default parameters yields less than
  1/3 of Peakachu loops — cutoffs must be retuned rather than compared as-is
- The goal is de novo loop detection rather than method benchmarking — use peakachu
  train/score_genome directly
- Data are from a platform without a dedicated enrichment-based loop caller (e.g.,
  DNA SPRITE), making the three-way comparison impossible
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Peakachu
  role: Random Forest classifier used to score genome-wide loop probabilities (score_genome)
    and select non-redundant loop pixels (pool) at threshold 0.9-0.95
  repo: github.com/tariks/peakachu
- name: HiCCUPS
  role: Juicer enrichment-based loop caller using a Poisson test on local background,
    run on the same 10 kb matrix for comparison
  repo: https://github.com/aidenlab/juicer
- name: Fit-Hi-C
  role: Distance-dependent two-pass spline caller; second pass filtered at q-value
    < 1e-5, top 140,000 interactions pooled by p-value
- name: Juicebox
  role: Visualization of the .hic maps and loop calls from the comparison
  repo: https://github.com/aidenlab/Juicebox
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-17239-9
    title: A supervised learning framework for chromatin loop detection in genome-wide
      contact maps
  source_content_hash: sha256:858af80e89ce955974b15bd211789b34ad156b38b2bd2a72a864fab8999bce63
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fit-Hi-C
  - HiCCUPS
  - Juicebox
  - Peakachu
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# cross-method-loop-set-overlap-analysis

## Summary

Benchmarks Peakachu's Random Forest-predicted chromatin loops against enrichment-based loop callers (HiCCUPS, Fit-Hi-C) by intersecting the three loop sets and computing method-specific validation ratios against orthogonal data in GM12878 Hi-C.

## When to use

Use when comparing loop sets from a supervised predictor (Peakachu) with enrichment-based callers (HiCCUPS, Fit-Hi-C) on the same Hi-C matrix, to quantify overlap and orthogonal validation.

## When NOT to use

- The input matrix is low depth (e.g., 1.5% down-sample) such that Fit-Hi-C retains no interactions at q < 1e-5 and HiCCUPS with default parameters yields less than 1/3 of Peakachu loops — cutoffs must be retuned rather than compared as-is
- The goal is de novo loop detection rather than method benchmarking — use peakachu train/score_genome directly
- Data are from a platform without a dedicated enrichment-based loop caller (e.g., DNA SPRITE), making the three-way comparison impossible

## Inputs

- Rao2014-GM12878-MboI-allreps-filtered.10kb.cool (10 kb binned GM12878 Hi-C cooler file)
- Trained Peakachu random forest models (per-chromosome pickled models)
- Fit-Hi-C two-pass spline output with q-values
- HiCCUPS (Juicer) loop list from the same 10 kb matrix
- Orthogonal validation sets (e.g., ENCODE ChIP-Seq peaks for 133 TFs and 10 histone modifications in GM12878; gm12878.mumbach.h3k27ac-hichip.hg19.bedpe)

## Outputs

- Three-way Venn overlap summary of Peakachu, HiCCUPS, and Fit-Hi-C loop sets
- method_comparison_summary.tsv with overlap percentages and per-method validation ratios
- Method-specific loop lists (Peakachu-only, HiCCUPS-only, Fit-Hi-C-only)

## How to apply

Load the Rao2014-GM12878 10 kb cooler file and generate the Peakachu genome-wide loop set with 'peakachu score_genome' followed by 'peakachu pool' at a probability threshold of 0.9-0.95. Run HiCCUPS (Juicer) on the same 10 kb matrix, which models local background via a Poisson test, and run Fit-Hi-C's distance-dependent two-pass spline, filtering the second pass at q-value < 1e-5, sorting by p-value, and pooling the top 140,000 interactions. Intersect the three loop sets to build a three-way Venn overlap summary, reporting the fraction of Peakachu loops shared with either enrichment-based method. Finally, compute validation ratios of method-specific loops against orthogonal validation sets (e.g., ENCODE ChIP-Seq peaks and HiChIP data in GM12878) and assemble into a method_comparison_summary.tsv. The rationale is that each caller models a different signal class, so overlap plus validation rates reveal whether Peakachu's unique short-range loops are as well supported as enrichment-based calls.

## Related tools

- **Peakachu** (Random Forest classifier used to score genome-wide loop probabilities (score_genome) and select non-redundant loop pixels (pool) at threshold 0.9-0.95) — github.com/tariks/peakachu
- **HiCCUPS** (Juicer enrichment-based loop caller using a Poisson test on local background, run on the same 10 kb matrix for comparison) — https://github.com/aidenlab/juicer
- **Fit-Hi-C** (Distance-dependent two-pass spline caller; second pass filtered at q-value < 1e-5, top 140,000 interactions pooled by p-value)
- **Juicebox** (Visualization of the .hic maps and loop calls from the comparison) — https://github.com/aidenlab/Juicebox

## Examples

```
peakachu score_genome -r 10000 -p Rao2014-GM12878-MboI-allreps-filtered.10kb.cool --clr-weight-name weight -m models -O score.txt --minimum-prob 0.9; peakachu pool -r 10000 -O loops.txt score.txt
```

## Evaluation signals

- The overlap summary reproduces the reported 72% (12,398/17,171) of Peakachu loops overlapping with HiCCUPS or Fit-Hi-C predictions
- Validation ratios reproduce 58%/48%/40% for Peakachu-/HiCCUPS-/Fit-Hi-C-specific loops respectively
- Fit-Hi-C survivors are filtered at q-value < 1e-5 and the top 140,000 by p-value are pooled
- Peakachu loops are pooled at a probability threshold in the 0.9-0.95 range
- All three methods are run on the same Rao2014-GM12878 10 kb matrix so intersection is on identical genomic bins

## Limitations

- Each method detects a unique set of loops, so the complete set of chromatin loops might not be captured by any single method
- HiCCUPS with default parameters output less than 1/3 of the Peakachu loops at each down-sample rate, complicating default-parameter comparisons
- At severe depth reduction (1.5%), no Fit-Hi-C interactions remain at q < 1e-5 and cutoffs must be relaxed (q < 0.1; HiCCUPS -f 0.9 -t 3.6,1,1,1)

## Evidence

- [abstract] We compare Peakachu with current enrichment-based approaches, and find that Peakachu identifies a unique set of short-range interactions.: "We compare Peakachu with current enrichment-based approaches, and find that Peakachu identifies a unique set of short-range interactions."
- [intro] Fit-Hi-C performs a distance-dependent spline fitting procedure: "Fit-Hi-C21 performs a distance-dependent spline fitting procedure to refine its global background"
- [intro] HiCCUPS incorporates local background into its model and utilizes the Poisson test: "HiCCUPS8,23 incorporates local background into its model and utilizes the Poisson test"
- [discussion] The results of the 2nd spline pass were then filtered with the q-value cutoff < 1e-5: "The results of the 2nd spline pass were then filtered with the q-value cutoff < 1e-5"
- [methods] The pool function serves to select the most significant non-redundant results: "The pool function serves to select the most significant non-redundant results from per-pixel probabilities calculated by the score functions"
- [readme] Peakachu supports .hic and .cool formats: "Peakachu (version>=1.1.2) now supports both [.hic](https://github.com/aidenlab/juicer/wiki/Data) and [.cool](https://cooler.readthedocs.io/en/latest/datamodel.html) formats."
