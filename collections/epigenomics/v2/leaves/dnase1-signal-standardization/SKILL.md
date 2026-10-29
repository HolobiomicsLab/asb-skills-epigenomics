---
name: dnase1-signal-standardization
description: Use when computing or reproducing EpiRegio Cell type scores from GeneQuery
  REST API output, where per-cell-type DNase1 signals of REMs must be log-transformed
  and z-scored per REM before weighting by regression coefficients.
when_to_use_negative:
- When you only need raw DNase1 accessibility magnitudes rather than a REM's relative
  contribution to gene expression — the z-scoring removes absolute signal level per
  REM.
- 'When the gene has only one associated REM: the Cell type score normalization over
  the set R of REMs is trivial/degenerate in that case.'
- When working with TF ChIP-seq peak overlap or motif enrichment workflows (e.g. PASTAA
  on REM sequences), which use REM coordinates/sequences rather than standardized
  DNase1 signals.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0749
tools:
- name: EpiRegio
  role: Provides the REM table with regressionCoefficient and per-cell-type DNase1
    signal via the GeneQuery REST API endpoint; input source for the standardization
  repo: https://github.com/TeamRegio/EpiRegioDB
- name: Requests
  role: Python package used to call the EpiRegio GeneQuery REST API and fetch the
    REM/DNase1 table programmatically
- name: STITCHIT
  role: Linear model underlying the regression coefficients (beta_r) that weight the
    standardized DNase1 signals in the Cell type score
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkaa382
    title: 'EpiRegio: analysis and retrieval of regulatory elements linked to genes'
  source_content_hash: sha256:1a64af47825f5d55091f9839f455cf391d4218554086646f366fc50f682896ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TeamRegio/EpiRegioDB
  tools:
  - EpiRegio
  - Requests
  - STITCHIT
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# dnase1-signal-standardization

## Summary

Log-transform and standardize per-cell-type DNase1 signals for each REM (mean=0, sd=1 across cell types) so that the standardized signals can be combined with StitchIt regression coefficients to compute EpiRegio's Cell type score of a REM's cell type-specific contribution to its target gene's expression.

## When to use

Use when computing or reproducing EpiRegio Cell type scores from GeneQuery REST API output, where per-cell-type DNase1 signals of REMs must be log-transformed and z-scored per REM before weighting by regression coefficients.

## When NOT to use

- When you only need raw DNase1 accessibility magnitudes rather than a REM's relative contribution to gene expression — the z-scoring removes absolute signal level per REM.
- When the gene has only one associated REM: the Cell type score normalization over the set R of REMs is trivial/degenerate in that case.
- When working with TF ChIP-seq peak overlap or motif enrichment workflows (e.g. PASTAA on REM sequences), which use REM coordinates/sequences rather than standardized DNase1 signals.

## Inputs

- EpiRegio REST API GeneQuery response for a gene (e.g. ENSG00000223972) containing regressionCoefficient and per-cell-type, per-sample DNase1 signal per REM

## Outputs

- Per-REM, per-cell-type standardized DNase1 values (mean=0, sd=1 per REM across cell types)
- Cell type scores per REM in each cell type, written to cell_type_scores.csv with REM IDs, cell types, and Cell type score columns

## How to apply

Retrieve the REM table for a chosen gene via the EpiRegio REST API GeneQuery endpoint (e.g. https://epiregio.de/REST_API/GeneQuery/ENSG00000223972/) using the Python Requests package, which returns the regressionCoefficient and per-cell-type DNase1 signal for each REM. For each REM, average the DNase1 signal over all samples of each cell type as provided in the API output, then log-transform the DNase1 signals. Standardize the log-transformed values per REM across cell types (mean=0, standard deviation=1) so signals are comparable within each REM. The standardized DNase1(r,c) values are then combined with the StitchIt regression coefficient beta_r to compute the Cell type score: beta_r * DNase1(r,c) divided by the sum of absolute values of beta_ri * DNase1(ri,c) over all REMs R associated to the gene. Rationale: this normalization is defined over the set R of all REMs of a gene and quantifies each REM's relative contribution to its gene's expression in a specific cell type as predicted by the linear model.

## Related tools

- **EpiRegio** (Provides the REM table with regressionCoefficient and per-cell-type DNase1 signal via the GeneQuery REST API endpoint; input source for the standardization) — https://github.com/TeamRegio/EpiRegioDB
- **Requests** (Python package used to call the EpiRegio GeneQuery REST API and fetch the REM/DNase1 table programmatically)
- **STITCHIT** (Linear model underlying the regression coefficients (beta_r) that weight the standardized DNase1 signals in the Cell type score)

## Examples

```
import requests; r = requests.get('https://epiregio.de/REST_API/GeneQuery/ENSG00000223972/'); rem_table = r.()
```

## Evaluation signals

- Every computed Cell type score lies in the range [-1, 1].
- For each gene and each cell type, the absolute Cell type scores of all REMs of that gene sum to 1 (within floating-point tolerance).
- Per REM across cell types, the standardized log-DNase1 values have mean 0 and standard deviation 1.
- The number of rows in cell_type_scores.csv matches the number of REMs returned by the GeneQuery endpoint times the number of cell types.
- Cell type scores match the values implied by beta_r * DNase1(r,c) / sum_i |beta_ri * DNase1(ri,c)| recomputed independently from the API output.

## Limitations

- The linear model used in STITCHIT, which forms the basis for assessing the Cell type score, is a simplification of the actual regulation of genes by their REMs.
- Standardization is per REM across cell types, so absolute DNase1 accessibility levels per REM are not preserved; scores only express relative contributions among the REMs of one gene.
- Averaging over samples of each cell type as provided in the API output discards within-cell-type sample variability.

## Evidence

- [methods] Compute the Cell type score for each REM r in cell type c as beta_r * DNase1(r,c) divided by the sum of absolute values of beta_ri * DNase1(ri,c) over all REMs R: [evidence span withheld — anchor status: not_found]
- [methods] Log-transform the DNase1 signals and standardize them per REM across cell types (mean=0, standard deviation=1), averaging over all samples of each cell type: [evidence span withheld — anchor status: not_found]
- [other] The Cell type score is computed over the set R of all REMs associated to a given gene and normalizes the contribution of REM r to its gene's expression in a specific cell type: [evidence span withheld — anchor status: not_found]
- [discussion] the linear model used in STITCHIT, which forms the basis for assessing the Cell type score, is a simplification to the actual regulation of genes by their REMs: "the linear model used in STITCHIT, which forms the basis for assessing the Cell type score, is a simplification to the actual regulation of genes by their REMs"
- [abstract] EpiRegio allows programmatic access through a REST API and is freely available at https://epiregio.de/.: "EpiRegio allows programmatic access through a REST API and is freely available at https://epiregio.de/."
