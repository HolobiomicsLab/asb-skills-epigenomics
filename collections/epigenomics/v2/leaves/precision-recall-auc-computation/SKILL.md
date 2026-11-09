---
name: precision-recall-auc-computation
description: Use when comparing loop-calling model performance on imbalanced Hi-C
  loop-detection tasks (positive loop labels are rare), especially across Hi-C sequencing-depth
  conditions or between dual-branch GILoop variants where mean PR-AUC (e.g., 0.3790
  vs 0.3718) is the reported metric.
when_to_use_negative:
- The metric of interest is loop coordinate concordance or spatial error rather than
  classification performance — use MAD (Mean Average Distance, Equations 14-16) instead
- Labels are balanced or per-class performance on multi-class data is needed, where
  AUROC or F1 per class is more informative than PR-AUC
- Single-cell Hi-C data is the input — the article notes GILoop 'is still not capable
  enough' for extremely sparse scHi-C, so PR-AUC comparisons across depths are out
  of scope
tools:
- name: GILoop
  role: Dual-branch (GCN + U-Net) loop-calling model whose per-branch predictions
    are scored with PR-AUC across sequencing depths
  repo: https://github.com/fzbio/GILoop
- name: SIP
  role: Prior image-based loop caller used as a state-of-the-art baseline whose detections
    would likewise be evaluated with precision-recall metrics
  repo: https://github.com/PouletAxel/SIP
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2022.105535
    title: 'GILoop: Robust chromatin loop calling across multiple sequencing depths
      on Hi-C data'
  source_content_hash: sha256:03bb847b1bfdbe093541832a8a65b7384b4724d41e39303ef6df598838cc82bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fzbio/GILoop
  tools:
  - GILoop
  - SIP
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# precision-recall-auc-computation

## Summary

Compute precision-recall AUC (PR-AUC) to score chromatin loop-calling predictions (e.g., GILoop GCN/U-Net branch scores for CTCF-mediated loops) against ground-truth loop labels, the primary metric used across sequencing depths and training regimes in GILoop.

## When to use

Use when comparing loop-calling model performance on imbalanced Hi-C loop-detection tasks (positive loop labels are rare), especially across Hi-C sequencing-depth conditions or between dual-branch GILoop variants where mean PR-AUC (e.g., 0.3790 vs 0.3718) is the reported metric.

## When NOT to use

- The metric of interest is loop coordinate concordance or spatial error rather than classification performance — use MAD (Mean Average Distance, Equations 14-16) instead
- Labels are balanced or per-class performance on multi-class data is needed, where AUROC or F1 per class is more informative than PR-AUC
- Single-cell Hi-C data is the input — the article notes GILoop 'is still not capable enough' for extremely sparse scHi-C, so PR-AUC comparisons across depths are out of scope

## Inputs

- Predicted loop scores per candidate pair from a GILoop branch (GCN or U-Net) on downsampled GM12878 Hi-C contact maps
- Ground-truth CTCF-mediated loop labels for held-out test chromosomes
- Sequencing-depth metadata for the seven downsampling levels (e.g., depth in reads/cis-contacts per level)

## Outputs

- Per-depth or per-condition PR-AUC values (e.g., gcn_depth_performance.csv, pr_auc_comparison.csv)
- Precision-recall curves plotted against sequencing depth (Figure 5-style trend)

## How to apply

Run the model (e.g., the GILoop GCN or U-Net branch) on held-out test chromosomes to score candidate CTCF-mediated loops, then compute PR-AUC from the predicted scores paired with ground-truth loop labels — precision-recall analysis is preferred here because true loops are a small fraction of candidate pairs. For depth-sweep experiments, compute PR-AUC separately at each of the seven downsampling levels generated with the GILoop repository's matrix downsampling function and assemble depth, PR-AUC (and MAD) values into a results table such as gcn_depth_performance.csv. For training-regime comparisons, compute the mean PR-AUC on held-out test chromosomes and compare against the baseline (e.g., mixed-density fine-tuned 0.3790 vs single-depth 90% 0.3718 per Table S11). Validate that the trend direction matches the paper's expectation: GCN PR-AUC rises as depth decreases (peaking around 260 million cis-contacts) while the U-Net branch falls.

## Related tools

- **GILoop** (Dual-branch (GCN + U-Net) loop-calling model whose per-branch predictions are scored with PR-AUC across sequencing depths) — https://github.com/fzbio/GILoop
- **SIP** (Prior image-based loop caller used as a state-of-the-art baseline whose detections would likewise be evaluated with precision-recall metrics) — https://github.com/PouletAxel/SIP

## Examples

```
from sklearn.metrics import average_precision_score; pr_auc = average_precision_score(y_true_labels, gcn_scores)
```

## Evaluation signals

- PR-AUC is computed strictly on held-out test chromosomes with ground-truth loop labels, not on training data
- Reproduced GCN-branch PR-AUC trend must rise monotonically (or near-monotonically) as Hi-C depth decreases, peaking near 260 million cis-contacts, while the U-Net branch shows the opposite trend
- Mixed-density fine-tuned model must exceed the single-depth baseline: mean PR-AUC 0.3790 vs 0.3718 (Table S11)
- Results table must contain one row per downsampling level with consistent depth, PR-AUC, and MAD columns before plotting the Figure 5 trend

## Limitations

- PR-AUC on this task is intrinsically modest (≈0.37–0.38 at best) because CTCF-mediated loop positives are rare among candidate pairs; treat absolute values as task-specific baselines rather than general-purpose scores
- Comparing PR-AUC across cell lines requires transfer-learning evaluation; the article reports performance decay is minimal but non-zero across cell lines
- Downsampling-based depth sweeps add substantial pre-processing overhead from matrix transformations and disk I/O

## Evidence

- [abstract] distinct preferences for matrix density: "We also uncover distinct preferences for matrix density by graph-based and image-based models"
- [abstract] depth-preference trend: "the model outperforms the state-of-the-art loop calling framework and is also more robust against low-quality Hi-C libraries"
- [readme] downsampling function availability: "In the GILoop repository, we provide a function for matrix downsampling at the training stage"
- [discussion] optimal density threshold: "which automatically downsamples the Hi-C contact map to the optimal density that we experimentally obtained in this study (260 million cis-contacts)"
