---
name: transformer-fine-tuning-genomic-sequences
description: Use when benchmarking a sequence-based classifier against pretrained
  DNA transformers on the same CRE train/test partitions, e.g. comparing BOM or k-mer
  SVMs to DNABERT/Enformer on mouse mm10 E8.25 distal regulatory elements.
when_to_use_negative:
- 'When unordered motif counts are sufficient: BOM outperformed DNABERT and Enformer
  by 55.1% and 10.3% auPR (211.9% and 33.4% MCC) on this task, so transformers may
  add cost without benefit for this data scale'
- When training data per cell state is small — the article notes performance of sequence-based
  deep learning methods is likely linked to the limited amount of data available for
  many cell states
- When the input is already a motif count/feature table rather than raw DNA sequence
tools:
- name: DNABERT
  role: Pretrained DNA transformer fine-tuned with 6-mer (k=6) tokenization as a sequence-based
    baseline on identical train/test partitions
- name: Enformer
  role: Pretrained transformer fine-tuned with an adaptive head of two fully connected
    linear layers, adjusted CRE-focused cropping, and hyperparameter exploration
- name: Bag-of-Motifs (BOM)
  role: Reference classifier whose auPR/MCC defines the relative-improvement benchmarks
    the transformer baselines are compared against
  repo: github.com/ewonglab/BOM_package
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-65362-2
    title: Motif-based models accurately predict cell type-specific distal regulatory
      elements
  source_content_hash: sha256:fd1c4a792f25a30bb212db6102aa7af8289a8a6d22a0c5a331fdca7715517dbf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bag-of-Motifs (BOM)
  - DNABERT
  - Enformer
  - LS-GKM
  - gkmSVM
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Transformer fine-tuning for genomic sequences

## Summary

Fine-tunes pretrained genomic transformers (DNABERT with 6-mer tokenization and Enformer with an adaptive two-layer head) as deep-learning baselines for binary cell-type-specific distal CRE classification on mouse E8.25 data.

## When to use

Use when benchmarking a sequence-based classifier against pretrained DNA transformers on the same CRE train/test partitions, e.g. comparing BOM or k-mer SVMs to DNABERT/Enformer on mouse mm10 E8.25 distal regulatory elements.

## When NOT to use

- When unordered motif counts are sufficient: BOM outperformed DNABERT and Enformer by 55.1% and 10.3% auPR (211.9% and 33.4% MCC) on this task, so transformers may add cost without benefit for this data scale
- When training data per cell state is small — the article notes performance of sequence-based deep learning methods is likely linked to the limited amount of data available for many cell states
- When the input is already a motif count/feature table rather than raw DNA sequence

## Inputs

- BED file with four columns: chromosome, start, stop, condition (mouse E8.25 distal CREs)
- Mouse mm10 genome (BSgenome.Mmusculus.UCSC.mm10) for FASTA extraction
- Pretrained DNABERT weights (6-mer tokenization) and pretrained Enformer weights

## Outputs

- Binary cell-type-specific CRE classification predictions on the held-out set for each transformer baseline
- auPR and MCC metrics per model, plus relative-improvement percentages versus BOM

## How to apply

First generate identical FASTA sequences for every model from the four-column mouse E8.25 BED file (chromosome, start, stop, condition) using generateAllFasta with genome = 'Mmusculus' (mm10) so all classifiers share identical training/testing sequences. Fine-tune DNABERT with 6-mer (k=6) tokenization on the identical train/test partitions. Fine-tune Enformer with an adaptive head consisting of two fully connected linear layers tailored to the new cell-type context, adjust the input cropping to focus precisely on each CRE of interest to reduce noise, and explore the hyperparameter space extensively. Train CNN, CNN+LSTM, and Basset variants on the same one-hot-encoded sequences for additional baselines. Evaluate all models on the held-out set with auPR and MCC, and compute BOM's relative improvement: 17.2%/55.1%/10.3% auPR and 77.5%/211.9%/33.4% MCC over LS-GKM/DNABERT/Enformer, with CNN multiclass recall (0.0-0.5) versus BOM recall 0.88 as a sanity check.

## Related tools

- **DNABERT** (Pretrained DNA transformer fine-tuned with 6-mer (k=6) tokenization as a sequence-based baseline on identical train/test partitions)
- **Enformer** (Pretrained transformer fine-tuned with an adaptive head of two fully connected linear layers, adjusted CRE-focused cropping, and hyperparameter exploration)
- **Bag-of-Motifs (BOM)** (Reference classifier whose auPR/MCC defines the relative-improvement benchmarks the transformer baselines are compared against) — github.com/ewonglab/BOM_package

## Examples

```
generateAllFasta(bedDir = "path/to/input", genome = "Mmusculus")
```

## Evaluation signals

- Computed auPR/MCC reproduce the reported relative improvements: 17.2%/55.1%/10.3% auPR and 77.5%/211.9%/33.4% MCC over LS-GKM/DNABERT/Enformer
- All models (transformers and motif baselines) are trained and tested on identical FASTA sequences generated from the same E8.25 BED file with mm10
- DNABERT uses 6-mer tokenization; Enformer uses an adaptive two-fully-connected-layer head with CRE-focused cropping
- CNN multiclass recall falls in the 0.0-0.5 range against BOM's recall of 0.88
- Evaluation uses both auPR and MCC on the held-out set for every baseline

## Limitations

- BOM outperformed the transformer baselines (DNABERT by 55.1% auPR / 211.9% MCC; Enformer by 10.3% auPR / 33.4% MCC) despite using fewer parameters
- Performance of sequence-based deep learning methods is likely limited by the amount of training data available for many cell states
- Open chromatin status, histone marks, and enhancer activity can be influenced by regions beyond the sequence length used in the models

## Evidence

- [discussion] We introduced an adaptive layer, consisting of two fully connected linear layers tailored to the new data and cell-type contexts.: "We introduced an adaptive layer, consisting of two fully connected linear layers tailored to the new data and cell-type contexts."
- [discussion] We adjusted the cropping to focus precisely on the CRE of interest to reduce noise.: "We adjusted the cropping to focus precisely on the CRE of interest, thereby reducing noise to enhance the model’s precision."
- [discussion] We also performed extensive exploration of the hyperparameter space to improve Enformer's accuracy.: "We also performed extensive exploration of the hyperparameter space to improve Enformer’s accuracy."
- [other] BOM outperformed LS-GKM, DNABERT, and Enformer by 17.2%, 55.1%, and 10.3% in auPR and by 77.5%, 211.9%, and 33.4% in MCC, respectively.: [evidence span withheld — anchor status: not_found]
