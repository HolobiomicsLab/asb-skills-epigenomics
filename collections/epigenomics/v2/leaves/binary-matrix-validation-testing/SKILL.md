---
name: binary-matrix-validation-testing
description: Use when you have a SCALE-imputed scATAC-seq accessibility matrix (peaks
  as rows, barcodes as columns) and need a validated binary matrix matching SCALE's
  --binarization behavior for downstream clustering or storage.
when_to_use_negative:
- Input is already a binary accessibility matrix — recomputing dual-mean thresholds
  would corrupt it
- Input is an h5ad adata object from a SCALE run — use the built-in --binary option
  instead of reimplementing the rule
- Data is scRNA-seq expression, where the close-to-binary scATAC-seq assumption does
  not hold
tools:
- name: SCALE
  role: Produces the imputed scATAC-seq matrix that is binarized; its --binary option
    defines the reference binarization behavior (adata.obsm['binary'])
  repo: https://github.com/jsxlei/SCALE
- name: scanpy
  role: Used to read the binary imputed results from adata.obsm['binary'] in the saved
    adata.h5ad file
  repo: https://scanpy.readthedocs.io/en/stable/index.html
- name: numpy/pandas
  role: Load the tab-separated count matrix, compute row/column mean thresholds, and
    save binary_matrix.npy
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-12630-7
    title: SCALE method for single-cell ATAC-seq analysis via latent feature extraction
  source_content_hash: sha256:0cc70afd62cceb66516c4e4d3eb9774d81c87c537386d843d90f24880ce3a175
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jsxlei/SCALE
  tools:
  - SCALE
  - numpy/pandas
  - scanpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# binary-matrix-validation-testing

## Summary

Converts a SCALE-imputed scATAC-seq peak-by-barcode matrix into a binary accessibility matrix via SCALE's dual-mean threshold rule and validates that every saved entry is exactly 0 or 1. This exploits the close-to-binary nature of scATAC-seq peak data to reproduce the SCALE --binary output.

## When to use

Use when you have a SCALE-imputed scATAC-seq accessibility matrix (peaks as rows, barcodes as columns) and need a validated binary matrix matching SCALE's --binarization behavior for downstream clustering or storage.

## When NOT to use

- Input is already a binary accessibility matrix — recomputing dual-mean thresholds would corrupt it
- Input is an h5ad adata object from a SCALE run — use the built-in --binary option instead of reimplementing the rule
- Data is scRNA-seq expression, where the close-to-binary scATAC-seq assumption does not hold

## Inputs

- SCALE-imputed scATAC-seq matrix (peaks x barcodes) as numpy array or adata.h5ad
- Raw scATAC-seq count matrix in txt/tsv format (tab-separated, row = peak, column = barcode) for computing row/column mean thresholds

## Outputs

- binary_matrix.npy: numpy array with entries strictly in {0, 1}
- adata.obsm['binary']: binary imputed data stored in adata.h5ad when using SCALE's --binary option

## How to apply

Load the raw test count matrix (rows = peaks, columns = barcodes) from a tab-separated txt/tsv file with pandas, then compute per-row means (mean of raw[i,:]) and per-column means (mean of raw[:,j]) with numpy. Apply SCALE's binarization rule: set imputed[i,j] = 1 only where imputed[i,j] exceeds BOTH the row mean and the column mean, otherwise set it to 0; as a supplementary rule, values greater than 1 can be masked to 1. Save the result as binary_matrix.npy with numpy.save. Validate by asserting every entry of the saved matrix is in {0, 1} and that each entry exactly matches the dual-mean threshold rule on the test input. On real SCALE runs, the equivalent output is obtained via the --binary flag, which stores the binary imputed data in adata.obsm['binary'] of adata.h5ad.

## Related tools

- **SCALE** (Produces the imputed scATAC-seq matrix that is binarized; its --binary option defines the reference binarization behavior (adata.obsm['binary'])) — https://github.com/jsxlei/SCALE
- **scanpy** (Used to read the binary imputed results from adata.obsm['binary'] in the saved adata.h5ad file) — https://scanpy.readthedocs.io/en/stable/index.html
- **numpy/pandas** (Load the tab-separated count matrix, compute row/column mean thresholds, and save binary_matrix.npy)

## Examples

```
import numpy as np, pandas as pd; raw = pd.read_csv('counts.tsv', sep='\t').values; rm = raw.mean(axis=1, keepdims=True); cm = raw.mean(axis=0, keepdims=True); binary = ((imputed > rm) & (imputed > cm)).astype(int); np.save('binary_matrix.npy', binary)
```

## Evaluation signals

- Assert every entry of the saved binary matrix is exactly 0 or 1 (no fractional imputed values)
- Recompute the dual-mean rule (entry > mean of raw row i AND entry > mean of raw column j) and diff against the saved matrix — differences must be zero
- Compare against SCALE's own --binary output stored in adata.obsm['binary'] for the same input
- Confirm the output file binary_matrix.npy loads via numpy.load with the same shape as the input peak x barcode matrix
- If replicates are available, check that replicate differences disappear after binarization, as reported for SCALE's binarization step

## Limitations

- Binarization discards quantitative imputed values, so it is intended for storage savings and replicate stability, not for preserving imputation confidence
- Direct applications of scRNA-seq analysis methods to scATAC-seq data may not be suitable due to the close-to-binary nature and increased sparsity of the data, so thresholds calibrated for RNA counts do not transfer
- SCALE is not specifically designed to identify and remove batch effects, so a binary matrix built from artifact-laden imputed data will retain those artifacts

## Evidence

- [readme] Binarization via dual-mean threshold: "Get binary imputed data in adata.h5ad file using scanpy **adata.obsm['binary']** with option **--binary** (recommended for saving storage)"
- [intro] Rationale: close-to-binary scATAC-seq data: "Direct applications of these scRNA-seq analysis methods to scATAC-seq data, however, may not be suitable due to the close-to-binary nature and increased sparsity of the data"
- [readme] Input format for count matrices: "row is peak and column is barcode, in **txt** / **tsv** (sep=**"\t"**) or **csv** (sep=**","**) format"
- [other] Binarization eliminates replicate differences: [evidence span withheld — anchor status: not_found]
