---
name: regression-coefficient-normalization
description: Use when you need to quantify or rank how strongly individual REMs associated
  to a gene contribute to that gene's expression in a specific cell type, e.g. after
  a GeneQuery retrieval of regressionCoefficient and DNase1 signal data.
when_to_use_negative:
- You want to identify enriched TF motifs in REM sequences — use PASTAA or similar
  motif enrichment tools instead of the Cell type score normalization.
- You need the actual regulatory regions linked to a gene from scratch — first run
  EpiRegio's GeneQuery; this skill only normalizes coefficients for REMs already associated
  to a gene.
- The input is a single REM without its full set R of associated REMs, since the denominator
  requires summing over all REMs of the gene.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: EpiRegio
  role: Source of the REM table (regressionCoefficient, per-cell-type DNase1 signal,
    cell type score) via the GeneQuery REST API endpoint
  repo: github.com/TeamRegio/EpiRegioDB
- name: Requests
  role: Python package used to query the EpiRegio REST API GeneQuery endpoint and
    retrieve the REM table
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkaa382
    title: 'EpiRegio: analysis and retrieval of regulatory elements linked to genes'
  source_content_hash: sha256:65820ffa7cb6decca328f531e13fd6089179ccf10d2322e0b097f378f25b4ff7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EpiRegio
  - Requests
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# regression-coefficient-normalization

## Summary

Computes EpiRegio's Cell type score, which normalizes each REM's regression coefficient times its per-cell-type DNase1 signal over all REMs of a gene, yielding a cell type-specific contribution to the target gene's expression bounded in [-1, 1].

## When to use

Use when you need to quantify or rank how strongly individual REMs associated to a gene contribute to that gene's expression in a specific cell type, e.g. after a GeneQuery retrieval of regressionCoefficient and DNase1 signal data.

## When NOT to use

- You want to identify enriched TF motifs in REM sequences — use PASTAA or similar motif enrichment tools instead of the Cell type score normalization.
- You need the actual regulatory regions linked to a gene from scratch — first run EpiRegio's GeneQuery; this skill only normalizes coefficients for REMs already associated to a gene.
- The input is a single REM without its full set R of associated REMs, since the denominator requires summing over all REMs of the gene.

## Inputs

- REM table from EpiRegio GeneQuery (regressionCoefficient beta_r and per-cell-type DNase1 signals per REM)
- Set R of all REMs associated to a given gene (Ensembl gene ID, e.g. ENSG00000223972)

## Outputs

- cell_type_scores.csv with REM IDs, cell types, and Cell type scores in [-1, 1]
- Ranked REMs per cell type (sortable by Cell type score)

## How to apply

Retrieve the REM table for a chosen gene via the EpiRegio REST API GeneQuery endpoint (e.g. https://epiregio.de/REST_API/GeneQuery/ENSG00000223972/) using the Python Requests package. Log-transform the per-cell-type DNase1 signals, average over all samples of each cell type, and standardize them per REM across cell types (mean=0, standard deviation=1). For each REM r in cell type c compute the Cell type score as beta_r * DNase1(r,c) divided by the sum of absolute values of beta_ri * DNase1(ri,c) over all REMs R associated to the gene. This normalization, based on the STITCHIT linear model, guarantees each score lies in [-1, 1] and that absolute scores of all REMs of one gene sum to 1 per cell type, which serves as the validation check before writing results to cell_type_scores.csv.

## Related tools

- **EpiRegio** (Source of the REM table (regressionCoefficient, per-cell-type DNase1 signal, cell type score) via the GeneQuery REST API endpoint) — github.com/TeamRegio/EpiRegioDB
- **Requests** (Python package used to query the EpiRegio REST API GeneQuery endpoint and retrieve the REM table)

## Examples

```
import requests; r = requests.get('https://epiregio.de/REST_API/GeneQuery/ENSG00000223972/'); rem_table = r.()  # then compute score = beta_r*DNase1(r,c)/sum(|beta_ri*DNase1(ri,c)|)
```

## Evaluation signals

- Every computed Cell type score lies within [-1, 1].
- For each gene and each cell type, the sum of absolute Cell type scores over all REMs R equals 1 (within floating-point tolerance).
- DNase1 signals are log-transformed and standardized per REM across cell types with mean=0 and standard deviation=1 before the ratio is computed.
- The output cell_type_scores.csv contains one row per (REM, cell type) pair with REM ID, cell type, and score columns.

## Limitations

- The linear model used in STITCHIT, which forms the basis for assessing the Cell type score, is a simplification of the actual regulation of genes by their REMs.
- The score is only defined relative to the set of REMs associated to one gene; it cannot be compared as an absolute regulatory strength across genes.
- Scores depend on the DNase1 signal averaging over all samples of each cell type as provided in the API output.

## Evidence

- [other] Compute the Cell type score for each REM r in cell type c as beta_r * DNase1(r,c) divided by the sum of absolute values of beta_ri * DNase1(ri,c) over all REMs R associated to the gene.: [evidence span withheld — anchor status: not_found]
- [discussion] The linear model used in STITCHIT forms the basis for assessing the Cell type score.: "the linear model used in STITCHIT, which forms the basis for assessing the Cell type score, is a simplification to the actual regulation of genes by their REMs"
- [methods] Use EpiRegio's Gene Query to identify the REMs associated to the genes of interest.: "Use EpiRegio's `Gene Query <https://epiregiodb.readthedocs.io/en/latest/UseCases.html#query-guide>`_ to identify the REMs associated to the genes of interest"
- [abstract] The EpiRegio web server provides REM activity and estimated cell type-specific contribution to target gene expression.: "Our web server allows the analysis of genes and their associated REMs, including the REM’s activity and its estimated cell type-specific contribution to its target gene’s expression."
