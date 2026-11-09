---
name: deep-learning-enhancer-classification
description: Use when comparing a motif-count classifier against deep-learning baselines
  for cell-type-specific distal cis-regulatory element prediction, or when establishing
  whether a simple motif representation suffices for enhancer classification.
when_to_use_negative:
- When the goal is motif interpretability or SHAP-based motif importance from a trained
  BOM model rather than baseline model comparison
- When the task requires modeling chromatin context beyond the input sequence, since
  BOM is constrained to cis-regulatory information within the input sequence
- When input data are already feature/count tables rather than genomic interval BED
  coordinates requiring FASTA generation
edam_operation: http://edamontology.org/operation_2995
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0654
tools:
- name: LS-GKM
  role: K-mer-based SVM baseline trained with default gkm-SVM kernel settings on the
    same binary cell-type-specific CRE labels and splits
- name: DNABERT
  role: Pretrained transformer baseline fine-tuned with 6-mer (k=6) tokenization on
    identical train/test partitions
- name: Enformer
  role: Deep-learning baseline fine-tuned with an adaptive head of two fully connected
    linear layers, adjusted CRE-focused cropping, and hyperparameter exploration
- name: Bag-of-Motifs (BOM)
  role: Reference model whose motif-count/XGBoost predictions are compared against
    all deep-learning baselines
  repo: https://github.com/ewonglab/BOM_package
- name: gkmSVM
  role: K-mer-based classifier family underlying the LS-GKM baseline
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-65362-2
    title: Motif-based models accurately predict cell type-specific distal regulatory
      elements
  source_content_hash: sha256:d48bd860fb3df068758218be5059bcf62d2cefba82f56c0d100d4c968b639d81
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
  edam_operation: http://edamontology.org/operation_2995
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# deep-learning-enhancer-classification

## Summary

Benchmarks sequence-based classifiers (LS-GKM, DNABERT, Enformer, CNN/CNN+LSTM/Basset variants) against BOM on cell-type-specific distal CRE binary classification, using shared FASTA training/testing partitions from mouse E8.25 data and auPR/MCC metrics.

## When to use

Use when comparing a motif-count classifier against deep-learning baselines for cell-type-specific distal cis-regulatory element prediction, or when establishing whether a simple motif representation suffices for enhancer classification.

## When NOT to use

- When the goal is motif interpretability or SHAP-based motif importance from a trained BOM model rather than baseline model comparison
- When the task requires modeling chromatin context beyond the input sequence, since BOM is constrained to cis-regulatory information within the input sequence
- When input data are already feature/count tables rather than genomic interval BED coordinates requiring FASTA generation

## Inputs

- Four-column BED file (chromosome, start, stop, condition) of distal CREs per cell type
- Mouse mm10 genome (BSgenome.Mmusculus.UCSC.mm10) for FASTA extraction
- Pretrained DNABERT model for k=6 6-mer tokenization fine-tuning
- Pretrained Enformer model for head adaptation and fine-tuning
- gimme.vertebrate.v5.0.meme motif library for BOM motif scanning

## Outputs

- Pairwise binary classification auPR and MCC values for each model across the 17 E8.25 cell types
- Relative improvement percentages of BOM over LS-GKM, DNABERT, and Enformer (auPR: 17.2%/55.1%/10.3%; MCC: 77.5%/211.9%/33.4%)
- CNN multiclass recall values (expected 0.0-0.5 vs BOM's 0.88)

## How to apply

Generate identical sequences for all models from a four-column BED input (chromosome, start, stop, condition) using generateAllFasta with genome = 'Mmusculus' (mm10), then train LS-GKM with default gkm-SVM kernel settings, fine-tune pretrained DNABERT with 6-mer tokenization (k=6), and fine-tune Enformer with an adaptive head of two fully connected linear layers, adjusted input cropping focused on each CRE, and hyperparameter exploration. Train CNN, CNN+LSTM, and Basset architectures on one-hot-encoded sequences and labels from the same partitions. Evaluate every model with auPR and MCC on the held-out set and compute relative improvements; the reference result is BOM outperforming LS-GKM, DNABERT, and Enformer by 17.2%, 55.1%, and 10.3% auPR and 77.5%, 211.9%, and 33.4% MCC, with CNN multiclass recall at 0.0-0.5 versus BOM's 0.88.

## Related tools

- **LS-GKM** (K-mer-based SVM baseline trained with default gkm-SVM kernel settings on the same binary cell-type-specific CRE labels and splits)
- **DNABERT** (Pretrained transformer baseline fine-tuned with 6-mer (k=6) tokenization on identical train/test partitions)
- **Enformer** (Deep-learning baseline fine-tuned with an adaptive head of two fully connected linear layers, adjusted CRE-focused cropping, and hyperparameter exploration)
- **Bag-of-Motifs (BOM)** (Reference model whose motif-count/XGBoost predictions are compared against all deep-learning baselines) — https://github.com/ewonglab/BOM_package
- **gkmSVM** (K-mer-based classifier family underlying the LS-GKM baseline)

## Examples

```
library(BagOfMotifs); generateAllFasta(bedDir = "path/to/input", genome = "Mmusculus"); BagOfMotifs::binModel(); BagOfMotifs::predict_binary_multi()
```

## Evaluation signals

- Verify computed auPR/MCC values reproduce the reported relative-improvement percentages over LS-GKM (17.2% auPR, 77.5% MCC), DNABERT (55.1% auPR, 211.9% MCC), and Enformer (10.3% auPR, 33.4% MCC)
- Check CNN multiclass recall falls in the 0.0-0.5 range against BOM's 0.88
- Confirm all models were trained and tested on identical FASTA sequences generated from the same BED input with genome = 'Mmusculus' (mm10)
- Confirm baseline evaluations use held-out test partitions rather than training data

## Limitations

- BOM is constrained to cis-regulatory information within the input sequence; open chromatin status, histone marks, and enhancer activity can be influenced by regions beyond the sequence length used
- Reliance on known TF motifs neglects informative sequences not captured by current motif annotations, and TFs do not bind all their motifs
- Performance of sequence-based deep learning methods is likely linked to the limited amount of data available to train the large number of cell states
- Unknown how much of the remaining genome regions may share similar motif composition yet not share regulatory activity

## Evidence

- [results] In pairwise binary classification across the 17 E8.25 cell types, BOM outperformed LS-GKM, DNABERT, and Enformer by 17.2%, 55.1%, and 10.3% in auPR and by 77.5%, 211.9%, and 33.4% in MCC, respectively.: [evidence span withheld — anchor status: not_found]
- [discussion] We introduced an adaptive layer, consisting of two fully connected linear layers tailored to the new data and cell-type contexts.: "We introduced an adaptive layer, consisting of two fully connected linear layers tailored to the new data and cell-type contexts."
- [discussion] We adjusted the cropping to focus precisely on the CRE of interest, thereby reducing noise to enhance the model’s precision.: "We adjusted the cropping to focus precisely on the CRE of interest, thereby reducing noise to enhance the model’s precision."
- [readme] generateAllFasta(bedDir = "path/to/input", genome = "Mmusculus"): "generateAllFasta(bedDir = "path/to/input", genome = "Mmusculus")"
- [discussion] BOM is constrained to cis-regulatory information within the input sequence.: "BOM is constrained to cis-regulatory information within the input sequence."
