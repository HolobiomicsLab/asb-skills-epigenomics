---
name: auroc-performance-evaluation-of-enhancer-classifiers
description: Use when a trained motif-count enhancer classifier (e.g., BagOfMotifs/XGBoost
  on FIMO-scanned CREs) needs quantitative performance assessment via auROC, or when
  verifying that models generalize across species or cell types.
when_to_use_negative:
- When input is already a feature table with labels other than cell-type/condition
  (BOM models are built for cis-regulatory cell-state classification, not arbitrary
  tabular prediction).
- When evaluating models that rely on motif grammar (order, orientation, spacing)
  — BOM deliberately discards these and its auROC cannot validate grammar-aware approaches.
- When regulatory activity depends on features outside the input sequence (open chromatin
  status, distal histone marks) — the article notes BOM is constrained to cis-regulatory
  information within the input sequence.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_0654
tools:
- name: Bag-of-Motifs (BOM)
  role: Provides predict_binary_multi() for multiclass prediction and per-cell-type
    auROC performance evaluation of the motif-count XGBoost classifiers.
  repo: github.com/ewonglab/BOM_package
- name: FIMO
  role: Produces the motif hit annotations (gimme.vertebrate.v5.0.meme library) that
    are aggregated into the unordered motif-count features whose classifier performance
    is scored by auROC.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-65362-2
    title: Motif-based models accurately predict cell type-specific distal regulatory
      elements
  source_content_hash: sha256:ded812110a1d7472f546b2d7870fc81ad4d81d7dc1333362567eb23e5c49eb39
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BSgenome.Mmusculus.UCSC.mm10
  - Bag-of-Motifs (BOM)
  - FIMO
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# auroc-performance-evaluation-of-enhancer-classifiers

## Summary

Evaluates cell-type-specific cis-regulatory element classifiers built from unordered TF motif counts with gradient-boosted trees (BOM/XGBoost) using area under the ROC curve (auROC), including multiclass cross-species generalization checks.

## When to use

Use when a trained motif-count enhancer classifier (e.g., BagOfMotifs/XGBoost on FIMO-scanned CREs) needs quantitative performance assessment via auROC, or when verifying that models generalize across species or cell types.

## When NOT to use

- When input is already a feature table with labels other than cell-type/condition (BOM models are built for cis-regulatory cell-state classification, not arbitrary tabular prediction).
- When evaluating models that rely on motif grammar (order, orientation, spacing) — BOM deliberately discards these and its auROC cannot validate grammar-aware approaches.
- When regulatory activity depends on features outside the input sequence (open chromatin status, distal histone marks) — the article notes BOM is constrained to cis-regulatory information within the input sequence.

## Inputs

- Trained BagOfMotifs gradient-boosted-tree models (output of BagOfMotifs::binModel())
- TF motif count matrices from FIMO scans of CRE FASTA sequences against gimme.vertebrate.v5.0.meme
- Held-out or cross-species test CREs (condition-annotated four-column BED: chromosome, start, stop, condition)

## Outputs

- Multiclass predictions over cell types
- Per-cell-type auROC values
- Mean auROC summary (benchmark: 0.90 human, 0.85 mouse heart CREs)

## How to apply

After training gradient-boosted-tree models on unordered TF motif count vectors (from FIMO scans against gimme.vertebrate.v5.0.meme), run BagOfMotifs::predict_binary_multi() to obtain multiclass predictions and per-cell-type auROC values. For the cross-species heart task, compare mean auROC against the reported benchmarks of 0.90 on human CREs and 0.85 on mouse CREs across seven shared adult heart cell types. Use auROC per cell type to identify which classes drive performance — ventricular cardiomyocytes showed the highest concordance (89%) in the SHAP-aggregated cross-species cell assignment. Because BOM outperforms deep-learning baselines with fewer parameters, auROC from this simple representation is a legitimate performance ceiling check rather than a lower-bound sanity check.

## Related tools

- **Bag-of-Motifs (BOM)** (Provides predict_binary_multi() for multiclass prediction and per-cell-type auROC performance evaluation of the motif-count XGBoost classifiers.) — github.com/ewonglab/BOM_package
- **FIMO** (Produces the motif hit annotations (gimme.vertebrate.v5.0.meme library) that are aggregated into the unordered motif-count features whose classifier performance is scored by auROC.)

## Examples

```
BagOfMotifs::predict_binary_multi()
```

## Evaluation signals

- Mean auROC across the seven shared adult heart cell types should approach the reported 0.90 on human CREs and 0.85 on mouse CREs.
- Per-cell-type auROC values must be reported for every class, not only the global mean, to localize strong/weak classes (ventricular cardiomyocytes were highest at 89% concordance).
- Predictions must come from predict_binary_multi() applied to the same models trained by binModel() — retraining before evaluation invalidates the comparison.
- SHAP-aggregated cross-species cell assignment rates can be cross-checked: 83% overall (46,223/55,834 human cells) should accompany auROC benchmarks.
- Motif counts must derive from FIMO >= 5.5.4 default parser settings (set FIMO_updated = FALSE for older FIMO versions) before model evaluation.

## Limitations

- BOM is constrained to cis-regulatory information within the input sequence; open chromatin status, histone marks, and enhancer activity can be influenced by regions beyond the sequence length used in BOM.
- Reliance on known TF motifs neglects informative sequences not captured by current motif annotations, and TFs do not bind all their motifs.
- Unknown how much of the remaining genome regions may share similar motif composition yet not share regulatory activity, so high auROC does not guarantee regulatory activity of all high-scoring regions.
- Performance of competing sequence-based deep learning methods is likely linked to the limited amount of data available to train the large number of cell states, which can bias comparisons.

## Evidence

- [abstract] This minimalist representation, combined with gradient-boosted trees, enables the accurate prediction of cell-type-specific enhancers across mouse, human, zebrafish, and Arabidopsis datasets.: "This minimalist representation, combined with gradient-boosted trees, enables the accurate prediction of cell-type-specific enhancers across mouse, human, zebrafish"
- [abstract] Despite its simplicity, BOM outperforms more complex deep-learning models while using fewer parameters.: "Despite its simplicity, BOM outperforms more complex deep-learning models while using fewer parameters."
- [readme] # Prediction and performance
BagOfMotifs::predict_binary_multi(): "# Prediction and performance
BagOfMotifs::predict_binary_multi()"
- [readme] BOM is an R package that uses XGBoost to construct predictive models of cis-regulatory sequences between cell states.: "BAG-of-Motifs (BOM) is an R package that uses XGBoost to construct predictive models of cis-regulatory sequences between cell states."
- [discussion] BOM is constrained to cis-regulatory information within the input sequence.: "BOM is constrained to cis-regulatory information within the input sequence."
