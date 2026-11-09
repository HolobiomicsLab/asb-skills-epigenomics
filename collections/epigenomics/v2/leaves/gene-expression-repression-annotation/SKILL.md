---
name: gene-expression-repression-annotation
description: 'Use when evaluating H3K27me3 (or other repressive-mark) ChIP-seq domain
  calls against functional ground truth: you have matched expression profiles for
  the same cell types and need per-cell-type sets of repressed genes to compute gene-body
  coverage fractions.'
when_to_use_negative:
- The cell type's ChIP-seq data has no matched gene expression profile available from
  the same source, so repression cannot be annotated consistently
- The goal is sharp-peak (point-source) evaluation such as TF binding site precision-recall,
  where whole-gene-body coverage of repressed genes is not a meaningful metric
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: RECOGNICER
  role: Produces the H3K27me3 broad domains (ENCODE broadPeak format) whose coverage
    of repressed gene bodies is evaluated against the expression-based annotations
  repo: github.com/zanglab/recognicer
- name: SICER
  role: Comparison method whose H3K27me3 domain calls are scored against the same
    repressed-gene annotations
- name: RSEG
  role: HMM-based broad domain caller compared against RECOGNICER using the repressed-gene
    coverage benchmark
- name: MUSIC
  role: Multiscale decomposition peak caller compared against RECOGNICER using the
    repressed-gene coverage benchmark
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s40484-020-0225-2
    title: 'RECOGNICER: A coarse‐graining approach for identifying broad domains from
      ChIP‐seq data'
  source_content_hash: sha256:cf359d6c4a3f3012cdcad3653165100d0a6b14fa1a77a496ba27999ba7c489a5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MUSIC
  - RECOGNICER
  - RSEG
  - SICER
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# gene-expression-repression-annotation

## Summary

Annotates genes as repressed or not repressed using matched gene expression profiles (microarray or RNA-seq) for each cell type, providing the functional ground truth used to benchmark H3K27me3 broad domain callers such as RECOGNICER against SICER, RSEG, and MUSIC.

## When to use

Use when evaluating H3K27me3 (or other repressive-mark) ChIP-seq domain calls against functional ground truth: you have matched expression profiles for the same cell types and need per-cell-type sets of repressed genes to compute gene-body coverage fractions.

## When NOT to use

- The cell type's ChIP-seq data has no matched gene expression profile available from the same source, so repression cannot be annotated consistently
- The goal is sharp-peak (point-source) evaluation such as TF binding site precision-recall, where whole-gene-body coverage of repressed genes is not a meaningful metric

## Inputs

- Matched gene expression profiles (microarray or RNA-seq) for each ENCODE cell type
- H3K27me3 ChIP-seq broad domain calls (ENCODE broadPeak format) from each method being benchmarked
- Gene body coordinates for the relevant genome annotation

## Outputs

- Per-cell-type binary repressed/not-repressed gene annotations
- Per-cell-type, per-method fractions of repressed genes fully covered by a single H3K27me3 domain
- Comparison table (h3k27me3_domain_coverage_comparison.tsv) of coverage fractions across the 9 cell types and 4 methods

## How to apply

For each of the ENCODE cell types, retrieve the matched gene expression profile (microarray or RNA-seq) that pairs with the H3K27me3 ChIP-seq data, and classify each gene as repressed or not repressed based on its expression level. These repressed-gene labels are then intersected with called domains: for each method (RECOGNICER, SICER, RSEG, MUSIC), compute the fraction of repressed genes whose entire gene body falls within a single H3K27me3 domain. The rationale is H3K27me3's known association with repressive gene expression, so a caller that recovers functionally integral domains should cover whole gene bodies of repressed genes more often — RECOGNICER achieves this in 8 of 9 ENCODE cell types. Assemble the per-cell-type, per-method coverage fractions into a comparison table (e.g., h3k27me3_domain_coverage_comparison.tsv) and confirm the expected benchmark outcome.

## Related tools

- **RECOGNICER** (Produces the H3K27me3 broad domains (ENCODE broadPeak format) whose coverage of repressed gene bodies is evaluated against the expression-based annotations) — github.com/zanglab/recognicer
- **SICER** (Comparison method whose H3K27me3 domain calls are scored against the same repressed-gene annotations)
- **RSEG** (HMM-based broad domain caller compared against RECOGNICER using the repressed-gene coverage benchmark)
- **MUSIC** (Multiscale decomposition peak caller compared against RECOGNICER using the repressed-gene coverage benchmark)

## Evaluation signals

- The assembled h3k27me3_domain_coverage_comparison.tsv reproduces the reported outcome: RECOGNICER achieves the highest repressed-gene full-gene-body coverage fraction in 8 of the 9 ENCODE cell types
- Each cell type's repressed-gene set is derived from the expression profile matched to that same cell type, not from a different cell type's data
- Coverage is counted only for repressed genes whose ENTIRE gene body falls within a SINGLE domain, not partially covered genes
- All four methods (RECOGNICER, SICER, RSEG, MUSIC) are scored on the same cell types using their default parameters, enabling a fair comparison table

## Limitations

- Repression annotation quality depends on the availability and quality of matched expression profiling (microarray or RNA-seq) for each cell type
- The benchmark covers only the 9 ENCODE cell types that have both high-quality H3K27me3 ChIP-seq and matched expression data, so results may not generalize to other cell types or marks

## Evidence

- [results] Comparison of methods by functional association of domains: [evidence span withheld — anchor status: not_found]
- [other] Workflow step: annotate repressed genes from matched expression data and compute coverage: [evidence span withheld — anchor status: not_found]
