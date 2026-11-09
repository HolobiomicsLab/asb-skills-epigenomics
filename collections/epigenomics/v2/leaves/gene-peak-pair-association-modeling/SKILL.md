---
name: gene-peak-pair-association-modeling
description: Use when you have a SnapATAC snap object for scATAC-seq cells plus matched
  scRNA-seq expression and need to connect distal accessible peaks to putative target
  genes, e.g. building a cis-regulatory network from the 10X_PBMC_15K PBMC dataset.
when_to_use_negative:
- Input is raw FASTQ/BAM — pre-processing (alignment, duplicate removal, snap-file
  creation) belongs to SnapTools, not this linkage step
- Goal is dimensionality reduction, clustering, batch correction, or motif analysis
  rather than enhancer–gene assignment
- No matched scRNA-seq expression data are available to impute gene expression, so
  gene activity cannot be modeled for the same cells
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3308
tools:
- name: SnapATAC
  role: R package that provides predictGenePeakPair to fit the logistic-regression
    association between imputed gene expression and distal peak accessibility
  repo: https://github.com/r3fang/SnapATAC
- name: SnapTools
  role: Python module that builds the snap file (AM/PM/GM/FM sessions) that supplies
    the accessibility and gene matrices consumed by this step
  repo: https://github.com/r3fang/SnapTools
- name: R
  role: Runtime environment for SnapATAC (>= 3.4.0 & < 3.6.0; 3.6 does not work for
    the rhdf5 package)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-21583-9
    title: Comprehensive analysis of single cell ATAC-seq data with SnapATAC
  source_content_hash: sha256:fb88fa22e116ea3f360fd7430f397761777d003d3fdead3390d3882ad96f87cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/r3fang/SnapATAC
  tools:
  - R
  - SnapATAC
  - SnapTools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# gene-peak-pair-association-modeling

## Summary

Fits a logistic-regression association between imputed gene expression (from matched scRNA-seq) and binarized distal enhancer accessibility in SnapATAC to link candidate regulatory elements to putative target genes. This is the enhancer–gene linkage step, benchmarked against monocyte cis-eQTLs (AUROC 0.66, AUPRC 0.68).

## When to use

Use when you have a SnapATAC snap object for scATAC-seq cells plus matched scRNA-seq expression and need to connect distal accessible peaks to putative target genes, e.g. building a cis-regulatory network from the 10X_PBMC_15K PBMC dataset.

## When NOT to use

- Input is raw FASTQ/BAM — pre-processing (alignment, duplicate removal, snap-file creation) belongs to SnapTools, not this linkage step
- Goal is dimensionality reduction, clustering, batch correction, or motif analysis rather than enhancer–gene assignment
- No matched scRNA-seq expression data are available to impute gene expression, so gene activity cannot be modeled for the same cells

## Inputs

- SnapATAC snap object for scATAC-seq cells (e.g. 10X_PBMC_15K PBMC)
- Matched PBMC scRNA-seq gene expression matrix (cell-by-gene matrix, GM)
- Gene name and gene loci (genomic coordinates) for the gene whose distal peaks are being linked
- Set of 3000 PBMC differentially expressed genes to restrict the analysis
- Fairfax monocyte interferon-γ/LPS cis-eQTL set used as positive reference for benchmarking

## Outputs

- Ranked gene–peak pair predictions (gene_peak_pair_predictions.csv)
- Benchmark metrics AUROC = 0.66 and AUPRC = 0.68 against the cis-eQTL positive set

## How to apply

Load the 10X_PBMC_15K PBMC scATAC-seq snap object and matched PBMC scRNA-seq expression data in SnapATAC (R). Impute gene expression from chromatin accessibility and binarize distal enhancer accessibility with makeBinary, restricting the analysis to the 3000 PBMC differentially expressed genes. For each gene locus, run predictGenePeakPair(obj, input.mat, gene.name, gene.loci, do.par, num.cores) across multiple cores to fit the logistic-regression association between imputed gene expression and accessibility of linked distal peaks. Rank the resulting gene-peak pairs and evaluate them against the Fairfax monocyte interferon-γ/LPS cis-eQTL set as the positive reference, computing AUROC and AUPRC, then write predictions and metrics to gene_peak_pair_predictions.csv.

## Related tools

- **SnapATAC** (R package that provides predictGenePeakPair to fit the logistic-regression association between imputed gene expression and distal peak accessibility) — https://github.com/r3fang/SnapATAC
- **SnapTools** (Python module that builds the snap file (AM/PM/GM/FM sessions) that supplies the accessibility and gene matrices consumed by this step) — https://github.com/r3fang/SnapTools
- **R** (Runtime environment for SnapATAC (>= 3.4.0 & < 3.6.0; 3.6 does not work for the rhdf5 package))

## Examples

```
predictions <- predictGenePeakPair(obj, input.mat = mat, gene.name = gene, gene.loci = loci, do.par = TRUE, num.cores = 8)
```

## Evaluation signals

- Ranked gene-peak pairs reproduce AUROC = 0.66 and AUPRC = 0.68 when evaluated against the Fairfax monocyte interferon-γ/LPS cis-eQTL positive reference
- Each candidate enhancer peak is binarized (makeBinary) rather than left as a continuous count before association fitting
- Predictions are restricted to the 3000 PBMC differentially expressed genes, not all genes
- Output table gene_peak_pair_predictions.csv contains ranked gene–peak pairs with the benchmark metrics attached

## Limitations

- Prediction power is only reasonable, not definitive: AUROC = 0.66 and AUPRC = 0.68 against monocyte cis-eQTLs, so links require downstream validation
- scATAC-seq sparsity (5–15% of peaks detected per cell) makes individual-cell accessibility signals noisy, motivating imputation from the matched scRNA-seq data
- Iterative clustering strategies require multiple rounds of clustering, aggregation, and peak calling, which can leave peaks in rare populations underrepresented before linkage

## Evidence

- [methods] SnapATAC links distal elements to putative target genes: "SnapATAC links distal elements to putative target genes"
- [full_text] predictGenePeakPair association fitting: [evidence span withheld — anchor status: not_found]
- [full_text] Imputation and binarization step: [evidence span withheld — anchor status: not_found]
- [full_text] Benchmark performance against cis-eQTLs: [evidence span withheld — anchor status: not_found]
