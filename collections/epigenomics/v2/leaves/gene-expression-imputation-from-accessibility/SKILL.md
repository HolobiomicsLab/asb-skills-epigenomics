---
name: gene-expression-imputation-from-accessibility
description: Use when you have a scATAC-seq snap object plus matched scRNA-seq expression
  data and need to link distal enhancer peaks to their putative target genes, or to
  rank enhancer–gene associations for benchmarking against eQTLs.
when_to_use_negative:
- You only need cell-type clustering or dimensionality reduction of the scATAC-seq
  data without enhancer–gene linking (use SnapATAC's diffusion-map/Leiden workflow
  instead).
- No matched scRNA-seq data are available to impute gene expression from accessibility.
- You are working in Python with SnapATAC2 (scverse) — the predictGenePeakPair workflow
  described here is from the R SnapATAC (r3fang) package.
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3308
tools:
- name: SnapATAC
  role: R package providing predictGenePeakPair to fit the logistic-regression association
    between imputed gene expression and linked distal peak accessibility for each
    gene locus
  repo: https://github.com/r3fang/SnapATAC
- name: SnapTools
  role: Preprocessing module that creates the snap file (cell-by-peak PM, cell-by-gene
    GM matrices) consumed by SnapATAC
  repo: https://github.com/r3fang/SnapTools
- name: R 3.4.0
  role: Runtime for SnapATAC; versions >= 3.4.0 & < 3.6.0 required (3.6 does not work
    for rhdf5)
- name: SnapATAC2
  role: Successor Python/Rust package implementing regulatory network analysis; linked
    from the SnapATAC README as the latest version (2.0)
  repo: https://github.com/scverse/SnapATAC2
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-21583-9
    title: Comprehensive analysis of single cell ATAC-seq data with SnapATAC
  source_content_hash: sha256:fc17b9895c6938bbe596ab4247d9641751f8c2ff38eaa0fa2fd508f4626f11d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/r3fang/SnapATAC
  tools:
  - R
  - R 3.4.0
  - SnapATAC
  - SnapATAC2
  - SnapTools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# gene-expression-imputation-from-accessibility

## Summary

Impute gene expression from scATAC-seq accessibility (using matched scRNA-seq data) and link distal candidate regulatory elements to putative target genes in SnapATAC, then benchmark the ranked gene–peak pairs against an independent cis-eQTL set (Fairfax monocyte IFN-γ/LPS; AUROC 0.66, AUPRC 0.68).

## When to use

Use when you have a scATAC-seq snap object plus matched scRNA-seq expression data and need to link distal enhancer peaks to their putative target genes, or to rank enhancer–gene associations for benchmarking against eQTLs.

## When NOT to use

- You only need cell-type clustering or dimensionality reduction of the scATAC-seq data without enhancer–gene linking (use SnapATAC's diffusion-map/Leiden workflow instead).
- No matched scRNA-seq data are available to impute gene expression from accessibility.
- You are working in Python with SnapATAC2 (scverse) — the predictGenePeakPair workflow described here is from the R SnapATAC (r3fang) package.

## Inputs

- PBMC scATAC-seq snap object (10X_PBMC_15K example dataset)
- Matched PBMC scRNA-seq gene expression matrix
- Gene loci (genomic coordinates of the genes to link)
- Positive benchmark set: Fairfax monocyte interferon-γ/LPS cis-eQTLs
- Vector of the 3000 PBMC differentially expressed genes

## Outputs

- Ranked gene–peak pair predictions (gene_peak_pair_predictions.csv)
- Benchmark metrics over ranked predictions (AUROC = 0.66, AUPRC = 0.68)

## How to apply

Load the 10X_PBMC_15K PBMC scATAC-seq snap object and the matched PBMC scRNA-seq expression matrix in SnapATAC (R). Impute gene expression from chromatin accessibility and binarize distal enhancer accessibility, restricting the analysis to the 3000 PBMC differentially expressed genes. Call predictGenePeakPair(obj, input.mat, gene.name, gene.loci, do.par, num.cores), which fits a logistic-regression association between imputed gene expression and the accessibility of linked distal peaks within each gene locus, using multiple cores to iterate over genes. Rank the resulting gene–peak pairs and evaluate them against the Fairfax monocyte interferon-γ/LPS cis-eQTL set as the positive reference, computing AUROC and AUPRC over the ranked predictions. A successful run reproduces reasonable prediction power (AUROC = 0.66, AUPRC = 0.68) and writes predictions to gene_peak_pair_predictions.csv.

## Related tools

- **SnapATAC** (R package providing predictGenePeakPair to fit the logistic-regression association between imputed gene expression and linked distal peak accessibility for each gene locus) — https://github.com/r3fang/SnapATAC
- **SnapTools** (Preprocessing module that creates the snap file (cell-by-peak PM, cell-by-gene GM matrices) consumed by SnapATAC) — https://github.com/r3fang/SnapTools
- **R 3.4.0** (Runtime for SnapATAC; versions >= 3.4.0 & < 3.6.0 required (3.6 does not work for rhdf5))
- **SnapATAC2** (Successor Python/Rust package implementing regulatory network analysis; linked from the SnapATAC README as the latest version (2.0)) — https://github.com/scverse/SnapATAC2

## Examples

```
predictGenePeakPair(obj, input.mat, gene.name, gene.loci, do.par=TRUE, num.cores=8)
```

## Evaluation signals

- Ranked gene–peak predictions reproduce AUROC = 0.66 and AUPRC = 0.68 when evaluated against the Fairfax monocyte IFN-γ/LPS cis-eQTL positive set.
- predictGenePeakPair completes per gene locus and returns gene–peak pairs whose accessibility matrix was binarized for distal peaks and restricted to the 3000 PBMC differentially expressed genes.
- Output file gene_peak_pair_predictions.csv exists and contains ranked gene–peak pairs.
- Input snap object is version-4 snap format containing the cell-by-peak (PM) and cell-by-gene (GM) sessions needed for imputation.

## Limitations

- Environment-dependent: SnapATAC requires Python (>= 2.7 & < 3.0) for SnapTools and R (>= 3.4.0 & < 3.6.0) because R 3.6 does not work for the rhdf5 package.
- Prediction power is moderate (AUROC = 0.66, AUPRC = 0.68), so enhancer–gene assignments are putative, not definitive regulatory wiring.
- The r3fang/SnapATAC R package is superseded by SnapATAC2 (kaizhang/SnapATAC2 / scverse/SnapATAC2), which uses a different API.

## Evidence

- [methods] SnapATAC links distal elements to putative target genes: "SnapATAC links distal elements to putative target genes"
- [methods] SnapATAC integrates scRNA and scATAC: "SnapATAC integrates scRNA and scATAC"
- [readme] SnapATAC links distal elements to putative target genes (10X_PBMC_15K example README link): "* [SnapATAC links distal elements to putative target genes](https://github.com/r3fang/SnapATAC/blob/master/examples/10X_PBMC_15K/README.md#gene_peak_pair)"
- [full_text] Enhancer–gene predictions match Fairfax monocyte cis-eQTLs with reasonable prediction power (AUROC 0.66, AUPRC 0.68): [evidence span withheld — anchor status: not_found]
