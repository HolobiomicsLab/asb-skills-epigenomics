---
name: auc-prc-performance-computation
description: Use when a set of ranked predicted regulatory interactions (e.g., gene–peak
  pairs) needs to be evaluated against a trusted positive set such as cis-eQTLs to
  quantify prediction performance.
when_to_use_negative:
- The enhancer–gene predictions themselves have not yet been generated — this skill
  evaluates a ranked prediction table, it does not produce it (use predictGenePeakPair
  first).
- No independent positive reference (e.g., cis-eQTLs) exists for the tissue/stimulation
  context, making AUROC/AUPRC against a truth set undefined.
- The task is cell-type clustering or dimensionality reduction of scATAC-seq data,
  where SnapATAC's diffusion-map and Leiden clustering steps — not linkage benchmarking
  — are the relevant workflow.
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: SnapATAC
  role: Provides predictGenePeakPair to produce the ranked gene–peak pair scores that
    are evaluated via AUROC/AUPRC against cis-eQTLs; implemented in R (>= 3.4.0 &
    < 3.6.0).
  repo: https://github.com/r3fang/SnapATAC
- name: R
  role: Analysis environment in which the predictions are ranked, metrics computed,
    and gene_peak_pair_predictions.csv written.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-21583-9
    title: Comprehensive analysis of single cell ATAC-seq data with SnapATAC
  source_content_hash: sha256:586e3136ed1b77ee18c6575d85fe98f4d4b38b9bbb98ca49be6912ec82480d9a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/r3fang/SnapATAC
  tools:
  - R
  - SnapATAC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# auc-prc-performance-computation

## Summary

Benchmark ranked enhancer–gene predictions against a curated positive reference set (cis-eQTLs from stimulated monocytes) by computing AUROC and AUPRC, providing a quantitative measure of enhancer–gene linkage prediction power.

## When to use

Use when a set of ranked predicted regulatory interactions (e.g., gene–peak pairs) needs to be evaluated against a trusted positive set such as cis-eQTLs to quantify prediction performance.

## When NOT to use

- The enhancer–gene predictions themselves have not yet been generated — this skill evaluates a ranked prediction table, it does not produce it (use predictGenePeakPair first).
- No independent positive reference (e.g., cis-eQTLs) exists for the tissue/stimulation context, making AUROC/AUPRC against a truth set undefined.
- The task is cell-type clustering or dimensionality reduction of scATAC-seq data, where SnapATAC's diffusion-map and Leiden clustering steps — not linkage benchmarking — are the relevant workflow.

## Inputs

- Ranked gene–peak pair predictions with association scores from predictGenePeakPair in SnapATAC (R)
- Positive reference set: Fairfax monocyte interferon-γ and LPS cis-eQTLs
- Imputed gene expression matrix from matched PBMC scRNA-seq data (SnapATAC snap object, 10X_PBMC_15K)
- Binarized distal enhancer accessibility matrix restricted to 3000 PBMC differentially expressed genes

## Outputs

- AUROC and AUPRC values quantifying enhancer–gene linkage prediction performance (expected: AUROC = 0.66, AUPRC = 0.68)
- gene_peak_pair_predictions.csv containing ranked gene–peak pairs and benchmark metrics

## How to apply

Generate the ranked predictions first: in SnapATAC (R), use predictGenePeakPair(obj, input.mat, gene.name, gene.loci, do.par, num.cores) to fit the logistic-regression association between imputed gene expression (imputed from accessibility using matched scRNA-seq data) and binarized distal peak accessibility, restricted here to the 3000 PBMC differentially expressed genes. Rank all gene–peak pairs by the model score, then treat the Fairfax monocyte interferon-γ/LPS cis-eQTL set as the positive reference. Sweep the ranking to compute AUROC and AUPRC over the ordered predictions. Write the predictions together with the benchmark metrics to gene_peak_pair_predictions.csv; for this PBMC analysis the expected performance is AUROC = 0.66 and AUPRC = 0.68, so results in this ballpark indicate correct application.

## Related tools

- **SnapATAC** (Provides predictGenePeakPair to produce the ranked gene–peak pair scores that are evaluated via AUROC/AUPRC against cis-eQTLs; implemented in R (>= 3.4.0 & < 3.6.0).) — https://github.com/r3fang/SnapATAC
- **R** (Analysis environment in which the predictions are ranked, metrics computed, and gene_peak_pair_predictions.csv written.)

## Examples

```
run predictGenePeakPair(obj, input.mat, gene.name, gene.loci, do.par=TRUE, num.cores=8); write predictions and metrics (AUROC = 0.66, AUPRC = 0.68) to gene_peak_pair_predictions.csv
```

## Evaluation signals

- AUROC and AUPRC values are reported alongside predictions and fall in a reasonable range (reference run: AUROC = 0.66, AUPRC = 0.68).
- gene_peak_pair_predictions.csv is produced and contains both the ranked gene–peak pairs and the benchmark metrics.
- Positive set is exactly the Fairfax monocyte interferon-γ/LPS cis-eQTL collection, matched by gene locus to the predicted pairs.
- Predictions were computed only over the restricted set of 3000 PBMC differentially expressed genes using imputed expression and binarized distal accessibility.
- Each gene's linked peaks derive from the logistic-regression fit in predictGenePeakPair and are ordered by association strength before metric computation.

## Limitations

- Prediction power is moderate (AUROC = 0.66, AUPRC = 0.68), so enhancer–gene links should be treated as putative, not definitive.
- Benchmarking relies on cis-eQTLs from interferon-γ/LPS-stimulated monocytes as a proxy positive set, which may not fully represent PBMC enhancer–gene biology.
- Functional validation of candidate regulatory elements, especially those from rare cell types, remains an open need per the article's discussion.

## Evidence

- [results] SnapATAC's enhancer–gene predictions match the expression quantitative trait loci (cis-eQTLs) derived from interferon-γ and lipopolysaccharide stimulation of monocytes, with reasonable prediction power (AUROC = 0.66, AUPRC = 0.68).: [evidence span withheld — anchor status: not_found]
- [methods] Run predictGenePeakPair(obj, input.mat, gene.name, gene.loci, do.par, num.cores) to fit the logistic-regression association between imputed gene expression and the accessibility of linked distal peaks for each gene locus, using multiple cores.: [evidence span withheld — anchor status: not_found]
- [methods] Rank the resulting gene-peak pairs and evaluate them against the Fairfax monocyte interferon-γ/LPS cis-eQTL set as the positive reference.: [evidence span withheld — anchor status: not_found]
- [readme] SnapATAC links distal elements to putative target genes: "SnapATAC links distal elements to putative target genes"
