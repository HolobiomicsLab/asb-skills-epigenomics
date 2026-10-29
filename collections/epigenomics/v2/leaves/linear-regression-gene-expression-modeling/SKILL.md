---
name: linear-regression-gene-expression-modeling
description: Use when you have paired chromatin accessibility (DNase1-seq) and RNA-seq
  expression data across many samples/cell types and need to identify putative REMs
  that explain expression variation of specific genes.
when_to_use_negative:
- Input is a single sample without matched DNase1-seq accessibility and RNA-seq expression
  across multiple conditions — the linear model requires variation across samples
  to correlate signal with expression.
- The goal is de novo peak calling from raw reads rather than annotating regions in
  relation to actual gene expression changes; StitchIt is not a peak caller.
- You already have REM–gene associations and only need retrieval or TF motif analysis
  — use EpiRegio's Gene Query / Region Query instead of refitting the model.
edam_operation: http://edamontology.org/operation_3657
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0203
tools:
- name: STITCHIT
  role: Fits the per-gene linear model relating depth-normalized DNase1-seq signal
    to gene expression (TPM) to identify REMs; the Cell type score is derived from
    this linear model.
- name: EpiRegio
  role: Web server and database built on StitchIt REMs; used for Gene Query/Region
    Query and Cell type scoring of the resulting REM–gene associations.
  repo: https://github.com/TeamRegio/EpiRegioDB
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkaa382
    title: 'EpiRegio: analysis and retrieval of regulatory elements linked to genes'
  source_content_hash: sha256:4fecd33daf8134b78d18cf25d699a9b246bc909490a205ff5722667499cab185
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TeamRegio/EpiRegioDB
  tools:
  - EpiRegio
  - STITCHIT
  edam_operation: http://edamontology.org/operation_3657
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# linear-regression-gene-expression-modeling

## Summary

Fits the STITCHIT linear model that partitions a gene's search window into regions whose depth-normalized DNase1-seq signal correlates with gene expression (transcripts per million), yielding regulatory elements (REMs) with regression coefficients. This is the core gene-centric approach behind EpiRegio's REM–gene associations.

## When to use

Use when you have paired chromatin accessibility (DNase1-seq) and RNA-seq expression data across many samples/cell types and need to identify putative REMs that explain expression variation of specific genes.

## When NOT to use

- Input is a single sample without matched DNase1-seq accessibility and RNA-seq expression across multiple conditions — the linear model requires variation across samples to correlate signal with expression.
- The goal is de novo peak calling from raw reads rather than annotating regions in relation to actual gene expression changes; StitchIt is not a peak caller.
- You already have REM–gene associations and only need retrieval or TF motif analysis — use EpiRegio's Gene Query / Region Query instead of refitting the model.

## Inputs

- Uniformly preprocessed paired DNase1-seq samples (e.g. 110 Roadmap + 56 Blueprint samples across 46 tissues/cell types)
- Matched RNA-seq gene expression quantified in transcripts per million (TPM)
- GENCODE V27 gene annotation (TSS, gene body, transcription termination sites)

## Outputs

- REMs (regulatory elements) with per-gene regression coefficients
- CREMs (Clusters of Regulatory EleMents, minimum two REMs, unique CREM IDs)
- Database-level summary statistics (e.g. total REMs, associated genes, CREM count, mean/SD REM length)

## How to apply

For each gene, restrict analysis to the STITCHIT search window: 100,000 bp upstream of the transcription start site, the entire gene body, and 100,000 bp downstream of the transcription termination site. Partition this window into distinct regions whose sequencing-depth-normalized DNase1-seq signal varies in correlation with gene expression quantified in transcripts per million, fitting the STITCHIT linear model per gene to obtain REMs with regression coefficients. Assign each REM to its gene, and group overlapping or directly adjacent REMs into Clusters of Regulatory EleMents (CREMs, minimum two REMs, unique CREM ID). The rationale is that EpiRegio starts with a gene and looks for REMs (not vice versa), annotating regions in relation to actual gene expression changes, which can yield higher specificity than peak-calling-based approaches.

## Related tools

- **STITCHIT** (Fits the per-gene linear model relating depth-normalized DNase1-seq signal to gene expression (TPM) to identify REMs; the Cell type score is derived from this linear model.)
- **EpiRegio** (Web server and database built on StitchIt REMs; used for Gene Query/Region Query and Cell type scoring of the resulting REM–gene associations.) — https://github.com/TeamRegio/EpiRegioDB

## Evaluation signals

- Reproduce the EpiRegio reference database statistics: 2,404,861 REMs, 35,379 protein-coding and non-protein-coding genes, 365,286 CREMs, and average REM length of 229 bp (±235 bp).
- Verify REMs fall within the STITCHIT search window: 100,000 bp upstream of TSS, full gene body, and 100,000 bp downstream of transcription termination site.
- Check every CREM contains a minimum of two REMs and carries a unique CREM ID; check overlapping/directly adjacent REMs are merged.
- Confirm DNase1-seq signal used in the fit is sequencing-depth normalized and expression is quantified in transcripts per million.
- Validate the fitted regions show DNase1-seq signal variation that correlates with gene expression variation across the 166 Roadmap+Blueprint samples.

## Limitations

- The linear model used in STITCHIT, which forms the basis for assessing the Cell type score, is a simplification of the actual regulation of genes by their REMs.
- Identifying REMs is difficult — no method locates them with absolute certainty; the linear model yields putative REMs based on indirect epigenomic indicators.
- No tool versions are stated for the software used, so exact reproducibility of version-dependent behavior is not guaranteed.

## Evidence

- [intro] EpiRegio uses the StitchIt algorithm which interprets chromatin accessibility data with respect to variation in gene expression: "EpiRegio uses the StitchIt algorithm which interprets chromatin accessibility data with respect to variation in gene expression (21)."
- [discussion] StitchIt linear model is a simplification: "the linear model used in STITCHIT, which forms the basis for assessing the Cell type score, is a simplification to the actual regulation of genes by their REMs"
- [intro] StitchIt gene-centric REM identification: "In contrast to existing approaches, it starts with a gene and looks for REMs, not viceversa."
- [intro] StitchIt approach rationale: "StitchIt’s approach to interpret epigenomic variation in gene expression has the advantage of annotating regions that were observed in relation to actual gene expression changes, potentially leading"
- [full_text] STITCHIT search window and REM partitioning from task workflow: [evidence span withheld — anchor status: not_found]
- [full_text] Fitting the linear model per gene: [evidence span withheld — anchor status: not_found]
- [full_text] Reference database metrics for validation: [evidence span withheld — anchor status: not_found]
