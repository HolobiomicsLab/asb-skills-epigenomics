---
name: auroc-performance-comparison-across-datasets
description: Use when you need to test whether a motif-count feature category (e.g.,
  overlapping vs. non-overlapping FIMO annotations) is biologically informative by
  retraining BagOfMotifs models and comparing auROC before and after the ablation.
when_to_use_negative:
- Input is already a per-sequence feature table or model predictions — the ablation
  requires regenerating motif counts from raw CRE coordinates via FIMO.
- The goal is de novo motif discovery rather than quantifying the predictive contribution
  of existing (gimme.vertebrate.v5.0.meme) motif annotations.
- The sequences of interest contain regulatory information beyond the input sequence
  (e.g., distal chromatin context), since BOM is constrained to cis-regulatory information
  within the input sequence.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3169
tools:
- name: FIMO
  role: Scans transcription factor motifs in CRE sequences at varying q-value cutoffs
    (0.1, 0.3, 0.5) to generate motif annotations used for the ablation
  repo: https://meme-suite.org/meme/doc/install.html
- name: BagOfMotifs (BOM)
  role: R package providing the motif-count representation, XGBoost model training,
    and binary/multi-class prediction used for the auROC comparison
  repo: https://github.com/ewonglab/BOM_package
- name: binModel
  role: Trains the gradient-boosted tree classifiers on the per-sequence motif count
    features after the ablation
  repo: https://github.com/ewonglab/BOM_package
- name: predict_binary_multi
  role: Evaluates cell-type-specific prediction performance (auROC) of the retrained
    models
  repo: https://github.com/ewonglab/BOM_package
- name: BSgenome.Mmusculus.UCSC.mm10
  role: Provides the mouse mm10 genome sequence used to extract CRE sequences for
    motif scanning
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-65362-2
    title: Motif-based models accurately predict cell type-specific distal regulatory
      elements
  source_content_hash: sha256:16f6f4d5dfcd5984862d6e950f6e7ca05db2d39550480a1bf1bb943388527150
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BSgenome.Mmusculus.UCSC.mm10
  - BagOfMotifs
  - BagOfMotifs (BOM)
  - FIMO
  - binModel
  - predict_binary_multi
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# auroc-performance-comparison-across-datasets

## Summary

Retrain BOM's gradient-boosted tree classifiers under an ablation (e.g., removing overlapping motif annotations from FIMO-based motif counts) and compare the resulting auROC values against the original models to quantify how much a seemingly redundant motif feature contributes to cell-type-specific CRE classification.

## When to use

Use when you need to test whether a motif-count feature category (e.g., overlapping vs. non-overlapping FIMO annotations) is biologically informative by retraining BagOfMotifs models and comparing auROC before and after the ablation.

## When NOT to use

- Input is already a per-sequence feature table or model predictions — the ablation requires regenerating motif counts from raw CRE coordinates via FIMO.
- The goal is de novo motif discovery rather than quantifying the predictive contribution of existing (gimme.vertebrate.v5.0.meme) motif annotations.
- The sequences of interest contain regulatory information beyond the input sequence (e.g., distal chromatin context), since BOM is constrained to cis-regulatory information within the input sequence.

## Inputs

- CRE coordinate files for the mouse E8.25 and human cell-line datasets, formatted as four-column text (chromosome, start, stop, condition)
- gimme.vertebrate.v5.0.meme motif library shipped with the BagOfMotifs package
- mouse mm10 genome (BSgenome.Mmusculus.UCSC.mm10)
- local FIMO installation path

## Outputs

- Per-dataset auROC values before and after retraining on non-overlapping motif annotations, saved as auroc_sensitivity_results.csv
- Confirmation (or refutation) of the reported 0.32 auROC reduction from removing overlapping motif annotations

## How to apply

Load the CRE coordinate files (four-column text: chromosome, start, stop, condition) and scan motifs with BagOfMotifs::runFIMO against the gimme.vertebrate.v5.0.meme library and the mouse mm10 genome (BSgenome.Mmusculus.UCSC.mm10). Repeat the FIMO scans at q-value cutoffs 0.1, 0.3, and 0.5, and at each threshold compute the fraction of CRE nucleotides covered by at least one motif annotation, checking that the annotation fraction reaches approximately 83% at q<=0.5. Convert the filtered (e.g., non-overlapping) motif annotations into per-sequence motif count features, retrain the gradient-boosted tree classifiers with BagOfMotifs::binModel(), and evaluate cell-type-specific prediction with BagOfMotifs::predict_binary_multi(). Compare the resulting auROC values against the original full-annotation models; in the source study, removing overlapping motif annotations reduced BOM's auROC by 0.32, indicating that seemingly redundant motif annotation (e.g., composite binding sites) carries biologically meaningful information.

## Related tools

- **FIMO** (Scans transcription factor motifs in CRE sequences at varying q-value cutoffs (0.1, 0.3, 0.5) to generate motif annotations used for the ablation) — https://meme-suite.org/meme/doc/install.html
- **BagOfMotifs (BOM)** (R package providing the motif-count representation, XGBoost model training, and binary/multi-class prediction used for the auROC comparison) — https://github.com/ewonglab/BOM_package
- **binModel** (Trains the gradient-boosted tree classifiers on the per-sequence motif count features after the ablation) — https://github.com/ewonglab/BOM_package
- **predict_binary_multi** (Evaluates cell-type-specific prediction performance (auROC) of the retrained models) — https://github.com/ewonglab/BOM_package
- **BSgenome.Mmusculus.UCSC.mm10** (Provides the mouse mm10 genome sequence used to extract CRE sequences for motif scanning)

## Examples

```
R> BagOfMotifs::runFIMO(motifs_path = motifs_path, FIMO_path = FIMO_path); R> BagOfMotifs::binModel(); R> BagOfMotifs::predict_binary_multi()
```

## Evaluation signals

- The fraction of CRE nucleotides covered by at least one motif annotation reaches approximately 83% at q<=0.5.
- Per-dataset auROC values for the original (full-annotation) models match those reported in the source study before retraining.
- Re-retraining on non-overlapping motif annotations reproduces the reported 0.32 auROC reduction relative to the original models.
- auroc_sensitivity_results.csv contains per-dataset auROC entries for both the before-ablation and after-ablation conditions.

## Limitations

- Removing overlapping motif annotations may discard biologically meaningful information such as composite binding sites, so ablation results must be interpreted as evidence about redundancy rather than proof of it.
- BOM's reliance on known TF motifs neglects informative sequences not captured by current motif annotations.
- BOM is constrained to cis-regulatory information within the input sequence; open chromatin status, histone marks, and enhancer activity can be influenced by regions beyond that sequence length.

## Evidence

- [results] auROC sensitivity of overlapping motif annotation: [evidence span withheld — anchor status: not_found]
- [methods] FIMO q-value threshold sweep: [evidence span withheld — anchor status: not_found]
- [methods] motif annotation coverage target: [evidence span withheld — anchor status: not_found]
- [methods] BOM prediction and evaluation functions: "BagOfMotifs::binModel()"
- [readme] FIMO scanning step: "BagOfMotifs::runFIMO(motifs_path = motifs_path, FIMO_path = FIMO_path)"
