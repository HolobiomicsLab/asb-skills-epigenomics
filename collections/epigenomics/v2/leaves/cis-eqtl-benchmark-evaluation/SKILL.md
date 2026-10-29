---
name: cis-eqtl-benchmark-evaluation
description: Use when a list of predicted enhancer–gene (gene–peak) pairs must be
  validated against an independent cis-eQTL reference set and summarized with AUROC/AUPRC
  benchmark metrics.
when_to_use_negative:
- No independent cis-eQTL or functional validation set exists for the cell type/stimulus
  — AUROC/AUPRC against a mismatched positive set is not meaningful
- The goal is de novo enhancer–gene prediction rather than benchmarking an existing
  ranked prediction list
- Input data are bulk ATAC/RNA or lack single-cell barcode matching, since the linkage
  model depends on per-cell imputed expression–accessibility coupling
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_0769
- http://edamontology.org/topic_3173
tools:
- name: SnapATAC
  role: Provides predictGenePeakPair to fit the logistic-regression association between
    imputed gene expression and linked distal peak accessibility, generating the ranked
    gene–peak pairs evaluated against cis-eQTLs
  repo: https://github.com/r3fang/SnapATAC
- name: R
  role: Runtime environment for SnapATAC v1 (requires R >= 3.4.0 & < 3.6.0 because
    rhdf5 does not work with R 3.6); used to run prediction and compute AUROC/AUPRC
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-21583-9
    title: Comprehensive analysis of single cell ATAC-seq data with SnapATAC
  source_content_hash: sha256:9312cbd732774a8fe04fd8d832d1d365a20fa2e7ebe74418e490b05c8f2d06a8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/r3fang/SnapATAC
  tools:
  - R
  - SnapATAC
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_0769
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# cis-eQTL-benchmark-evaluation

## Summary

Benchmarks computational enhancer–gene linkage predictions against experimentally derived cis-eQTLs by ranking predicted gene-peak pairs and computing AUROC and AUPRC. Provides quantitative evidence that imputed expression–accessibility associations recover validated regulatory links (AUROC = 0.66, AUPRC = 0.68 in the SnapATAC PBMC analysis).

## When to use

Use when a list of predicted enhancer–gene (gene–peak) pairs must be validated against an independent cis-eQTL reference set and summarized with AUROC/AUPRC benchmark metrics.

## When NOT to use

- No independent cis-eQTL or functional validation set exists for the cell type/stimulus — AUROC/AUPRC against a mismatched positive set is not meaningful
- The goal is de novo enhancer–gene prediction rather than benchmarking an existing ranked prediction list
- Input data are bulk ATAC/RNA or lack single-cell barcode matching, since the linkage model depends on per-cell imputed expression–accessibility coupling

## Inputs

- Ranked list of predicted gene–peak (enhancer–gene) pairs with association scores from predictGenePeakPair
- Independent cis-eQTL positive reference set (e.g., Fairfax et al. monocyte IFN-γ/LPS eQTLs)
- Imputed gene expression and binarized distal enhancer accessibility matrix from matched scRNA/scATAC data
- Set of target genes for evaluation (e.g., 3000 PBMC differentially expressed genes)

## Outputs

- gene_peak_pair_predictions.csv containing ranked gene–peak pairs with benchmark metrics
- AUROC value over the ranked predictions (expected ~0.66 for the 10X_PBMC_15K/Fairfax benchmark)
- AUPRC value over the ranked predictions (expected ~0.68 for the 10X_PBMC_15K/Fairfax benchmark)

## How to apply

Run enhancer–gene linkage prediction (e.g., SnapATAC's predictGenePeakPair) on matched scATAC/scRNA data — in the reference workflow, the 10X_PBMC_15K PBMC snap object with matched PBMC scRNA-seq, imputing gene expression from accessibility and binarizing distal enhancer accessibility, restricted to 3000 PBMC differentially expressed genes. Rank the resulting gene–peak pairs by the fitted logistic-regression association score. Treat an independent cis-eQTL set (Fairfax monocyte interferon-γ/LPS stimulation eQTLs) as positives, and compute AUROC and AUPRC over the ranked predictions. Report the metrics alongside the predictions (e.g., write predictions and metrics to gene_peak_pair_predictions.csv); AUROC = 0.66 and AUPRC = 0.68 constitute 'reasonable prediction power' per the article. Use the metrics comparatively — AUPRC is informative given the large number of unlinked gene-peak pairs in the negative class.

## Related tools

- **SnapATAC** (Provides predictGenePeakPair to fit the logistic-regression association between imputed gene expression and linked distal peak accessibility, generating the ranked gene–peak pairs evaluated against cis-eQTLs) — https://github.com/r3fang/SnapATAC
- **R** (Runtime environment for SnapATAC v1 (requires R >= 3.4.0 & < 3.6.0 because rhdf5 does not work with R 3.6); used to run prediction and compute AUROC/AUPRC)

## Examples

```
gene.pairs <- predictGenePeakPair(obj, input.mat=pmat, gene.name, gene.loci, do.par=TRUE, num.cores=8); auroc/auprc computed from ranked pairs vs Fairfax IFN-g/LPS cis-eQTLs -> gene_peak_pair_predictions.csv
```

## Evaluation signals

- AUROC over the ranked gene–peak pairs against the cis-eQTL positive set should be ~0.66 (reported as reasonable prediction power)
- AUPRC over the ranked predictions should be ~0.68
- Predictions file (gene_peak_pair_predictions.csv) contains both the ranked gene–peak pairs and the benchmark metrics
- Prediction inputs are restricted to the 3000 PBMC differentially expressed genes, with distal enhancer accessibility binarized
- predictGenePeakPair runs per gene locus with multi-core execution (do.par, num.cores) completing without errors

## Limitations

- Performance is moderate (AUROC = 0.66, AUPRC = 0.68), so predictions should be treated as putative rather than definitive enhancer–gene assignments
- The benchmark is specific to monocyte IFN-γ/LPS cis-eQTLs; transferability to other tissues or stimuli is not established by this evaluation
- SnapATAC v1 requires R (>= 3.4.0 & < 3.6.0) since R 3.6 breaks the rhdf5 package; SnapATAC 2.0 (SnapATAC2) is a separate Python/Rust package

## Evidence

- [results] SnapATAC's enhancer–gene predictions match the expression quantitative trait loci (cis-eQTLs) derived from interferon-γ and lipopolysaccharide stimulation of monocytes, with reasonable prediction power (AUROC = 0.66, AUPRC = 0.68).: [evidence span withheld — anchor status: not_found]
- [methods] predictGenePeakPair fits logistic-regression association between imputed gene expression and linked distal peak accessibility: [evidence span withheld — anchor status: not_found]
- [methods] SnapATAC links distal elements to putative target genes via the 10X_PBMC_15K example workflow: "SnapATAC links distal elements to putative target genes"
- [methods] SnapATAC integrates scRNA and scATAC data: "SnapATAC integrates scRNA and scATAC"
- [readme] SnapATAC requires R 3.4.0 or later but not 3.6.0 due to rhdf5 incompatibility: "R (>= 3.4.0 & < 3.6.0) (SnapATAC) (3.6 does not work for rhdf5 package);"
