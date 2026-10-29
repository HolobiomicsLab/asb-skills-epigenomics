---
name: rest-api-data-retrieval
description: Use when you need programmatic access to EpiRegio's REM data — e.g. regressionCoefficient
  and per-cell-type DNase1 signal and cell type score values — for a chosen gene identified
  by an Ensembl ID such as ENSG00000223972.
when_to_use_negative:
- When the analysis goal is TF motif enrichment on REM sequences (PASTAA/Clover/MotifCounter/Homer)
  — retrieve sequences via bedtools getfasta instead of raw API tables
- When intersecting TF ChIP-seq peaks with REMs — use EpiRegio's Region Query with
  a 50% overlap option rather than the GeneQuery endpoint
- When the gene has no REM associations in EpiRegioDB — the endpoint will not return
  a usable REM table
edam_operation: http://edamontology.org/operation_2422
tools:
- name: EpiRegio
  role: Resource of REMs queried via its REST API GeneQuery endpoint to retrieve REM
    tables (regression coefficients, DNase1 signals, cell type scores) for a gene
  repo: https://epiregio.de/
- name: Requests
  role: Python HTTP library used to call the EpiRegio REST API endpoints programmatically
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkaa382
    title: 'EpiRegio: analysis and retrieval of regulatory elements linked to genes'
  source_content_hash: sha256:b5232c99fd66560f1f55c82d195d25cb24b7609f1959fb5874cb25f7633e4abd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EpiRegio
  - Requests
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# rest-api-data-retrieval

## Summary

Programmatically retrieve regulatory element (REM) tables from EpiRegio via its REST API (e.g. the GeneQuery endpoint) using the Python Requests package, obtaining regression coefficients and per-cell-type DNase1 signals for a gene's associated REMs.

## When to use

Use when you need programmatic access to EpiRegio's REM data — e.g. regressionCoefficient and per-cell-type DNase1 signal and cell type score values — for a chosen gene identified by an Ensembl ID such as ENSG00000223972.

## When NOT to use

- When the analysis goal is TF motif enrichment on REM sequences (PASTAA/Clover/MotifCounter/Homer) — retrieve sequences via bedtools getfasta instead of raw API tables
- When intersecting TF ChIP-seq peaks with REMs — use EpiRegio's Region Query with a 50% overlap option rather than the GeneQuery endpoint
- When the gene has no REM associations in EpiRegioDB — the endpoint will not return a usable REM table

## Inputs

- Ensembl gene identifier (e.g. ENSG00000223972) for a gene of interest
- Optional: cell types/tissues filter string (e.g. 'heart') and Cell type score / Cell type DNase1 signal thresholds

## Outputs

- REM table (per gene) containing REM IDs, regressionCoefficient values, per-cell-type DNase1 signals, and cell type scores
- Optionally a downloaded CSV of the Gene Query result table

## How to apply

Issue an HTTP GET against EpiRegio's REST API, e.g. https://epiregio.de/REST_API/GeneQuery/ENSG00000223972/, using the Python Requests package to fetch the REM table for the gene of interest. The returned table contains each REM's regressionCoefficient plus its DNase1 signal per cell type, averaged over all samples of each cell type as provided in the API output. Parse the response to extract REM IDs, beta_r values, and per-cell-type DNase1 signals, then downstream these feed into computations such as the Cell type score (beta_r * DNase1(r,c) normalized over all REMs R associated to the gene). Optionally use EpiRegio's web-interface Gene Query first to identify the REMs associated to the genes of interest, filtered by cell types/tissues (e.g. 'heart') and by Cell type score / Cell type DNase1 signal thresholds, and download the resulting table as CSV.

## Related tools

- **EpiRegio** (Resource of REMs queried via its REST API GeneQuery endpoint to retrieve REM tables (regression coefficients, DNase1 signals, cell type scores) for a gene) — https://epiregio.de/
- **Requests** (Python HTTP library used to call the EpiRegio REST API endpoints programmatically)

## Examples

```
import requests; r = requests.get('https://epiregio.de/REST_API/GeneQuery/ENSG00000223972/'); rem_table = r.()
```

## Evaluation signals

- HTTP response is valid JSON/structured table containing regressionCoefficient and per-cell-type DNase1 signal fields for each REM of the queried gene
- Every REM ID returned is associated to the queried Ensembl gene ID
- Downstream validation: if computing Cell type scores from the retrieved table, all values lie in [-1, 1] and absolute scores per gene per cell type sum to 1 (within floating-point tolerance)
- DNase1 signals are log-transformed and standardized per REM across cell types (mean=0, standard deviation=1) as expected for the linear model basis
- Result files (e.g. cell_type_scores.csv) contain REM IDs, cell types, and scores as columns

## Limitations

- The linear model used in STITCHIT, which forms the basis for assessing the Cell type score, is a simplification to the actual regulation of genes by their REMs
- No tool versions are stated for any of the software mentioned (including EpiRegio)
- REMs cannot be located with absolute certainty; EpiRegio's annotations are indirect epigenomic indicators of regulatory activity

## Evidence

- [methods] Retrieve the REM table (regressionCoefficient and per-cell-type DNase1 signal and cell type score) for a chosen gene via the EpiRegio REST API GeneQuery endpoint: [evidence span withheld — anchor status: not_found]
- [abstract] REST API availability: "EpiRegio allows programmatic access through a REST API and is freely available at https://epiregio.de/."
- [methods] Gene Query usage: "Use EpiRegio's `Gene Query <https://epiregiodb.readthedocs.io/en/latest/UseCases.html#query-guide>`_ to identify the REMs associated to the genes of interest"
- [other] REST API endpoint example: "https://epiregio.de/REST_API/GeneQuery/ENSG00000223972/"
- [readme] Repository pointer: "EpiRegio is a resource of Regulatory EleMents (REMs) associated to protein coding, and non-protein coding genes."
