---
name: block-bootstrap-null-range-generation
description: Use when testing overlap/enrichment between two genomic range sets (e.g.
  ChIP-seq peaks, caQTLs, CREs) and a null distribution is needed that preserves local
  genomic correlation structure rather than shuffling ranges.
when_to_use_negative:
- When the range sets have no local genomic correlation structure to preserve, so
  simple shuffling is an acceptable null
- When the downstream analysis is not range-overlap/enrichment hypothesis testing
  (e.g. DEG logFC threshold optimization via generalized splines, macrophage.R, or
  multiome mean correlation, multiomicsCor.R), where bootRanges is applied differently
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_3318
tools:
- name: bootRanges
  role: Generates block-bootstrapped null genomic ranges preserving local genomic
    correlation structure
  repo: https://bioconductor.org/packages/nullranges
- name: nullranges
  role: R/Bioconductor package hosting the bootRanges function
  repo: https://bioconductor.org/packages/nullranges
- name: GSC
  role: Comparison baseline permutation/shuffling method timed against bootRanges
    on genome-scale statistics
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad190
    title: 'bootRanges: flexible generation of null sets of genomic ranges for hypothesis
      testing'
  source_content_hash: sha256:e25ff4e029efe9d444fc2851c807e78c73afc6fe5dcc5b94d3bf00395c1dd91f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GSC
  - bootRanges
  - nullranges
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# block-bootstrap-null-range-generation

## Summary

Generates null sets of genomic ranges via block bootstrapping with the bootRanges function from the R/Bioconductor nullranges package, preserving local genomic correlation structure for enrichment and hypothesis testing. Contrasts with conventional shuffling/permutation, which yields overly narrow null distributions and inflated false positive rates.

## When to use

Use when testing overlap/enrichment between two genomic range sets (e.g. ChIP-seq peaks, caQTLs, CREs) and a null distribution is needed that preserves local genomic correlation structure rather than shuffling ranges.

## When NOT to use

- When the range sets have no local genomic correlation structure to preserve, so simple shuffling is an acceptable null
- When the downstream analysis is not range-overlap/enrichment hypothesis testing (e.g. DEG logFC threshold optimization via generalized splines, macrophage.R, or multiome mean correlation, multiomicsCor.R), where bootRanges is applied differently

## Inputs

- Query and subject genomic range sets as GRanges objects (e.g. ENCODE kidney and bladder ChIP-seq peak sets)
- Segmentation-derived block lengths for matched segmental blocks
- Nominal significance threshold alpha

## Outputs

- Block-bootstrapped null range sets per iteration
- Overlap rates and z-score statistics per null iteration
- False positive rate tabulation (fpr_simulation_results.csv) for bootRanges vs shuffling
- Timing benchmark (timing_benchmark.csv, e.g. 0.30 s / 0.37 s vs 7.56 s)

## How to apply

Construct GRanges objects for the query and subject range sets (GRanges_construct.R-style preprocessing). Generate block-bootstrapped null range sets with bootRanges from nullranges, using segmentation-derived block lengths (segBootranges.R) so local genomic correlation is preserved. In parallel, generate null sets via conventional shuffling (bootstrap.R) only as a comparison baseline, since shuffling produces nulls with similar mean but much lower variance. For each null iteration compute the overlap rate and convert to a z-score statistic using variance of the rate of overlaps (var.R). Validate calibration empirically: in a no-association simulation, bootRanges FPR should sit near the nominal alpha while shuffling FPR is clearly elevated; on ENCODE kidney/bladder ChIP-seq, bootRanges timing (~0.30 s generation, ~0.37 s with overlaps vs GSC 7.56 s) confirms scalability.

## Related tools

- **bootRanges** (Generates block-bootstrapped null genomic ranges preserving local genomic correlation structure) — https://bioconductor.org/packages/nullranges
- **nullranges** (R/Bioconductor package hosting the bootRanges function) — https://bioconductor.org/packages/nullranges
- **GSC** (Comparison baseline permutation/shuffling method timed against bootRanges on genome-scale statistics)

## Examples

```
Rscript segBootranges.R  # performs bootRanges block bootstrapping after GRanges_construct.R preprocessing
```

## Evaluation signals

- In a Supplementary Section S2.1 no-association simulation (x and y with no true association), bootRanges FPR is close to the nominal alpha while the shuffling-based FPR is clearly elevated above alpha
- Shuffling null distribution has a similar mean to bootRanges null but much lower variance, indicating over-estimated significance
- Timing benchmark on ENCODE kidney/bladder peaks reproduces bootRanges ~0.30 s generation (~0.37 s with overlaps) vs GSC 7.56 s, roughly a 20-fold speedup
- Pipeline scripts (GRanges_construct.R, segBootranges.R, var.R, bootstrap.R, plot.R) run in sequence without error and outputs (fpr_simulation_results.csv, timing_benchmark.csv) are produced

## Limitations

- Conventional shuffling/permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance, so shuffled nulls are unsuitable for calibrated FPR
- Block lengths should be matched to segmentation-derived block lengths to preserve local genomic correlation; mismatched blocks may degrade null fidelity
- Timing/speedup figures (~0.30/0.37 s vs 7.56 s) are hardware-dependent

## Evidence

- [abstract] bootRanges provides fast functions for generation of block bootstrapped genomic ranges representing the null hypothesis in enrichment analysis.: "bootRanges provides fast functions for generation of block bootstrapped genomic ranges representing the null hypothesis in enrichment analysis."
- [abstract] Shuffling or permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance: "We show that shuffling or permutation schemes may result in overly narrow test statistic null distributions and over-estimation of statistical significance"
- [abstract] Block bootstrap creates new range sets that preserve local genomic correlation structure and generate more reliable null distributions: "while creating new range sets with a block bootstrap preserves local genomic correlation structure and generates more reliable null distributions."
- [readme] var.R (Variance of the rate of overlaps and z score): "*var.R*(Variance of the rate of overlaps and $z$ score)"
- [readme] bootstrap.R: perform conventional shuffling.: "*bootstrap.R*: perform conventional shuffling."
