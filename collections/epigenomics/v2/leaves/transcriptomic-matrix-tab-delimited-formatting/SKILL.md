---
name: transcriptomic-matrix-tab-delimited-formatting
description: Use when building a cross-experiment gene expression compendium where
  per-experiment (per-GSM) offsets must be removed before downstream correlation or
  statistical analysis, following COnTORT's mean-centering-per-experiment output format.
when_to_use_negative:
- The analysis requires the non-mean-centered output file (COnTORT also produces a
  second file with no mean centering) — do not apply centering when raw organized
  expression values are the target.
- Input data are raw/unprocessed reads rather than normalized and processed data imported
  into NCBI GEO; COnTORT can only download and combine normalized and processed GEO
  data.
- Only a single experiment/sample is present — per-experiment mean-centering would
  reduce every column to zero and provide no cross-experiment normalization benefit.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: COnTORT
  role: Pipeline that organizes GEO expression data, mean-centers per experiment,
    and joins results into the single mean-centered text file
  repo: https://github.com/GLBRC/contort
- name: Python 3
  role: Runtime for reimplementing the mean-centering step on the organized expression
    matrix
  repo: https://github.com/GLBRC/contort
- name: pandas
  role: Python module used by COnTORT (among argparse, Bio, ftplib, functools, GEOparse,
    os, re, shutil, sys, subprocess, tkinter, time, urllib) for loading, transforming,
    and writing the expression matrices
  repo: https://github.com/GLBRC/contort
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1128/mra.00587-20
    title: 'COnTORT: COmprehensive Transcriptomic ORganizational Tool for Simultaneously
      Retrieving and Organizing Numerous Gene Expression Data Sets from the NCBI Gene
      Expression Omnibus Database'
  source_content_hash: sha256:44a16fe09871394eb5d63e24e8483eab558c886b5db33ccd481c064c6eaea27b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GLBRC/contort
  tools:
  - COnTORT
  - Python 3
  - pandas
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# transcriptomic-matrix-tab-delimited-formatting

## Summary

Reconstruct COnTORT's mean-centered output file: organize GEO expression data annotated from a GenBank (.gbff) file, subtract each experiment's (GSM column's) mean from every value, and write a tab-delimited text file with the five annotation columns intact so experiments are comparable across a compendium.

## When to use

Use when building a cross-experiment gene expression compendium where per-experiment (per-GSM) offsets must be removed before downstream correlation or statistical analysis, following COnTORT's mean-centering-per-experiment output format.

## When NOT to use

- The analysis requires the non-mean-centered output file (COnTORT also produces a second file with no mean centering) — do not apply centering when raw organized expression values are the target.
- Input data are raw/unprocessed reads rather than normalized and processed data imported into NCBI GEO; COnTORT can only download and combine normalized and processed GEO data.
- Only a single experiment/sample is present — per-experiment mean-centering would reduce every column to zero and provide no cross-experiment normalization benefit.

## Inputs

- Organized non-mean-centered expression matrix (tab-delimited) with five GenBank annotation columns (Locus_Tag, Old_Locus_Tag, Gene_Name, Gene_Synonyms, Product) plus expression columns keyed by GSM IDs
- GenBank (.gbff) gene annotation file for the organism, used to produce the annotation columns via gffMatch

## Outputs

- Mean-centered tab-delimited text file (e.g., COnTORT_organized_transcriptomic_data.txt variant in mean_centered_results) with five annotation columns, per-experiment mean-centered expression columns headed by GEO/GSM IDs, and 'N/A' placeholders for missing annotations

## How to apply

Load the organized non-mean-centered expression matrix (first five columns Locus_Tag, Old_Locus_Tag, Gene_Name, Gene_Synonyms, Product; remaining columns keyed by GSM IDs) with pandas in Python 3. Compute the mean of each expression column (each experiment) and subtract it from every value in that column, leaving the five GenBank-derived annotation columns and the matrix dimensions unchanged. Retain all blanks ('N/A' annotations) for consistency, as COnTORT does when joining via gene annotation keys. Write the result as a tab-delimited text file (the mean_centered_results content) alongside the non-mean-centered file, with a header row referencing the specific GEO IDs (GSM_ID_1, GSM_ID_2, ...). Mean-centering is the mechanism COnTORT uses to control for differences between experiments before the data are joined into a single file usable in Excel or R.

## Related tools

- **COnTORT** (Pipeline that organizes GEO expression data, mean-centers per experiment, and joins results into the single mean-centered text file) — https://github.com/GLBRC/contort
- **Python 3** (Runtime for reimplementing the mean-centering step on the organized expression matrix) — https://github.com/GLBRC/contort
- **pandas** (Python module used by COnTORT (among argparse, Bio, ftplib, functools, GEOparse, os, re, shutil, sys, subprocess, tkinter, time, urllib) for loading, transforming, and writing the expression matrices) — https://github.com/GLBRC/contort

## Examples

```
import pandas as pd; df = pd.read_csv('COnTORT_organized_transcriptomic_data.txt', sep='\t'); expr = df.columns[5:]; df[expr] = df[expr] - df[expr].mean(); df.to_csv('mean_centered_results.txt', sep='\t', index=False)
```

## Evaluation signals

- Each mean-centered expression column has a mean within a small tolerance (e.g., ~1e-10) of zero.
- Output dimensions match the input matrix: same number of rows (genes) and columns (5 annotation columns + expression columns).
- The five annotation columns (Locus_Tag, Old_Locus_Tag, Gene_Name, Gene_Synonyms, Product) are byte-identical to the input, with all 'N/A' blanks retained for consistency.
- Header row references the specific GEO IDs (GSM IDs) for each experiment, matching the source matrix column keys.
- File is tab-delimited and opens cleanly in Excel or R, consistent with COnTORT's stated output format.

## Limitations

- COnTORT can only download and combine normalized and processed data imported into NCBI GEO; raw or unprocessed data cannot be used for this centering step.
- Annotations not present in the provided GenBank file are listed as 'N/A', so genes absent from the .gbff will not be fully annotated in the output.
- Mean-centering is per experiment (per GSM column) only; it does not perform more sophisticated batch correction or cross-platform normalization.

## Evidence

- [abstract] One output file will mean-center the data based on experiments to control for differences between experiments, while the second output file will not mean-center the data.: "One output file will mean-center the data based on experiments to control for differences between experiments, while the second output file will not mean-center the data."
- [intro] Mean-center and join via gene annotation into a single text file: "The data will be organized, mean centered, and then joined using the gene annotation into a single text file"
- [readme] gffMatch mean-centers and joins data retaining all blanks: "The data are then mean centered and joined together with the gene annotations annotation as the key. All blanks are retained for consistency."
- [readme] Primary output format: annotations in first five columns, mean-centered data in remaining columns, GEO ID header row: "Primary output is a single text file containing the gene annotations on the first five columns and the mean centered expression data present in the remaining columns."
