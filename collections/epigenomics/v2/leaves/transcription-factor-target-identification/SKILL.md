---
name: transcription-factor-target-identification
description: Use when the goal is to identify candidate transcription factor targets
  by correlating gene expression across many diverse GEO experiments, given a mean-centered
  COnTORT output file with GenBank annotation columns.
when_to_use_negative:
- The expression matrix has not been mean-centered per experiment (use the non-mean-centered
  COnTORT output for other purposes; raw cross-experiment correlation would be confounded
  by experiment-level differences)
- The data are not normalized, processed GEO series data — COnTORT can only download
  and combine normalized and processed data imported into NCBI GEO
- Genes of interest lack matches in the provided GenBank file, since COnTORT retains
  only matches to gene annotations present in the GenBank file (missing annotations
  are listed as 'N/A')
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: COnTORT
  role: Produces the mean-centered, GenBank-annotated transcriptomic matrix that is
    the input to the correlation analysis
  repo: https://github.com/GLBRC/contort
- name: pandas
  role: Loads the tab-delimited COnTORT matrix and computes pairwise Pearson correlations
    between gene rows across experiment columns
- name: Python 3
  role: Runtime for the correlation and ranking workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1128/mra.00587-20
    title: 'COnTORT: COmprehensive Transcriptomic ORganizational Tool for Simultaneously
      Retrieving and Organizing Numerous Gene Expression Data Sets from the NCBI Gene
      Expression Omnibus Database'
  source_content_hash: sha256:96de12504504934bf7f27be495402abc5fe7df6c75837f055f103ae606e8189f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GLBRC/contort
  tools:
  - COnTORT
  - Python 3
  - pandas
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# transcription-factor-target-identification

## Summary

Rank co-expressed gene pairs by computing pairwise Pearson correlations across the experimentally mean-centered COnTORT transcriptomic matrix (COnTORT_organized_transcriptomic_data.txt) to surface candidate targets of bacterial transcription factors.

## When to use

Use when the goal is to identify candidate transcription factor targets by correlating gene expression across many diverse GEO experiments, given a mean-centered COnTORT output file with GenBank annotation columns.

## When NOT to use

- The expression matrix has not been mean-centered per experiment (use the non-mean-centered COnTORT output for other purposes; raw cross-experiment correlation would be confounded by experiment-level differences)
- The data are not normalized, processed GEO series data — COnTORT can only download and combine normalized and processed data imported into NCBI GEO
- Genes of interest lack matches in the provided GenBank file, since COnTORT retains only matches to gene annotations present in the GenBank file (missing annotations are listed as 'N/A')

## Inputs

- COnTORT_organized_transcriptomic_data.txt (mean-centered, tab-delimited; first five columns: locus tag, old locus tag, gene name, gene synonyms, product; remaining columns: mean-centered expression values per GEO experiment, header row of GEO IDs)
- GenBank (.gbff) annotation file for the organism of choice from NCBI (used by COnTORT upstream to annotate rows)

## Outputs

- gene_pair_correlations_ranked.tsv — gene-pair table with annotation identifiers for both genes and the Pearson correlation value, ranked in descending order of correlation

## How to apply

Load COnTORT_organized_transcriptomic_data.txt into pandas as a tab-delimited table, treating the first five columns (locus tag, old locus tag, gene name, gene synonyms, product from the GenBank .gbff file) as gene row identifiers and the remaining columns as per-experiment mean-centered expression values keyed by GSM IDs. Compute pairwise Pearson correlation coefficients between all gene rows across the experiment columns, which is valid because COnTORT mean-centered the data per experiment to control for between-experiment differences. Assemble a gene-pair table (annotations for both genes plus the correlation value), exclude self-pairs, and rank pairs by correlation in descending order, optionally by absolute correlation to capture anti-correlated pairs; write as gene_pair_correlations_ranked.tsv. High-ranking positive pairs are candidate transcription factor targets, as the authors used COnTORT files to correlate expression across a large number of diverse data sets to identify new targets of transcription factors in bacteria.

## Related tools

- **COnTORT** (Produces the mean-centered, GenBank-annotated transcriptomic matrix that is the input to the correlation analysis) — https://github.com/GLBRC/contort
- **pandas** (Loads the tab-delimited COnTORT matrix and computes pairwise Pearson correlations between gene rows across experiment columns)
- **Python 3** (Runtime for the correlation and ranking workflow)

## Examples

```
import pandas as pd; df = pd.read_csv('COnTORT_organized_transcriptomic_data.txt', sep='\t', index_col=0); corr = df.iloc[:, 5:].T.corr(method='pearson')
```

## Evaluation signals

- All correlation values in the output lie within [-1, 1]
- gene_pair_correlations_ranked.tsv is sorted in descending rank order of correlation coefficient
- No self-pairs (gene paired with itself) appear in the output table
- Every gene pair row carries both genes' GenBank annotations (locus tag, gene name, product) with 'N/A' only where the annotation was absent from the GenBank file
- The number of experiment columns used equals the number of GSM ID headers in COnTORT_organized_transcriptomic_data.txt

## Limitations

- COnTORT can only download and combine normalized and processed data imported into NCBI GEO, so downstream correlations inherit whatever normalization each GEO series applied
- Annotations not present in the GenBank file are listed with 'N/A', limiting pair annotation for genes missing from the provided .gbff
- Correlation ranking is purely expression-based; high-ranking pairs are candidates that still require biological validation as transcription factor targets

## Evidence

- [abstract] We have used COnTORT files to correlate expression of genes across a large number of diverse data sets to identify new targets of transcription factors in bacteria.: "We have used COnTORT files to correlate expression of genes across a large number of diverse data sets to identify new targets of transcription factors in bacteria."
- [abstract] One output file will mean-center the data based on experiments to control for differences between experiments.: "One output file will mean-center the data based on experiments to control for differences between experiments, while the second output file will not mean-center the data."
- [abstract] The first five columns contain the annotation information from the GenBank file (locus tag, old locus tag, gene name, gene synonyms, and product): "The first five columns contain the annotation information from the GenBank file (locus tag, old locus tag, gene name, gene synonyms, and product)"
- [readme] README output description of primary file and mean-centering variants: "Primary output is a single text file containing the gene annotations on the first five columns and the mean centered expression data present in the remaining columns."
