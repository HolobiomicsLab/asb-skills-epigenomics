---
name: genomic-interval-coverage-calculation
description: Use when you need to quantify how completely called broad ChIP-seq domains
  (e.g., ENCODE broadPeak-format H3K27me3 domains) cover annotated gene bodies of
  repressed genes across multiple methods or samples.
when_to_use_negative:
- The ChIP-seq signal marks punctate (narrow) peaks rather than kb-to-Mb broad domains,
  where whole-gene-body coverage is not the appropriate functional metric
- No matched gene expression data are available to define repressed genes, so the
  coverage fractions cannot be stratified by repression status
- You only need per-base-pileup coverage tracks (e.g., bedGraph visualization) rather
  than whole-gene-body containment statistics
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0654
tools:
- name: RECOGNICER
  role: Calls H3K27me3 broad domains (default 150 bp fragment, 200 bp window, block
    transformation factor 3) output in ENCODE broadPeak format for the coverage comparison
  repo: github.com/zanglab/recognicer
- name: SICER
  role: Baseline broad-domain caller run with default parameters to generate comparison
    domains
- name: RSEG
  role: HMM-based baseline broad-domain caller run with default parameters to generate
    comparison domains
- name: MUSIC
  role: Multiscale-decomposition baseline caller run with default parameters to generate
    comparison domains
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s40484-020-0225-2
    title: 'RECOGNICER: A coarse‐graining approach for identifying broad domains from
      ChIP‐seq data'
  source_content_hash: sha256:d4f6759ba06c3eda27eba37f64df3383e37ac5fc65cbf63199a056a1cc3f9344
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MUSIC
  - RECOGNICER
  - RSEG
  - SICER
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# Genomic interval coverage calculation

## Summary

Computes, per cell type and per broad-domain caller, the fraction of repressed genes whose entire gene body falls within a single called H3K27me3 domain, serving as the functional-association benchmark in the RECOGNICER paper.

## When to use

Use when you need to quantify how completely called broad ChIP-seq domains (e.g., ENCODE broadPeak-format H3K27me3 domains) cover annotated gene bodies of repressed genes across multiple methods or samples.

## When NOT to use

- The ChIP-seq signal marks punctate (narrow) peaks rather than kb-to-Mb broad domains, where whole-gene-body coverage is not the appropriate functional metric
- No matched gene expression data are available to define repressed genes, so the coverage fractions cannot be stratified by repression status
- You only need per-base-pileup coverage tracks (e.g., bedGraph visualization) rather than whole-gene-body containment statistics

## Inputs

- H3K27me3 ChIP-seq mapped-read BED files (treatment and matched control) per ENCODE cell type
- Called broad domains in ENCODE broadPeak format from RECOGNICER, SICER, RSEG, and MUSIC
- Matched gene expression profiles (microarray or RNA-seq) for the same 9 cell types
- Gene body coordinate annotations per cell type

## Outputs

- Per-cell-type, per-method fraction of repressed genes with entire gene body within a single H3K27me3 domain
- Comparison table (h3k27me3_domain_coverage_comparison.tsv)

## How to apply

Run each broad domain caller (RECOGNICER with fragment size 150 bp, window size 200 bp, block transformation factor 3; SICER, RSEG, MUSIC with defaults) on the same treatment/control ChIP-seq BED files to produce domains in ENCODE broadPeak format. Annotate genes as repressed or not using the matched gene expression profile (microarray or RNA-seq) for each of the 9 ENCODE cell types. For each method and cell type, intersect the called domains with gene-body coordinates and compute the fraction of repressed genes whose entire gene body is contained within a single H3K27me3 domain. Assemble the per-cell-type, per-method fractions into a comparison table (h3k27me3_domain_coverage_comparison.tsv); the correct outcome is that RECOGNICER achieves the highest coverage fraction in 8 of the 9 cell types.

## Related tools

- **RECOGNICER** (Calls H3K27me3 broad domains (default 150 bp fragment, 200 bp window, block transformation factor 3) output in ENCODE broadPeak format for the coverage comparison) — github.com/zanglab/recognicer
- **SICER** (Baseline broad-domain caller run with default parameters to generate comparison domains)
- **RSEG** (HMM-based baseline broad-domain caller run with default parameters to generate comparison domains)
- **MUSIC** (Multiscale-decomposition baseline caller run with default parameters to generate comparison domains)

## Evaluation signals

- The output table h3k27me3_domain_coverage_comparison.tsv has one coverage fraction per method per cell type (4 methods x 9 cell types)
- RECOGNICER achieves the highest repressed-gene full-gene-body coverage fraction among the four methods in 8 of the 9 cell types
- All domain inputs parse as valid ENCODE broadPeak format (chrom, start, end, name, score, strand, signalValue, pValue, qValue), with score = fold change and signalValue = ChIP read count for RECOGNICER output
- Repressed-gene labels are derived from the expression profile matched to the same cell type as the ChIP-seq data, not from a different cell type
- A repressed gene counts as covered only when its entire gene body lies within a single called domain, not split across multiple domains

## Limitations

- RECOGNICER could falsely merge two domains that are separated by gaps on the fine scale into one domain on the coarse scale; users should monitor individual cases and may incorporate featured or masked regions
- The benchmark depends on availability of cell types with both high-quality H3K27me3 ChIP-seq and matched gene expression profiling data (only 9 ENCODE cell types qualified)
- Coverage fractions are specific to repressed genes and H3K27me3 biology; they do not directly validate domain calls for other histone marks

## Evidence

- [other] For 8 out of 9 ENCODE cell types, RECOGNICER-identified H3K27me3 domains cover more repressed genes in their entire gene bodies than the other 3 methods: [evidence span withheld — anchor status: not_found]
- [results] Functional association as the evaluation basis: [evidence span withheld — anchor status: not_found]
- [readme] RECOGNICER output is broadPeak-format domains: [evidence span withheld — anchor status: not_found]
