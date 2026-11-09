---
name: aupr-mcc-model-comparison
description: Use when evaluating whether a motif-count classifier (BOM) or any sequence
  model outperforms established baselines on the same binary/multiclass CRE classification
  task, using auPR and MCC.
when_to_use_negative:
- When models are trained or tested on different sequence sets or splits — the comparison
  requires all models to share identical FASTA sequences generated from the same BED
  file
- When the goal is motif interpretation rather than classifier benchmarking — use
  BOM's SHAP workflow (save_shap_multi, shapPlots_multi) instead
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3168
tools:
- name: Bag-of-Motifs (BOM)
  role: Motif-count classifier with XGBoost being benchmarked against deep-learning
    and k-mer baselines
  repo: https://github.com/ewonglab/BOM_package
- name: LS-GKM
  role: K-mer SVM baseline trained with default gkm-SVM kernel settings on the same
    labels and splits
- name: DNABERT
  role: Pretrained transformer baseline fine-tuned with 6-mer (k=6) tokenization on
    identical partitions
- name: Enformer
  role: Deep-learning baseline fine-tuned with a two-layer fully connected adaptive
    head and CRE-focused input cropping
- name: gkmSVM
  role: K-mer classifier family from which the LS-GKM baseline is derived
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-65362-2
    title: Motif-based models accurately predict cell type-specific distal regulatory
      elements
  source_content_hash: sha256:0994591a74f8abfbf1993027a4f1100dbe0f8792bd113fee3acc503220cdfacc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ewonglab/BOM_package
  tools:
  - Bag-of-Motifs (BOM)
  - DNABERT
  - Enformer
  - LS-GKM
  - gkmSVM
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# aupr-mcc-model-comparison

## Summary

A benchmarking protocol for comparing sequence-based classifiers of cell-type-specific distal CREs (BOM vs LS-GKM, DNABERT, Enformer, CNN/CNN+LSTM/Basset) on identical train/test splits, scored with auPR and MCC.

## When to use

Use when evaluating whether a motif-count classifier (BOM) or any sequence model outperforms established baselines on the same binary/multiclass CRE classification task, using auPR and MCC.

## When NOT to use

- When models are trained or tested on different sequence sets or splits — the comparison requires all models to share identical FASTA sequences generated from the same BED file
- When the goal is motif interpretation rather than classifier benchmarking — use BOM's SHAP workflow (save_shap_multi, shapPlots_multi) instead

## Inputs

- Mouse E8.25 input BED file with four columns (chromosome, start, stop, condition)
- FASTA sequences generated from the BED file using the mouse mm10 genome (BSgenome.Mmusculus.UCSC.mm10)
- Binary cell-type-specific CRE labels and train/test splits shared across all models
- One-hot-encoded sequences for CNN, CNN+LSTM, and Basset baselines

## Outputs

- auPR and MCC values for each model (LS-GKM, DNABERT, Enformer, CNN, CNN+LSTM, Basset, BOM) on the held-out set
- Pairwise binary classification results across the 17 E8.25 cell types
- Relative improvement percentages of BOM over each baseline (auPR and MCC)
- Multiclass recall values for CNN baselines vs BOM (e.g., CNN recall 0.0-0.5 vs BOM recall 0.88)

## How to apply

Generate FASTA sequences once from the mouse E8.25 input BED file (chromosome, start, stop, condition) with generateAllFasta and genome = 'Mmusculus' (mm10) so all models share identical training/testing sequences. Train LS-GKM with default gkm-SVM kernel settings, fine-tune pretrained DNABERT with 6-mer (k=6) tokenization, and fine-tune Enformer with an adaptive head of two fully connected linear layers, cropping input to focus precisely on each CRE and exploring the hyperparameter space. Train CNN, CNN+LSTM, and Basset on the same one-hot-encoded sequences and labels. Evaluate every model with auPR and MCC on the held-out set and compute BOM's relative improvement, which should reproduce the reported 17.2%/55.1%/10.3% auPR and 77.5%/211.9%/33.4% MCC gains over LS-GKM/DNABERT/Enformer.

## Related tools

- **Bag-of-Motifs (BOM)** (Motif-count classifier with XGBoost being benchmarked against deep-learning and k-mer baselines) — https://github.com/ewonglab/BOM_package
- **LS-GKM** (K-mer SVM baseline trained with default gkm-SVM kernel settings on the same labels and splits)
- **DNABERT** (Pretrained transformer baseline fine-tuned with 6-mer (k=6) tokenization on identical partitions)
- **Enformer** (Deep-learning baseline fine-tuned with a two-layer fully connected adaptive head and CRE-focused input cropping)
- **gkmSVM** (K-mer classifier family from which the LS-GKM baseline is derived)

## Examples

```
generateAllFasta(bedDir = "path/to/input", genome = "Mmusculus"); BagOfMotifs::binModel(); BagOfMotifs::predict_binary_multi()
```

## Evaluation signals

- Computed auPR/MCC relative improvements reproduce the reported values: 17.2%/55.1%/10.3% auPR and 77.5%/211.9%/33.4% MCC over LS-GKM/DNABERT/Enformer
- CNN multiclass recall falls in the 0.0-0.5 range, against BOM's recall of 0.88
- All models are evaluated on identical held-out FASTA sequences derived from the same mm10 BED input (generateAllFasta with genome = 'Mmusculus')
- Pairwise binary classification covers all 17 E8.25 cell types with shared binary cell-type-specific CRE labels
- MCC and auPR are both reported; auPR is preferred given class imbalance typical of CRE classification

## Limitations

- Performance of sequence-based deep learning methods is likely limited by the amount of training data available for many cell states
- Deep-learning baselines required non-trivial adaptation (adaptive head, input cropping, hyperparameter search) to be comparable on the CRE task
- Comparison is specific to the mouse E8.25 distal CRE task; results may not transfer to other assays or genomes

## Evidence

- [results] Pairwise binary classification improvements over LS-GKM, DNABERT, Enformer: [evidence span withheld — anchor status: not_found]
- [methods] Identical sequences for all models: [evidence span withheld — anchor status: not_found]
- [discussion] Enformer adaptation: "We introduced an adaptive layer, consisting of two fully connected linear layers tailored to the new data and cell-type contexts."
- [abstract] BOM outperforms deep-learning models: "Despite its simplicity, BOM outperforms more complex deep-learning models while using fewer parameters."
- [discussion] Deep-learning training data limitation: "the performance of sequence-based deep learning methods is likely linked to the limited amount of data available to train the large number of cell states"
