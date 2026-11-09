---
name: epigenomic-dataset-summary-statistics-validation
description: Use when a task asks to rebuild or validate a regulatory-element database
  from chromatin accessibility plus matched expression data, and database-level counts
  (total REMs, associated genes, mean REM length) must be reproduced.
when_to_use_negative:
- The task is to query an existing REM resource (e.g. EpiRegio Gene Query or Region
  Query for a TF of interest) rather than rebuild or validate the underlying database
  statistics
- Input data lacks matched chromatin accessibility (DNase1-seq) and gene expression
  (RNA-seq) samples across multiple cell types, since the StitchIt model requires
  paired signal-expression variation across samples
- Only peak-calling-style REM discovery is needed without gene-expression-linked annotation;
  EpiRegio's StitchIt approach deliberately annotates regions relative to actual gene
  expression changes, which is a different objective
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: STITCHIT
  role: Linear model that interprets depth-normalized DNase1-seq chromatin accessibility
    with respect to variation in gene expression (TPM) to identify REMs with regression
    coefficients
- name: EpiRegio
  role: Web server / database resource whose published REM statistics serve as the
    reference values to reproduce; REMs are grouped into CREMs with unique IDs
  repo: github.com/TeamRegio/EpiRegioDB
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkaa382
    title: 'EpiRegio: analysis and retrieval of regulatory elements linked to genes'
  source_content_hash: sha256:f1b26508ca55d7c439abe596f55b63c8afa09f6f5b0cb00e52e9fb56c0ec2684
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EpiRegio
  - STITCHIT
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# epigenomic-dataset-summary-statistics-validation

## Summary

Reproduces the database-level summary statistics of the EpiRegio resource by running the StitchIt algorithm on paired DNase1-seq and RNA-seq data from 110 Roadmap and 56 Blueprint samples (46 tissues/cell types) and validating the computed counts against the published reference metrics.

## When to use

Use when a task asks to rebuild or validate a regulatory-element database from chromatin accessibility plus matched expression data, and database-level counts (total REMs, associated genes, mean REM length) must be reproduced.

## When NOT to use

- The task is to query an existing REM resource (e.g. EpiRegio Gene Query or Region Query for a TF of interest) rather than rebuild or validate the underlying database statistics
- Input data lacks matched chromatin accessibility (DNase1-seq) and gene expression (RNA-seq) samples across multiple cell types, since the StitchIt model requires paired signal-expression variation across samples
- Only peak-calling-style REM discovery is needed without gene-expression-linked annotation; EpiRegio's StitchIt approach deliberately annotates regions relative to actual gene expression changes, which is a different objective

## Inputs

- Paired DNase1-seq and RNA-seq samples from the Roadmap (110 samples) and Blueprint (56 samples) consortia, uniformly preprocessed
- GENCODE V27 gene annotation
- Search-window definition: 100,000 bp upstream of TSS, full gene body, 100,000 bp downstream of transcription termination site

## Outputs

- Set of regulatory elements (REMs) with per-gene regression coefficients from the StitchIt linear model
- Clusters of Regulatory EleMents (CREMs, minimum two REMs, unique IDs)
- Database summary metrics (stitchit_database_metrics.csv): total REMs, total genes, total CREMs, number of tissues/cell types, mean and SD of REM length

## How to apply

Download the uniformly preprocessed paired DNase1-seq and RNA-seq samples used by EpiRegio (110 Roadmap samples plus 56 Blueprint samples spanning 46 tissues/cell types) together with the GENCODE V27 gene annotation. For every gene, restrict analysis to the StitchIt search window: 100,000 bp upstream of the TSS, the entire gene body, and 100,000 bp downstream of the transcription termination site. Partition the window into regions whose depth-normalized DNase1-seq signal correlates with gene expression quantified in transcripts per million (TPM), fitting the StitchIt linear model per gene to obtain REMs with regression coefficients. Aggregate REMs across all 166 samples, assign each REM to its gene, and group overlapping or directly adjacent REMs into Clusters of Regulatory EleMents (CREMs, minimum two REMs, unique CREM ID). Finally compute total REM count, associated gene count, CREM count, and mean/standard deviation of REM length, and check them against the published values: 2,404,861 REMs, 35,379 protein-coding and non-protein-coding genes, 365,286 CREMs, 46 tissues/cell types, and an average REM length of 229 bp (±235 bp).

## Related tools

- **STITCHIT** (Linear model that interprets depth-normalized DNase1-seq chromatin accessibility with respect to variation in gene expression (TPM) to identify REMs with regression coefficients)
- **EpiRegio** (Web server / database resource whose published REM statistics serve as the reference values to reproduce; REMs are grouped into CREMs with unique IDs) — github.com/TeamRegio/EpiRegioDB

## Evaluation signals

- Total REM count reproduces the published value of 2,404,861 REMs
- Number of associated protein-coding and non-protein-coding genes reproduces 35,379
- Number of distinct CREMs reproduces 365,286 (each CREM contains a minimum of two REMs and carries a unique ID)
- Mean REM length reproduces 229 bp with standard deviation ±235 bp
- Tissue/cell-type coverage equals 46, matching the 110 Roadmap + 56 Blueprint sample panel; metrics are recorded in stitchit_database_metrics.csv

## Limitations

- The linear model used in STITCHIT, which forms the basis for assessing the Cell type score, is a simplification of the actual regulation of genes by their REMs
- Identifying REMs is inherently uncertain — there is no method yet to locate them with absolute certainty; StitchIt relies on indirect epigenomic indicators
- No tool versions are stated for the software involved (STITCHIT, EpiRegio), so exact reproducibility depends on dataset versions available via the Zenodo record 3758494

## Evidence

- [intro] EpiRegio uses the StitchIt algorithm which interprets chromatin accessibility data with respect to variation in gene expression: "EpiRegio uses the StitchIt algorithm which interprets chromatin accessibility data with respect to variation in gene expression (21). The algorithm identifies putative REMs that explain variation of"
- [discussion] the linear model used in STITCHIT is a simplification of actual gene regulation: "the linear model used in STITCHIT, which forms the basis for assessing the Cell type score, is a simplification to the actual regulation of genes by their REMs"
- [readme] EpiRegio is a resource of REMs associated to protein coding and non-protein coding genes: "EpiRegio is a resource of Regulatory EleMents (REMs) associated to protein coding, and non-protein coding genes."
- [intro] StitchIt's gene-expression-linked annotation advantage: "StitchIt’s approach to interpret epigenomic variation in gene expression has the advantage of annotating regions that were observed in relation to actual gene expression changes, potentially leading"
