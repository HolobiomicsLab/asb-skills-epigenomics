---
name: motif-count-feature-engineering
description: Use when building cell-type-specific CRE classifiers from four-column
  BED-like inputs (chromosome, start, stop, condition) and you need interpretable
  motif-count features rather than k-mer or deep-learning representations.
when_to_use_negative:
- The input is already a feature table or count matrix rather than genomic CRE coordinates
  needing motif annotation
- The analysis requires motif order, orientation, or spacing (grammar) information
  — BOM's representation is explicitly unordered and independent of spacing
- The target sequences rely on regulatory information outside the input sequence,
  such as distal enhancer activity influenced by open chromatin or histone marks beyond
  the CRE
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_0102
tools:
- name: BagOfMotifs (BOM)
  role: R package providing generateAllFasta, runFIMO, binModel, predict_binary_multi,
    and save_shap_multi for motif-count feature engineering and XGBoost model training
  repo: https://github.com/ewonglab/BOM_package
- name: FIMO
  role: MEME Suite tool invoked via BagOfMotifs::runFIMO to scan the gimme.vertebrate.v5.0.meme
    motifs across CRE sequences at q-value thresholds 0.1–0.5
- name: BSgenome.Mmusculus.UCSC.mm10
  role: Mouse mm10 genome Bioconductor package used to extract CRE sequences from
    coordinates
- name: GimmeMotifs
  role: Source of the gimme.vertebrate.v5.0.meme motif library used as FIMO input
  repo: https://github.com/vanheeringen-lab/gimmemotifs
- name: XGBoost
  role: Gradient-boosted tree backend used by binModel() to train classifiers on motif
    count features
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-65362-2
    title: Motif-based models accurately predict cell type-specific distal regulatory
      elements
  source_content_hash: sha256:c8aa3ce47ffcfba05a88d4736d86dc2aebbf27f6d24e1403f579400572501f89
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ewonglab/BOM_package
  tools:
  - BSgenome.Mmusculus.UCSC.mm10
  - BagOfMotifs
  - BagOfMotifs (BOM)
  - FIMO
  - GimmeMotifs
  - XGBoost
  - binModel
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# motif-count-feature-engineering

## Summary

Engineer unordered per-sequence TF motif-count features from CRE sequences by scanning with FIMO (gimme.vertebrate.v5.0.meme motifs) and converting motif annotations into count matrices for gradient-boosted tree classifiers in the BagOfMotifs (BOM) framework.

## When to use

Use when building cell-type-specific CRE classifiers from four-column BED-like inputs (chromosome, start, stop, condition) and you need interpretable motif-count features rather than k-mer or deep-learning representations.

## When NOT to use

- The input is already a feature table or count matrix rather than genomic CRE coordinates needing motif annotation
- The analysis requires motif order, orientation, or spacing (grammar) information — BOM's representation is explicitly unordered and independent of spacing
- The target sequences rely on regulatory information outside the input sequence, such as distal enhancer activity influenced by open chromatin or histone marks beyond the CRE

## Inputs

- Four-column text CRE coordinate files (chromosome, start, stop, condition) for mouse E8.25 and human cell-line datasets
- gimme.vertebrate.v5.0.meme motif library (shipped in BagOfMotifs extdata)
- Mouse mm10 genome (BSgenome.Mmusculus.UCSC.mm10)
- Locally installed FIMO binary (MEME Suite)

## Outputs

- FIMO motif annotation tracks over CRE sequences
- Per-sequence motif count feature matrix
- Trained gradient-boosted tree (XGBoost) classifiers from binModel()
- Per-dataset auROC values (e.g., auroc_sensitivity_results.csv)
- SHAP importance scores per motif via save_shap_multi()

## How to apply

Convert the CRE coordinate files to sequences with BagOfMotifs::generateAllFasta against the mouse mm10 genome (BSgenome.Mmusculus.UCSC.mm10), then scan motifs by running FIMO via BagOfMotifs::runFIMO with the gimme.vertebrate.v5.0.meme library; threshold choice is a key decision point — FIMO q-value cutoffs of 0.1, 0.3, and 0.5 were ablated, with the fraction of CRE nucleotides covered by at least one annotation reaching ~83% at q<=0.5. Convert the resulting non-overlapping motif annotations into per-sequence motif count features and train XGBoost gradient-boosted tree classifiers with BagOfMotifs::binModel(). Evaluate cell-type-specific prediction with BagOfMotifs::predict_binary_multi(), comparing auROC values against alternative feature representations. An ablation removing overlapping motif annotations reduces auROC by 0.32, so overlapping (composite) annotations should be retained as they carry biologically meaningful signal.

## Related tools

- **BagOfMotifs (BOM)** (R package providing generateAllFasta, runFIMO, binModel, predict_binary_multi, and save_shap_multi for motif-count feature engineering and XGBoost model training) — https://github.com/ewonglab/BOM_package
- **FIMO** (MEME Suite tool invoked via BagOfMotifs::runFIMO to scan the gimme.vertebrate.v5.0.meme motifs across CRE sequences at q-value thresholds 0.1–0.5)
- **BSgenome.Mmusculus.UCSC.mm10** (Mouse mm10 genome Bioconductor package used to extract CRE sequences from coordinates)
- **GimmeMotifs** (Source of the gimme.vertebrate.v5.0.meme motif library used as FIMO input) — https://github.com/vanheeringen-lab/gimmemotifs
- **XGBoost** (Gradient-boosted tree backend used by binModel() to train classifiers on motif count features)

## Examples

```
library(BagOfMotifs); generateAllFasta(bedDir="path/to/input", genome="Mmusculus"); BagOfMotifs::runFIMO(motifs_path=motifs_path, FIMO_path=FIMO_path); BagOfMotifs::binModel()
```

## Evaluation signals

- Fraction of CRE nucleotides covered by at least one motif annotation reaches approximately 83% at q<=0.5
- auROC sensitivity comparison: removing overlapping motif annotations should reproduce the reported 0.32 auROC reduction (saved as auroc_sensitivity_results.csv)
- Per-dataset auROC values from predict_binary_multi() should match or exceed the original motif-annotation models when overlapping annotations are retained
- FIMO runs must complete successfully at all three q-value cutoffs (0.1, 0.3, 0.5) for downstream count features to be comparable

## Limitations

- BOM is constrained to cis-regulatory information within the input sequence; open chromatin, histone marks, and enhancer activity can be influenced by regions beyond the sequence length used
- Reliance on known TF motifs neglects informative sequences not captured by current motif annotations, and TFs do not bind all their motifs
- Unknown how much of the remaining genome regions may share similar motif composition yet not share regulatory activity
- Integration with motif grammar (order, orientation, spacing) remains an unimplemented future direction
- Missing R package dependencies may cause errors; install the listed packages (foreach, xgboost, GenomicRanges, shapviz, etc.) before running

## Evidence

- [abstract] BOM represents CREs as unordered motif counts: "We present Bag-of-Motifs (BOM), a computational framework that represents distal cis-regulatory elements as unordered counts of transcription factor (TF) motifs."
- [methods] FIMO scanning workflow: "BagOfMotifs::runFIMO(
	motifs_path = motifs_path,
	FIMO_path   = FIMO_path
)"
- [full_text] Threshold ablation and 83% coverage: [evidence span withheld — anchor status: not_found]
- [results] Overlapping motif ablation finding: [evidence span withheld — anchor status: not_found]
- [readme] Installation and FIMO dependency: "devtools::install_github("ewonglab/BOM_package")"
- [discussion] Sequence-only constraint limitation: "BOM is constrained to cis-regulatory information within the input sequence."
