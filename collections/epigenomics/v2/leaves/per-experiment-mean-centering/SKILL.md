---
name: per-experiment-mean-centering
description: Use when combining gene expression values from multiple GEO series into
  a single matrix and you need to remove per-experiment offset differences, e.g.,
  building COnTORT_organized_transcriptomic_data.txt (mean-centered variant).
when_to_use_negative:
- When you need the raw joined values — COnTORT also produces a second output file
  with no mean centering; use that instead
- When the input is not normalized/processed GEO expression data imported into NCBI
  GEO — COnTORT can only download and combine normalized and processed data
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: COnTORT
  role: Pipeline whose gffMatch/mean-centering step performs per-experiment centering
    and joins all series on the gene annotation into the mean-centered output file
  repo: https://github.com/GLBRC/contort
- name: pandas
  role: Loads the organized matrix, computes per-column (per-experiment) means, and
    writes the tab-delimited mean-centered table
- name: Python 3
  role: Runtime for the mean-centering script (argparse, pandas)
provenance:
  source_task_ids:
  - task_004
  - task_001
  - task_006
  - task_008
  source_papers:
  - doi: 10.1128/mra.00587-20
    title: 'COnTORT: COmprehensive Transcriptomic ORganizational Tool for Simultaneously
      Retrieving and Organizing Numerous Gene Expression Data Sets from the NCBI Gene
      Expression Omnibus Database'
  source_content_hash: sha256:ac9c30d63f52742c5dfc5ef0521a27b9f7bb0df2928f8c57217c780ca7d4ae1a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GLBRC/contort
  tools:
  - Bio
  - COnTORT
  - Python 3
  - argparse
  - ftplib
  - pandas
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# per-experiment-mean-centering

## Summary

Per-experiment mean centering of COnTORT-organized GEO transcriptomic data: each expression column (GSM experiment) is column-mean subtracted to control for inter-experiment differences, producing COnTORT's mean-centered output file alongside the non-mean-centered file.

## When to use

Use when combining gene expression values from multiple GEO series into a single matrix and you need to remove per-experiment offset differences, e.g., building COnTORT_organized_transcriptomic_data.txt (mean-centered variant).

## When NOT to use

- When you need the raw joined values — COnTORT also produces a second output file with no mean centering; use that instead
- When the input is not normalized/processed GEO expression data imported into NCBI GEO — COnTORT can only download and combine normalized and processed data

## Inputs

- Non-mean-centered organized expression matrix (tab-delimited; five GenBank annotation columns Locus_Tag, Old_Locus_Tag, Gene_Name, Gene_Synonyms, Product followed by log2 normalized expression columns keyed by GSM IDs)
- pandas DataFrame loaded from COnTORT_organized_transcriptomic_data.txt (non-mean-centered variant)

## Outputs

- Mean-centered tab-delimited text file (e.g., COnTORT_organized_transcriptomic_data.txt mean-centered counterpart) with identical rows, columns, and annotation fields; each expression column mean ≈ 0

## How to apply

Load the gffMatch-organized non-mean-centered matrix with pandas in Python 3; its first five columns are the GenBank annotations (Locus_Tag, Old_Locus_Tag, Gene_Name, Gene_Synonyms, Product) keyed by 'N/A' for absent annotations, and remaining columns are log2 normalized expression values keyed by GSM IDs. Compute the mean of each expression column (one mean per experiment/GSM ID) and subtract it from every value in that column, leaving the five annotation columns and the row/column dimensions unchanged. Write the result as a tab-delimited text file, retaining all blanks ('N/A' annotations) for consistency. Rationale: COnTORT mean-centers based on experiments to control for differences between experiments, and joins all series on the gene annotation into one file usable in Excel or R. Note COnTORT writes both a mean-centered and a non-mean-centered output; this skill applies to the mean-centered one.

## Related tools

- **COnTORT** (Pipeline whose gffMatch/mean-centering step performs per-experiment centering and joins all series on the gene annotation into the mean-centered output file) — https://github.com/GLBRC/contort
- **pandas** (Loads the organized matrix, computes per-column (per-experiment) means, and writes the tab-delimited mean-centered table)
- **Python 3** (Runtime for the mean-centering script (argparse, pandas))

## Examples

```
import pandas as pd; df = pd.read_csv('organized.txt', sep='\t'); df[df.columns[5:]] = df[df.columns[5:]] - df[df.columns[5:]].mean(); df.to_csv('mean_centered.txt', sep='\t', index=False)
```

## Evaluation signals

- The mean of each expression column in the output is within a small tolerance of zero
- Output dimensions (row and column count) exactly match the input organized matrix
- The five annotation columns (Locus_Tag, Old_Locus_Tag, Gene_Name, Gene_Synonyms, Product) are present and unchanged, with 'N/A' retained for annotations absent from the GenBank file
- Output is a tab-delimited file; first row headers reference the specific GEO IDs (GSM accession) for each experiment

## Limitations

- Centering controls only inter-experiment offset differences (column means); it is not full normalization of the underlying data
- All blanks ('N/A' annotations) are retained for consistency, so downstream users must handle N/A annotation rows
- Applies only to normalized and processed data imported into NCBI GEO; raw data are out of scope

## Evidence

- [abstract] One output file will mean-center the data based on experiments to control for differences between experiments, while the second output file will not mean-center the data.: "One output file will mean-center the data based on experiments to control for differences between experiments, while the second output file will not mean-center the data."
- [other] Compute the mean of each expression column (experiment) and subtract it from every value in that column, leaving the five annotation columns and the row/column count unchanged.: [evidence span withheld — anchor status: not_found]
- [readme] The data are then mean centered and joined together with the gene annotations annotation as the key. All blanks are retained for consistency.: "The data are then mean centered and joined together with the gene annotations annotation as the key. All blanks are retained for consistency."
- [intro] joined using the gene annotation into a single text file that can be easily manipulated or opened in Excel: "joined using the gene annotation into a single text file that can be easily manipulated or opened in Excel."
