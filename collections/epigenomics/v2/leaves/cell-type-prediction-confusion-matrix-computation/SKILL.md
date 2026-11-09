---
name: cell-type-prediction-confusion-matrix-computation
description: Use when you have Cellcano (or similar supervised) predicted cell-type
  labels for a target scATAC-seq dataset plus ground-truth labels (e.g., FACS-sorted
  FACS labels embedded in cell identifiers) and need to quantify per-class prediction
  accuracy.
when_to_use_negative:
- Target cells carry no ground-truth labels (labels are not embedded in identifiers
  and no FACS/reference annotation exists), so no true-vs-predicted cross-tabulation
  is possible
- Predictions come from unsupervised clustering rather than a supervised transfer,
  so predicted label names are not directly comparable to the reference label vocabulary
  (B, CD4 T, CD8 T, Mono, NK)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Cellcano
  role: 'Produces the predicted cell-type labels (via two-round supervised learning:
    MLP plus self-Knowledge Distiller) whose accuracy the confusion matrix evaluates
    on the human PBMC target data'
  repo: https://marvinquiet.github.io/Cellcano/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-37439-3
    title: 'Cellcano: supervised cell type identification for single cell ATAC-seq
      data'
  source_content_hash: sha256:ec78ca255a2775cf9147bb74e2ea8daffa379b05045f30509923970c4d6bcf81
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cellcano
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# cell-type-prediction-confusion-matrix-computation

## Summary

Computes a true-vs-predicted cross-tabulation (confusion matrix) of cell-type labels to quantify prediction accuracy of a supervised celltyping method such as Cellcano, using ground-truth labels embedded in target cell identifiers. In the Cellcano quick-start demo, diagonal counts against FACS ground truth serve as the validation statistic.

## When to use

Use when you have Cellcano (or similar supervised) predicted cell-type labels for a target scATAC-seq dataset plus ground-truth labels (e.g., FACS-sorted FACS labels embedded in cell identifiers) and need to quantify per-class prediction accuracy.

## When NOT to use

- Target cells carry no ground-truth labels (labels are not embedded in identifiers and no FACS/reference annotation exists), so no true-vs-predicted cross-tabulation is possible
- Predictions come from unsupervised clustering rather than a supervised transfer, so predicted label names are not directly comparable to the reference label vocabulary (B, CD4 T, CD8 T, Mono, NK)

## Inputs

- Cellcano quick-start prediction output for the human PBMC target scATAC-seq dataset (cell identifiers embedding FACS ground-truth labels B, CD4 T, CD8 T, Mono, NK)
- Cellcano final predicted cell-type label per target cell

## Outputs

- 5x5 confusion matrix (pandas.crosstab, rows ordered B, CD4, CD8, Mono, NK) of true FACS vs predicted labels
- confusion_matrix.csv with per-class diagonal counts

## How to apply

Load the Cellcano quick-start prediction output for the human PBMC target dataset, where the ground-truth FACS labels (B, CD4 T, CD8 T, Mono, NK) are embedded in each cell identifier. Parse each identifier via string splitting to recover the true FACS label and pair it with Cellcano's final predicted label (produced by the two-round pipeline: MLP on reference gene scores, then the self-Knowledge Distiller on anchor cells selected by low prediction entropy). Build a 5x5 cross-tabulation of true vs predicted labels with pandas.crosstab, ordering rows B, CD4, CD8, Mono, NK, and write the result to confusion_matrix.csv. Judge correctness by checking that the diagonal counts approximately match the quick-start-reported values: B cells 1022, CD4 T cells 1084, CD8 T cells 899, Monocytes 637, and NK cells 637.

## Related tools

- **Cellcano** (Produces the predicted cell-type labels (via two-round supervised learning: MLP plus self-Knowledge Distiller) whose accuracy the confusion matrix evaluates on the human PBMC target data) — https://marvinquiet.github.io/Cellcano/

## Examples

```
import pandas as pd; true = [c.split('_')[1] for c in cells]; cm = pd.crosstab(pd.Series(true, name='true'), pd.Series(pred, name='pred')).reindex(index=['B','CD4','CD8','Mono','NK']); cm.to_csv('confusion_matrix.csv')
```

## Evaluation signals

- Diagonal counts approximately match the quick-start-reported values: B cells 1022, CD4 T cells 1084, CD8 T cells 899, Monocytes 637, NK cells 637
- Confusion matrix is exactly 5x5 with rows and columns ordered B, CD4, CD8, Mono, NK
- Every target cell appears exactly once (row sums equal the number of cells per true FACS label; grand total equals number of target cells)
- confusion_matrix.csv is written and parses back to the same cross-tabulation values
- Off-diagonal (misclassified) cells are a small fraction, consistent with the claim that most target cells are correctly predicted against FACS ground truth

## Limitations

- Cellcano's ground-truth comparison relies on the FACS labels being embedded in the target cell identifiers; without such embedded or external truth labels this validation cannot be performed
- Off-diagonal structure is not benchmarked to a fixed threshold in the demo — only diagonal counts have reported reference values, so 'approximately match' must be judged by the curator's tolerance
- Cellcano was benchmarked on human PBMCs and mouse brain systems; the reported diagonal counts are specific to the quick-start PBMC target dataset and will not generalize to other targets

## Evidence

- [results] quick-start demo's confusion matrix reports diagonal counts of B cells 1022, CD4 T cells 1084, CD8 T cells 899, Monocytes 637, and NK cells 637: [evidence span withheld — anchor status: not_found]
- [abstract] two-round supervised learning algorithm: "Here we develop Cellcano, a computational method based on a two-round supervised learning algorithm to identify cell types from scATAC-seq data."
- [readme] Cellcano is well-documented and freely available: "Cellcano is an open source software for supervised cell type identification (celltyping) in scATAC-seq data, published in [Nature Communications](https://doi.org/10.1038/s41467-023-37439-3)."
- [intro] anchors selected by lower prediction entropies: "we first calculate entropy for each cell based on the prediction probabilities from the first-round prediction and then select cells with lower entropies as anchors"
