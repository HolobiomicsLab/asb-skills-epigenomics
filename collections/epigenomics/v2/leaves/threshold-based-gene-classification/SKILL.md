---
name: threshold-based-gene-classification
description: Use when you have CITRUS 'attn_wt' attention weights plus a gene list
  and need to binarize genes by attention magnitude to test driver-gene enrichment
  (e.g., against IntOGen).
when_to_use_negative:
- You need continuous attention magnitudes rather than a binary high/low grouping
  — the log(attention+1) >= 2 cutoff discards within-group ordering
- Your model output lacks the 'attn_wt' key (e.g., you only have TF activity 'hid_tmr'
  or embeddings), so no per-gene attention thresholding is possible
- Testing enrichment against a gene set other than curated drivers where a 2x2 Fisher's
  exact contingency framing does not apply
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_3474
tools:
- name: CITRUS
  role: Produces the self-attention weights ('attn_wt') over somatic alterations that
    are thresholded here
  repo: https://github.com/osmanbeyoglulab/CITRUS
- name: SciPy
  role: scipy.stats.fisher_exact computes the odds ratio and p-value on the 2x2 driver-by-attention
    contingency table
- name: Pandas
  role: Loads the CITRUS output pickle and writes attention_driver_enrichment.csv
- name: Numpy
  role: Computes the log(attention+1) transform and threshold comparison on the attention-weight
    array
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkac881
    title: Interpretable deep learning for chromatin-informed inference of transcriptional
      programs driven by somatic alterations across cancers
  source_content_hash: sha256:b2225a41c12a44ff7e6602333ecd7ddb4454885bf11fbfe492085a3c245a1f42
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/osmanbeyoglulab/CITRUS
  tools:
  - CITRUS
  - Numpy
  - Pandas
  - Python
  - SciPy
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# threshold-based-gene-classification

## Summary

Classifies genes into high- vs low-attention groups using a log(attention+1) >= 2 cutoff on CITRUS self-attention weights, then tests enrichment of IntOGen known cancer drivers via Fisher's exact test (pan-cancer P = 4.48e-41).

## When to use

Use when you have CITRUS 'attn_wt' attention weights plus a gene list and need to binarize genes by attention magnitude to test driver-gene enrichment (e.g., against IntOGen).

## When NOT to use

- You need continuous attention magnitudes rather than a binary high/low grouping — the log(attention+1) >= 2 cutoff discards within-group ordering
- Your model output lacks the 'attn_wt' key (e.g., you only have TF activity 'hid_tmr' or embeddings), so no per-gene attention thresholding is possible
- Testing enrichment against a gene set other than curated drivers where a 2x2 Fisher's exact contingency framing does not apply

## Inputs

- CITRUS output pickle file containing the 'attn_wt' attention-weight array and gene list (e.g., output_dataset_CITRUS.pkl)
- IntOGen pan-cancer known cancer driver gene list
- SGA binary somatic alteration matrix (11998 genes) for alteration-frequency ranking

## Outputs

- attention_driver_enrichment.csv with Fisher's exact test odds ratio, p-value, and 2x2 contingency table
- High/low-attention membership labels for the top 100 most frequently mutated genes

## How to apply

Load the CITRUS output pickle with pandas/pickle and extract the 'attn_wt' attention-weight array and gene list from the output dictionary. Transform each gene's attention weight as log(attention+1) and assign genes with log(attention+1) >= 2 to the high-attention group, the rest to the low-attention group. Build a 2x2 contingency table of IntOGen driver vs non-driver status by attention group and run scipy.stats.fisher_exact; the pan-cancer result should reproduce P = 4.48e-41. As a companion analysis, rank genes by somatic alteration frequency from the SGA binary matrix, take the top 100 most frequently mutated genes, and label each with the same log(attention+1) >= 2 threshold. Write odds ratio, p-value, contingency table, and top-100 membership to attention_driver_enrichment.csv.

## Related tools

- **CITRUS** (Produces the self-attention weights ('attn_wt') over somatic alterations that are thresholded here) — https://github.com/osmanbeyoglulab/CITRUS
- **SciPy** (scipy.stats.fisher_exact computes the odds ratio and p-value on the 2x2 driver-by-attention contingency table)
- **Pandas** (Loads the CITRUS output pickle and writes attention_driver_enrichment.csv)
- **Numpy** (Computes the log(attention+1) transform and threshold comparison on the attention-weight array)

## Examples

```
odds, p = scipy.stats.fisher_exact(table); high = np.log(np.array(attn_wt)+1) >= 2
```

## Evaluation signals

- Reproduced Fisher's exact test p-value matches the reported pan-cancer enrichment value of 4.48e-41
- 2x2 contingency table rows/columns sum correctly: high-attention + low-attention counts equal the total gene list size
- Every gene in the top-100 SGA-frequency ranking receives exactly one high/low attention label with no missing values
- attention_driver_enrichment.csv contains odds ratio, p-value, and contingency table fields
- Threshold check: all genes labeled high-attention satisfy log(attn_wt+1) >= 2 exactly

## Limitations

- The log(attention+1) >= 2 cutoff is a fixed heuristic; applying it to datasets trained with different hyperparameters may not reproduce the reported enrichment
- CITRUS attention weights are specific to the self-attention encoder over the sparse SGA index-list encoding; other model outputs (e.g., TF activity) cannot be substituted
- Fisher's exact test assumes independent gene classifications; correlated alterations or shared driver pathways may inflate significance

## Evidence

- [other] Using Fisher's exact test with the log(attention + 1) >= 2 threshold, known cancer driver genes were verified as enriched in the highly attended group (P = 4.48 × 10−41): [evidence span withheld — anchor status: not_found]
- [abstract] Our approach employs a self-attention mechanism to model the contextual impact of somatic alterations.: "Our approach employs a self-attention mechanism to model the contextual impact of somatic alterations."
- [readme] SGA is originally a binary matrix containing 11998 genes with 1 as having a distinct mutation or copy number alteration and 0 as none of them.: "SGA is originally a binary matrix containing 11998 genes with 1 as having a distinct mutation or copy number alteration and 0 as none of them."
- [readme] attention_weight = output_data["attn_wt"]: "attention_weight = output_data["attn_wt"]"
