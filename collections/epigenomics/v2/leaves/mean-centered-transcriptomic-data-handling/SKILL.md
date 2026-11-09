---
name: mean-centered-transcriptomic-data-handling
description: Use when you have a COnTORT_organized_transcriptomic_data.txt matrix
  (5 annotation columns + mean-centered experiment columns) and need to correlate
  gene expression across diverse GEO data sets to rank co-expressed gene pairs.
when_to_use_negative:
- The data have not been mean-centered per experiment (use the non-mean-centered COnTORT
  output only when between-experiment differences are not a concern)
- COnTORT can only download and combine normalized and processed data imported into
  NCBI GEO — raw or unnormalized data are outside this workflow
- The goal is retrieving/organizing GEO data itself rather than analyzing an already-produced
  COnTORT output file (run contort first instead)
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: COnTORT
  role: Produces the mean-centered organized transcriptomic matrix (COnTORT_organized_transcriptomic_data.txt)
    that serves as the input for downstream correlation analysis
  repo: https://github.com/GLBRC/contort
- name: pandas
  role: Python module used to load the tab-delimited COnTORT matrix and compute pairwise
    Pearson correlations between gene rows
- name: Python 3
  role: Runtime environment for the correlation analysis of the COnTORT output
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1128/mra.00587-20
    title: 'COnTORT: COmprehensive Transcriptomic ORganizational Tool for Simultaneously
      Retrieving and Organizing Numerous Gene Expression Data Sets from the NCBI Gene
      Expression Omnibus Database'
  source_content_hash: sha256:bebb48cc9c69e700de3068bc5a0183a21e9f11799d77ccf58da5a3a66038949e
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
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# mean-centered-transcriptomic-data-handling

## Summary

Downstream analysis of the COnTORT mean-centered transcriptomic matrix (COnTORT_organized_transcriptomic_data.txt): treat the first five GenBank annotation columns as gene row identifiers and compute pairwise Pearson correlations across all per-experiment mean-centered expression columns to rank co-expressed gene pairs and identify candidate transcription factor targets.

## When to use

Use when you have a COnTORT_organized_transcriptomic_data.txt matrix (5 annotation columns + mean-centered experiment columns) and need to correlate gene expression across diverse GEO data sets to rank co-expressed gene pairs.

## When NOT to use

- The data have not been mean-centered per experiment (use the non-mean-centered COnTORT output only when between-experiment differences are not a concern)
- COnTORT can only download and combine normalized and processed data imported into NCBI GEO — raw or unnormalized data are outside this workflow
- The goal is retrieving/organizing GEO data itself rather than analyzing an already-produced COnTORT output file (run contort first instead)

## Inputs

- COnTORT_organized_transcriptomic_data.txt: tab-delimited file with first five annotation columns (locus tag, old locus tag, gene name, gene synonyms, product) and remaining columns of per-experiment mean-centered expression values keyed by GEO GSM IDs
- GenBank (.gbff) gene annotation file for the organism (used upstream by COnTORT to produce the annotations)

## Outputs

- gene_pair_correlations_ranked.tsv: gene-pair table with annotation identifiers for both genes and Pearson correlation values, ranked in descending order

## How to apply

Load COnTORT_organized_transcriptomic_data.txt with pandas, using the first five columns (locus tag, old locus tag, gene name, gene synonyms, product) as gene row identifiers and the remaining GSM_ID columns as the per-experiment mean-centered expression values. Compute pairwise Pearson correlation coefficients between all gene rows across the experiment columns. Assemble a gene-pair table with annotation identifiers for both genes plus the correlation value, excluding self-pairs, then rank pairs in descending order (or by absolute correlation to capture anti-correlated pairs) and write as gene_pair_correlations_ranked.tsv. The mean-centering per experiment is the rationale: it controls for differences between experiments so correlations across many diverse data sets are meaningful.

## Related tools

- **COnTORT** (Produces the mean-centered organized transcriptomic matrix (COnTORT_organized_transcriptomic_data.txt) that serves as the input for downstream correlation analysis) — https://github.com/GLBRC/contort
- **pandas** (Python module used to load the tab-delimited COnTORT matrix and compute pairwise Pearson correlations between gene rows)
- **Python 3** (Runtime environment for the correlation analysis of the COnTORT output)

## Examples

```
import pandas as pd; df = pd.read_csv('COnTORT_organized_transcriptomic_data.txt', sep='\t'); corr = df.iloc[:, 5:].T.corr(); corr.to_csv('gene_pair_correlations_ranked.tsv', sep='\t')
```

## Evaluation signals

- All correlation values in gene_pair_correlations_ranked.tsv lie within [-1, 1]
- The output table is sorted in descending rank order of correlation coefficient
- No self-pairs (gene correlated with itself, r = 1) are present in the output
- Each row carries the COnTORT annotation identifiers (locus tag, old locus tag, gene name, synonyms, product) for both genes of the pair
- Number of experiment columns consumed equals the number of GSM_ID columns present in COnTORT_organized_transcriptomic_data.txt

## Limitations

- COnTORT can only download and combine normalized and processed data imported into NCBI GEO; it cannot use raw data
- Genes whose annotations are not present in the user-provided GenBank file are listed with 'N/A', limiting interpretability of pairs involving such rows
- Only one organism's GenBank annotation is used per run, so the output is organism-specific

## Evidence

- [abstract] One output file will mean-center the data based on experiments to control for differences between experiments: "One output file will mean-center the data based on experiments to control for differences between experiments"
- [abstract] We have used COnTORT files to correlate expression of genes across a large number of diverse data sets to identify new targets of transcription factors in bacteria.: "We have used COnTORT files to correlate expression of genes across a large number of diverse data sets to identify new targets of transcription factors in bacteria."
- [abstract] The first five columns contain the annotation information from the GenBank file (locus tag, old locus tag, gene name, gene synonyms, and product): "The first five columns contain the annotation information from the GenBank file (locus tag, old locus tag, gene name, gene synonyms, and product)"
- [intro] COnTORT can only download and combine normalized and processed data imported into NCBI GEO.: "COnTORT can only download and combine normalized and processed data imported into NCBI GEO."
- [readme] Primary output is a single text file containing the gene annotations on the first five columns and the mean centered expression data present in the remaining columns.: "Primary output is a single text file containing the gene annotations on the first five columns
    and the mean centered expression data present in the remaining columns."
