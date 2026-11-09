---
name: range-validation-of-computed-scores
description: 'Use when computing or consuming EpiRegio Cell type scores from GeneQuery
  REM tables and you must verify normalization: every score in [-1, 1] and absolute
  scores per gene per cell type summing to 1.'
when_to_use_negative:
- Input REM annotations are already pre-normalized Cell type scores (e.g. downloaded
  directly as scored output) rather than raw regression coefficients and DNase1 signals
- The goal is motif enrichment or TF target-gene identification (PASTAA, ChIP-seq
  peak overlap), not verifying score normalization
- REMs have no associated cell-type-resolved DNase1 signal, so the linear-model normalization
  cannot be computed
edam_operation: http://edamontology.org/operation_2428
tools:
- name: EpiRegio
  role: Source of the Cell type score definition, the GeneQuery REM table, and the
    REST API used to fetch regressionCoefficient and DNase1 signal values to validate
  repo: github.com/TeamRegio/EpiRegioDB
- name: Requests
  role: Python package used to retrieve the REM table for a chosen gene via the EpiRegio
    REST API GeneQuery endpoint
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkaa382
    title: 'EpiRegio: analysis and retrieval of regulatory elements linked to genes'
  source_content_hash: sha256:f4c261d63f3351acf63e7763153e0d5e99fe89814838b251fb6caace50f43af8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EpiRegio
  - Requests
  edam_operation: http://edamontology.org/operation_2428
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# range-validation-of-computed-scores

## Summary

Validates that EpiRegio Cell type scores, computed as normalized per-REM contributions (regressionCoefficient beta_r * standardized DNase1 signal) to a target gene's expression, fall within [-1, 1] and that absolute scores of all REMs of a gene sum to 1 per cell type.

## When to use

Use when computing or consuming EpiRegio Cell type scores from GeneQuery REM tables and you must verify normalization: every score in [-1, 1] and absolute scores per gene per cell type summing to 1.

## When NOT to use

- Input REM annotations are already pre-normalized Cell type scores (e.g. downloaded directly as scored output) rather than raw regression coefficients and DNase1 signals
- The goal is motif enrichment or TF target-gene identification (PASTAA, ChIP-seq peak overlap), not verifying score normalization
- REMs have no associated cell-type-resolved DNase1 signal, so the linear-model normalization cannot be computed

## Inputs

- EpiRegio REST API GeneQuery REM table for a gene (e.g. https://epiregio.de/REST_API/GeneQuery/ENSG00000223972/) containing regressionCoefficient and per-cell-type DNase1 signal columns
- List of cell types with per-cell-type DNase1 signal samples per REM

## Outputs

- cell_type_scores.csv with REM IDs, cell types, and Cell type scores
- Validation report confirming scores are within [-1, 1] and absolute scores sum to 1 per gene per cell type

## How to apply

Retrieve the REM table (regressionCoefficient and per-cell-type DNase1 signal) for a gene via EpiRegio's REST API GeneQuery endpoint using the Python Requests package. Log-transform the DNase1 signals and standardize them per REM across cell types (mean=0, standard deviation=1), averaging over all samples of each cell type. Compute the Cell type score for each REM r in cell type c as beta_r * DNase1(r,c) divided by the sum of absolute values of beta_ri * DNase1(ri,c) over all REMs R associated to the gene — this division is what guarantees the [-1, 1] range. Then assert two invariants: every score lies in [-1, 1] (floating-point tolerance allowed) and the absolute Cell type scores of all REMs of one gene sum to 1 for each cell type. Write REM IDs, cell types, and scores to cell_type_scores.csv only after both checks pass.

## Related tools

- **EpiRegio** (Source of the Cell type score definition, the GeneQuery REM table, and the REST API used to fetch regressionCoefficient and DNase1 signal values to validate) — github.com/TeamRegio/EpiRegioDB
- **Requests** (Python package used to retrieve the REM table for a chosen gene via the EpiRegio REST API GeneQuery endpoint)

## Examples

```
score = beta_r * dnase1_rc / sum(abs(b_ri * d_ric) for ri in R); assert all(-1 <= s <= 1 for s in scores) and abs(sum(map(abs, scores)) - 1) < 1e-9
```

## Evaluation signals

- All Cell type score values in cell_type_scores.csv are within [-1, 1] (floating-point tolerance)
- The sum of absolute Cell type scores per gene per cell type equals 1 within floating-point tolerance
- Each computed score equals beta_r * DNase1(r,c) divided by the sum of absolute values of beta_ri * DNase1(ri,c) over all REMs R of the gene
- DNase1 signals were log-transformed and standardized per REM across cell types with mean=0 and standard deviation=1 before scoring
- Output CSV schema contains exactly REM ID, cell type, and Cell type score columns

## Limitations

- The linear model used in STITCHIT, which forms the basis for assessing the Cell type score, is a simplification of the actual regulation of genes by their REMs
- Range and sum-to-1 checks pass within floating-point tolerance only, so exact equality assertions will fail

## Evidence

- [discussion] the linear model used in STITCHIT, which forms the basis for assessing the Cell type score, is a simplification to the actual regulation of genes by their REMs: "the linear model used in STITCHIT, which forms the basis for assessing the Cell type score, is a simplification to the actual regulation of genes by their REMs"
- [intro] EpiRegio uses the StitchIt algorithm which interprets chromatin accessibility data with respect to variation in gene expression: "EpiRegio uses the StitchIt algorithm which interprets chromatin accessibility data with respect to variation in gene expression"
- [abstract] Our web server allows the analysis of genes and their associated REMs, including the REM’s activity and its estimated cell type-specific contribution to its target gene’s expression.: "Our web server allows the analysis of genes and their associated REMs, including the REM’s activity and its estimated cell type-specific contribution"
- [abstract] EpiRegio allows programmatic access through a REST API and is freely available at https://epiregio.de/.: "EpiRegio allows programmatic access through a REST API and is freely available at https://epiregio.de/."
- [readme] EpiRegio is a resource of Regulatory EleMents (REMs) associated to protein coding, and non-protein coding genes.: "EpiRegio is a resource of Regulatory EleMents (REMs) associated to protein coding, and non-protein coding genes."
