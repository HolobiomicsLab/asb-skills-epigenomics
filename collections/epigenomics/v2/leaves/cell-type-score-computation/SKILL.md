---
name: cell-type-score-computation
description: Use when you have EpiRegio GeneQuery REM tables (regressionCoefficient,
  per-cell-type DNase1 signal) for a gene and need cell type-specific REM contributions
  to gene expression.
when_to_use_negative:
- Input is already a scored REM table (e.g. a downloaded CSV with Cell type score
  thresholds already applied) rather than raw regressionCoefficient and DNase1 signals
- The goal is TF motif enrichment or ChIP-seq peak overlap analysis rather than quantifying
  REM contributions to gene expression
tools:
- name: EpiRegio
  role: Source of the REM table (regressionCoefficient, per-cell-type DNase1 signal,
    cell type score) via the GeneQuery REST endpoint
  repo: https://github.com/TeamRegio/EpiRegioDB
- name: Requests
  role: Python HTTP client used to call the EpiRegio REST API GeneQuery endpoint programmatically
- name: STITCHIT
  role: Linear model underlying the Cell type score; provides the beta (regressionCoefficient)
    values used in the normalization
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkaa382
    title: 'EpiRegio: analysis and retrieval of regulatory elements linked to genes'
  source_content_hash: sha256:32937c84843d4c948793a6f686f9722956b95213dd4da349215d02e8827afbf5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TeamRegio/EpiRegioDB
  tools:
  - EpiRegio
  - Requests
  - STITCHIT
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# cell-type-score-computation

## Summary

Computes EpiRegio's Cell type score, which normalizes the contribution of each REM to its target gene's expression in a specific cell type, as predicted by the STITCHIT linear model, and validates that scores fall in [-1, 1] and sum to 1 in absolute value per gene per cell type.

## When to use

Use when you have EpiRegio GeneQuery REM tables (regressionCoefficient, per-cell-type DNase1 signal) for a gene and need cell type-specific REM contributions to gene expression.

## When NOT to use

- Input is already a scored REM table (e.g. a downloaded CSV with Cell type score thresholds already applied) rather than raw regressionCoefficient and DNase1 signals
- The goal is TF motif enrichment or ChIP-seq peak overlap analysis rather than quantifying REM contributions to gene expression

## Inputs

- EpiRegio REST API GeneQuery response table for a gene (regressionCoefficient and per-cell-type DNase1 signal fields, multiple samples per cell type)
- Ensembl gene identifier of the target gene (e.g. ENSG00000223972)

## Outputs

- cell_type_scores.csv with columns REM ID, cell type, and Cell type score

## How to apply

Retrieve the REM table for a chosen gene via the EpiRegio REST API GeneQuery endpoint (e.g. https://epiregio.de/REST_API/GeneQuery/ENSG00000223972/) using the Python Requests package. Log-transform the DNase1 signals and standardize them per REM across cell types (mean=0, standard deviation=1), averaging over all samples of each cell type as provided in the API output. For each REM r in cell type c, compute the Cell type score as beta_r * DNase1(r,c) divided by the sum of absolute values of beta_ri * DNase1(ri,c) over all REMs R associated to the gene, using the regressionCoefficient from the API. Validate that every score lies in [-1, 1] and that absolute Cell type scores of all REMs of one gene sum to 1 per cell type, then write REM IDs, cell types, and scores to cell_type_scores.csv.

## Related tools

- **EpiRegio** (Source of the REM table (regressionCoefficient, per-cell-type DNase1 signal, cell type score) via the GeneQuery REST endpoint) — https://github.com/TeamRegio/EpiRegioDB
- **Requests** (Python HTTP client used to call the EpiRegio REST API GeneQuery endpoint programmatically)
- **STITCHIT** (Linear model underlying the Cell type score; provides the beta (regressionCoefficient) values used in the normalization)

## Examples

```
import requests; r = requests.get('https://epiregio.de/REST_API/GeneQuery/ENSG00000223972/')
```

## Evaluation signals

- Every computed Cell type score in cell_type_scores.csv lies in the range [-1, 1]
- For each gene and each cell type, the sum of absolute Cell type scores over all REMs equals 1 (within floating-point tolerance)
- Per-REM DNase1 signals are log-transformed and standardized to mean=0, standard deviation=1 across cell types before scoring
- The set of REMs scored matches the REMs returned by the GeneQuery endpoint for the target gene

## Limitations

- The linear model used in STITCHIT, which forms the basis for assessing the Cell type score, is a simplification of the actual regulation of genes by their REMs
- Scores depend on the REMs returned by EpiRegio for the queried gene; a different gene's REM set yields a different normalization denominator
- DNase1 signals must be averaged over all samples of each cell type as provided in the API output; per-sample variation is collapsed

## Evidence

- [discussion] STITCHIT limitation: "the linear model used in STITCHIT, which forms the basis for assessing the Cell type score, is a simplification to the actual regulation of genes by their REMs"
- [abstract] Cell type score semantics: "Our web server allows the analysis of genes and their associated REMs, including the REM’s activity and its estimated cell type-specific contribution to its target gene’s expression."
- [abstract] REST access: "EpiRegio allows programmatic access through a REST API and is freely available at https://epiregio.de/."
- [readme] Scope of EpiRegio resource: "EpiRegio is a resource of Regulatory EleMents (REMs) associated to protein coding, and non-protein coding genes."
