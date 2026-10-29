---
name: co-expressed-gene-pair-ranking
description: Use when you have a mean-centered, gene-by-experiment expression matrix
  (e.g., COnTORT_organized_transcriptomic_data.txt with 5 GenBank annotation columns
  plus per-GSM columns) and need to rank co-expressed or anti-correlated gene pairs
  to nominate transcription factor targets.
when_to_use_negative:
- Your expression matrix has not been mean-centered per experiment — COnTORT's non-mean-centered
  output is intended to preserve raw differences between experiments, so cross-experiment
  Pearson correlations on it would confound experiment-level offsets with co-expression.
- Your organism's expression data are not available in NCBI GEO as normalized and
  processed data — COnTORT can only download and combine normalized and processed
  data imported into NCBI GEO.
- You need raw expression retrieval or annotation rather than correlation ranking
  — use the upstream COnTORT retrieval/gffMatch steps instead of this downstream analysis.
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: COnTORT
  role: Produces the mean-centered, gene-annotated transcriptomic matrix (COnTORT_organized_transcriptomic_data.txt)
    that this correlation-ranking analysis consumes
  repo: https://github.com/GLBRC/contort
- name: pandas
  role: Python module used to load the tab-delimited matrix and compute the pairwise
    Pearson correlation coefficients between gene rows
- name: Python 3
  role: Runtime environment (with argparse, Bio, ftplib, functools, GEOparse, os,
    pandas, re, shutil, sys, subprocess, tkinter, time, urllib) for the analysis script
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1128/mra.00587-20
    title: 'COnTORT: COmprehensive Transcriptomic ORganizational Tool for Simultaneously
      Retrieving and Organizing Numerous Gene Expression Data Sets from the NCBI Gene
      Expression Omnibus Database'
  source_content_hash: sha256:e4fcdd39054e676eeb4986501d25e699237649e220e3827c2ed5f7d657aaa0b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GLBRC/contort
  tools:
  - COnTORT
  - Python 3
  - pandas
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# co-expressed-gene-pair-ranking

## Summary

Ranks all pairwise gene-gene expression correlations computed from the mean-centered COnTORT_organized_transcriptomic_data.txt output to identify candidate transcription factor targets in bacteria, as demonstrated by the COnTORT authors who correlated gene expression across a large number of diverse GEO data sets.

## When to use

Use when you have a mean-centered, gene-by-experiment expression matrix (e.g., COnTORT_organized_transcriptomic_data.txt with 5 GenBank annotation columns plus per-GSM columns) and need to rank co-expressed or anti-correlated gene pairs to nominate transcription factor targets.

## When NOT to use

- Your expression matrix has not been mean-centered per experiment — COnTORT's non-mean-centered output is intended to preserve raw differences between experiments, so cross-experiment Pearson correlations on it would confound experiment-level offsets with co-expression.
- Your organism's expression data are not available in NCBI GEO as normalized and processed data — COnTORT can only download and combine normalized and processed data imported into NCBI GEO.
- You need raw expression retrieval or annotation rather than correlation ranking — use the upstream COnTORT retrieval/gffMatch steps instead of this downstream analysis.

## Inputs

- COnTORT_organized_transcriptomic_data.txt — the mean-centered output file: tab-delimited text with the first five columns containing GenBank annotation (locus tag, old locus tag, gene name, gene synonyms, product) and remaining columns of per-experiment mean-centered expression values headed by GEO GSM IDs
- Optional: the non-mean-centered COnTORT output file (not recommended for cross-experiment correlation)

## Outputs

- gene_pair_correlations_ranked.tsv — a gene-pair table with annotation identifiers for both genes and the Pearson correlation value, sorted in descending rank order of correlation (optionally by absolute correlation)

## How to apply

Load COnTORT_organized_transcriptomic_data.txt with pandas, treating the first five columns (locus tag, old locus tag, gene name, gene synonyms, product) as gene row identifiers and the remaining columns — whose headers are the specific GEO IDs (GSM IDs) for each experiment — as the per-experiment mean-centered expression values. The mean-centered version is essential because COnTORT mean-centers data based on experiments to control for differences between experiments, which is what makes Pearson correlations comparable across the diverse joined GEO series. Compute all pairwise Pearson correlation coefficients between gene rows with pandas.DataFrame.T.corr(), exclude self-pairs, and assemble a gene-pair table carrying annotation identifiers for both genes plus the correlation value. Rank pairs by correlation coefficient in descending order (or by absolute correlation to also capture anti-correlated pairs) and write the result as a TSV named gene_pair_correlations_ranked.tsv; top-ranked co-expressed pairs are then inspected for new transcription factor targets.

## Related tools

- **COnTORT** (Produces the mean-centered, gene-annotated transcriptomic matrix (COnTORT_organized_transcriptomic_data.txt) that this correlation-ranking analysis consumes) — https://github.com/GLBRC/contort
- **pandas** (Python module used to load the tab-delimited matrix and compute the pairwise Pearson correlation coefficients between gene rows)
- **Python 3** (Runtime environment (with argparse, Bio, ftplib, functools, GEOparse, os, pandas, re, shutil, sys, subprocess, tkinter, time, urllib) for the analysis script)

## Examples

```
import pandas as pd; df = pd.read_csv('COnTORT_organized_transcriptomic_data.txt', sep='\t', index_col=list(range(5))); corr = df.T.corr(); pairs = corr.stack().reset_index(); pairs = pairs[pairs.iloc[:,0] != pairs.iloc[:,1]].sort_values(0, ascending=False); pairs.to_csv('gene_pair_correlations_ranked.tsv', sep='\t', index=False)
```

## Evaluation signals

- All correlation values in gene_pair_correlations_ranked.tsv lie within [-1, 1].
- The table is sorted in descending rank order of correlation (monotonically non-increasing down the ranked column).
- No gene is paired with itself (all self-pairs excluded); every row has two distinct gene identifiers plus one correlation value.
- Row identifiers and column headers of the correlation input match the README schema: first five annotation columns (locus tag, old locus tag, gene name, gene synonyms, product) and GSM-ID experiment headers; annotations absent from the GenBank file appear as 'N/A'.
- The number of ranked pairs equals N*(N-1) for N genes (or N*(N-1)/2 for the unique upper triangle), confirming no pair duplication.

## Limitations

- COnTORT can only download and combine normalized and processed data imported into NCBI GEO, so raw or unprocessed series cannot feed this correlation analysis.
- Correlations are only as good as the joined GEO matrix: annotations not present in the user-provided GenBank file are listed as 'N/A' and blank/missing values are retained for consistency, which must be handled before correlation.
- Mean-centering is per experiment to control for differences between experiments; using the non-mean-centered output defeats this control and can distort cross-data-set correlations.

## Evidence

- [abstract] We have used COnTORT files to correlate expression of genes across a large number of diverse data sets to identify new targets of transcription factors in bacteria.: "We have used COnTORT files to correlate expression of genes across a large number of diverse data sets to identify new targets of transcription factors in bacteria."
- [abstract] One output file will mean-center the data based on experiments to control for differences between experiments, while the second output file will not mean-center the data.: "One output file will mean-center the data based on experiments to control for differences between experiments, while the second output file will not mean-center the data."
- [abstract] The first five columns contain the annotation information from the GenBank file (locus tag, old locus tag, gene name, gene synonyms, and product): "The first five columns contain the annotation information from the GenBank file (locus tag, old locus tag, gene name, gene synonyms, and product)"
- [readme] Primary output is a single text file containing the gene annotations on the first five columns and the mean centered expression data present in the remaining columns.: "Primary output is a single text file containing the gene annotations on the first five columns
    and the mean centered expression data present in the remaining columns."
- [readme] COnTORT can only download and combine normalized and processed data imported into NCBI GEO.: "Note that COnTORT can only download and combine normalized and processed data imported into NCBI GEO."
