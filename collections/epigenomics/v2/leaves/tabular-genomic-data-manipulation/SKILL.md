---
name: tabular-genomic-data-manipulation
description: Use when you hold a COnTORT mean-centered tab-delimited transcriptomic
  matrix (locus tag, old locus tag, gene name, gene synonyms, product + GSM-labeled
  columns) and need ranked co-expressed gene pairs across many GEO experiments.
when_to_use_negative:
- The input is the non-mean-centered COnTORT output file — experiment-to-experiment
  offsets are not controlled for, so cross-experiment correlations are not comparable.
- The data are raw or unnormalized reads not imported into NCBI GEO; COnTORT can only
  download and combine normalized and processed data imported into NCBI GEO.
- The goal is retrieving/organizing GEO data rather than downstream analysis of an
  already-organized COnTORT matrix (use the COnTORT download pipeline instead).
edam_operation: http://edamontology.org/operation_0531
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: COnTORT
  role: Produces the mean-centered, annotation-joined single text file that this skill
    consumes for correlation analysis
  repo: https://github.com/GLBRC/contort
- name: pandas
  role: Python module used to load the tab-delimited matrix and compute pairwise Pearson
    correlations between gene rows
- name: Python 3
  role: Runtime for the pandas-based correlation and ranking workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1128/mra.00587-20
    title: 'COnTORT: COmprehensive Transcriptomic ORganizational Tool for Simultaneously
      Retrieving and Organizing Numerous Gene Expression Data Sets from the NCBI Gene
      Expression Omnibus Database'
  source_content_hash: sha256:352c47bb0dca05e9915d96daf5548c177a70a3dcbe55c386b0698de87b2f4707
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GLBRC/contort
  tools:
  - COnTORT
  - Python 3
  - pandas
  edam_operation: http://edamontology.org/operation_0531
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# tabular-genomic-data-manipulation

## Summary

Manipulates the COnTORT-organized, mean-centered transcriptomic matrix (COnTORT_organized_transcriptomic_data.txt) — a tab-delimited file with five GenBank annotation columns followed by per-experiment expression columns — to compute and rank pairwise gene-gene Pearson correlations for identifying candidate transcription factor targets.

## When to use

Use when you hold a COnTORT mean-centered tab-delimited transcriptomic matrix (locus tag, old locus tag, gene name, gene synonyms, product + GSM-labeled columns) and need ranked co-expressed gene pairs across many GEO experiments.

## When NOT to use

- The input is the non-mean-centered COnTORT output file — experiment-to-experiment offsets are not controlled for, so cross-experiment correlations are not comparable.
- The data are raw or unnormalized reads not imported into NCBI GEO; COnTORT can only download and combine normalized and processed data imported into NCBI GEO.
- The goal is retrieving/organizing GEO data rather than downstream analysis of an already-organized COnTORT matrix (use the COnTORT download pipeline instead).

## Inputs

- COnTORT_organized_transcriptomic_data.txt (tab-delimited, mean-centered per experiment; first five columns = locus tag, old locus tag, gene name, gene synonyms, product; remaining columns = per-experiment expression headed by GEO IDs)
- GenBank (.gbff) annotation file for the organism (source of the five annotation columns)

## Outputs

- gene_pair_correlations_ranked.tsv (gene pair annotations plus Pearson correlation coefficient, sorted descending, self-pairs excluded)

## How to apply

Load COnTORT_organized_transcriptomic_data.txt with Python 3 pandas, treating the first five annotation columns (locus tag, old locus tag, gene name, gene synonyms, product) as gene row identifiers and the remaining GSM-labeled columns as mean-centered expression values; the mean centering is what controls for inter-experiment differences so correlations are computed across diverse data sets. Compute pairwise Pearson correlation coefficients between all gene rows across the experiment columns with pandas (e.g., df.T.corr()), assemble a gene-pair table with annotation identifiers for both genes, and exclude self-pairs. Rank pairs by correlation in descending order (or by absolute correlation to capture anti-correlated pairs) and write the result as a TSV, gene_pair_correlations_ranked.tsv. Validate that all coefficients lie within [-1, 1] and that rows are sorted in descending correlation order before release.

## Related tools

- **COnTORT** (Produces the mean-centered, annotation-joined single text file that this skill consumes for correlation analysis) — https://github.com/GLBRC/contort
- **pandas** (Python module used to load the tab-delimited matrix and compute pairwise Pearson correlations between gene rows)
- **Python 3** (Runtime for the pandas-based correlation and ranking workflow)

## Examples

```
import pandas as pd; df = pd.read_csv('COnTORT_organized_transcriptomic_data.txt', sep='\t'); expr = df.iloc[:,5:]; corr = expr.T.corr(); corr.to_csv('gene_pair_correlations_ranked.tsv', sep='\t')
```

## Evaluation signals

- Every correlation value in gene_pair_correlations_ranked.tsv lies within [-1, 1].
- The gene-pair table is sorted in descending rank order of correlation coefficient.
- No self-pairs (same locus tag on both sides) appear in the output table.
- Both genes in each output row carry annotation identifiers drawn from the five GenBank-derived columns of the input file.
- The number of pairwise comparisons matches n*(n-1)/2 for n gene rows (or the full off-diagonal count if directionality is retained).

## Limitations

- COnTORT can only download and combine normalized and processed data imported into NCBI GEO, so correlations are limited to such data.
- Mean-centered output is per experiment; annotations absent from the provided GenBank file are listed as 'N/A' and may not map cleanly.
- Pearson correlation across a large number of diverse data sets ranks co-expressed pairs but does not itself establish regulatory relationships; targets are candidates requiring further validation.

## Evidence

- [abstract] Abstract claim that files support correlation analyses: "We have used COnTORT files to correlate expression of genes across a large number of diverse data sets to identify new targets of transcription factors in bacteria."
- [abstract] Mean-centering rationale: "One output file will mean-center the data based on experiments to control for differences between experiments"
- [readme] Output file schema: "Primary output is a single text file containing the gene annotations on the first five columns and the mean centered expression data present in the remaining columns."
- [abstract] Annotation columns: "The first five columns contain the annotation information from the GenBank file (locus tag, old locus tag, gene name, gene synonyms, and product)"
- [intro] Scope limitation of source data: "COnTORT can only download and combine normalized and processed data imported into NCBI GEO."
