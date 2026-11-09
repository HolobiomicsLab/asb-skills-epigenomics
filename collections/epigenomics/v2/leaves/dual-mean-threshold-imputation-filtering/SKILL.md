---
name: dual-mean-threshold-imputation-filtering
description: Use when you have a SCALE-imputed scATAC-seq peak-by-barcode matrix and
  need a close-to-binary accessibility matrix; scATAC-seq open-chromatin data is inherently
  close-to-binary and binarization removes replicate-specific imputation artifacts.
when_to_use_negative:
- The input is raw scRNA-seq expression data — the article notes direct application
  of scRNA-seq methods is unsuitable for scATAC-seq precisely because of its close-to-binary
  nature, and this rule is calibrated for scATAC-seq peak data
- The input matrix is already binary — re-thresholding would destroy the values
- You need continuous imputed values for downstream quantitative analysis rather than
  binary accessibility calls
edam_topics:
- http://edamontology.org/topic_3673
tools:
- name: SCALE
  role: Produces the imputed matrix and provides the --binary option that yields binary
    imputed data in adata.obsm['binary']; the dual-mean rule reproduces this behavior
  repo: https://github.com/jsxlei/SCALE
- name: scanpy
  role: Supplies highly_variable_genes for upstream peak filtering and stores the
    binary matrix as adata.obsm['binary'] in the output AnnData
- name: numpy
  role: Computes per-row and per-column means of the raw matrix and saves the binary
    matrix via numpy.save
- name: pandas
  role: Loads the tab-separated peak-by-barcode count matrix used for the mean thresholds
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-12630-7
    title: SCALE method for single-cell ATAC-seq analysis via latent feature extraction
  source_content_hash: sha256:9cc5d52ed05c935f56b9e6956a1e972a43d0a16e4ea0bdb10ef1323a504dba05
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jsxlei/SCALE
  tools:
  - SCALE
  - numpy
  - pandas
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# dual-mean-threshold-imputation-filtering

## Summary

Binarizes a SCALE-imputed scATAC-seq accessibility matrix (peaks x barcodes) by keeping an entry as 1 only where the imputed value exceeds both the row (peak) mean of the raw matrix and the column (cell) mean of the raw matrix, reproducing SCALE's --binary behavior.

## When to use

Use when you have a SCALE-imputed scATAC-seq peak-by-barcode matrix and need a close-to-binary accessibility matrix; scATAC-seq open-chromatin data is inherently close-to-binary and binarization removes replicate-specific imputation artifacts.

## When NOT to use

- The input is raw scRNA-seq expression data — the article notes direct application of scRNA-seq methods is unsuitable for scATAC-seq precisely because of its close-to-binary nature, and this rule is calibrated for scATAC-seq peak data
- The input matrix is already binary — re-thresholding would destroy the values
- You need continuous imputed values for downstream quantitative analysis rather than binary accessibility calls

## Inputs

- SCALE-imputed accessibility matrix (peaks x barcodes)
- Raw scATAC-seq count matrix used to compute row/column means (txt/tsv, sep="\t", rows=peaks, columns=barcodes; or h5ad/mtx from SCALE output)

## Outputs

- Binary accessibility matrix with every entry in {0, 1} (e.g. binary_matrix.npy or adata.obsm['binary'])

## How to apply

Load the count matrix (rows = peaks, columns = barcodes) from txt/tsv (sep="\t") or csv with pandas, matching SCALE's supported input formats. Compute per-row means over the raw matrix raw[i,:] and per-column means raw[:,j] with numpy. Apply the dual-mean rule: set imputed[i,j] = 1 only where imputed[i,j] > mean(raw[i,:]) AND imputed[i,j] > mean(raw[:,j]); otherwise set it to 0. Save the result (e.g. binary_matrix.npy via numpy.save, or as scanpy adata.obsm['binary'] when using SCALE's --binary option). The rationale is that scATAC-seq accessibility is close-to-binary due to limited capture per open chromatin site, so thresholding the imputed values against both peak-level and cell-level raw means recovers a binary state and, as the source finding reports, differences between replicates disappeared after this binarization.

## Related tools

- **SCALE** (Produces the imputed matrix and provides the --binary option that yields binary imputed data in adata.obsm['binary']; the dual-mean rule reproduces this behavior) — https://github.com/jsxlei/SCALE
- **scanpy** (Supplies highly_variable_genes for upstream peak filtering and stores the binary matrix as adata.obsm['binary'] in the output AnnData)
- **numpy** (Computes per-row and per-column means of the raw matrix and saves the binary matrix via numpy.save)
- **pandas** (Loads the tab-separated peak-by-barcode count matrix used for the mean thresholds)

## Examples

```
SCALE.py -d [input] --binary; binary = ((imputed > raw.mean(axis=1, keepdims=True)) & (imputed > raw.mean(axis=0, keepdims=True))).astype(int)
```

## Evaluation signals

- Assert every entry of the output matrix is exactly in {0, 1}
- Recompute the dual-mean rule on the test input and verify each entry matches: imputed[i,j]==1 iff imputed[i,j] > mean(raw[i,:]) and imputed[i,j] > mean(raw[:,j])
- Confirm replicate differences present in the imputed data disappear after binarization (the reported finding)
- When using SCALE --binary, check the result equals adata.obsm['binary'] in the output adata.h5ad
- Verify the loaded raw matrix has rows = peaks and columns = barcodes before computing means (orientation errors flip the rule)

## Limitations

- SCALE is not specifically designed to identify and remove batch effects or other artifacts from the input data, so binarization is not a substitute for batch correction
- Explicit modeling of batch effects and technical variations is deferred to future work in SCALE and not implemented
- The rule depends on correct matrix orientation (peaks as rows, barcodes as columns per SCALE's txt/tsv input convention); transposed input silently produces wrong thresholds

## Evidence

- [intro] Direct applications of scRNA-seq analysis methods to scATAC-seq data may not be suitable due to the close-to-binary nature and increased sparsity of the data: "Direct applications of these scRNA-seq analysis methods to scATAC-seq data, however, may not be suitable due to the close-to-binary nature and increased sparsity"
- [readme] SCALE supports binarized imputation output via --binary: "Get binary imputed data in adata.h5ad file using scanpy **adata.obsm['binary']** with option **--binary** (recommended for saving storage)"
- [readme] SCALE count matrix input format for computing means: "row is peak and column is barcode, in **txt** / **tsv** (sep=**"\t"**) or **csv** (sep=**","**) format"
- [full_text] Binarization makes replicate differences disappear: [evidence span withheld — anchor status: not_found]
- [discussion] SCALE covers denoising and imputation as core outputs: "SCALE is thus a powerful tool for scATAC-seq data analysis, including data visualization, clustering, and denoising and imputation."
