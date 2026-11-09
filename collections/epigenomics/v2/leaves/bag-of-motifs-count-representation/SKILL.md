---
name: bag-of-motifs-count-representation
description: Use when you need to classify or regress context-specific cis-regulatory
  elements (enhancers) from sequence alone across species, cell types, or disease
  states, and want an interpretable motif-count model rather than a deep-learning
  sequence model.
when_to_use_negative:
- The regulatory activity is expected to depend on features beyond the input sequence
  length, such as open chromatin status, histone marks, or distal interactions outside
  the scanned region — BOM is constrained to cis-regulatory information within the
  input sequence.
- The relevant biology depends on motif order, orientation, or spacing grammar — BOM
  represents sequences as unordered motif counts independent of motif grammar, which
  remains an important future direction.
- You need to discover novel sequence patterns not captured by current TF motif annotations
  — BOM's reliance on known TF motifs neglects informative sequences outside existing
  PWM libraries.
edam_operation: http://edamontology.org/operation_0445
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
tools:
- name: Bag-of-Motifs (BOM)
  role: 'Core R package: generates FASTA, builds motif-count matrices, trains XGBoost
    models, computes SHAP motif importances and visualizations'
  repo: https://github.com/ewonglab/BOM_package
- name: FIMO
  role: Scans generated enhancer sequences for occurrences of TF motifs from the gimme.vertebrate.v5.0.meme
    PWM collection to produce motif annotations for counting
- name: GimmeMotifs
  role: Source of the gimme.vertebrate.v5.0.meme motif collection used as the motifs_path
    input to BOM's FIMO scan
  repo: https://github.com/vanheeringen-lab/gimmemotifs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-65362-2
    title: Motif-based models accurately predict cell type-specific distal regulatory
      elements
  source_content_hash: sha256:d602b59c6d252fb15e66e118003ed46022c427a7c2f67d16b7d79303adb94c15
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ewonglab/BOM_package
  tools:
  - Bag-of-Motifs (BOM)
  - BagOfMotifs (BOM)
  - FIMO
  - GimmeMotifs
  edam_operation: http://edamontology.org/operation_0445
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# bag-of-motifs-count-representation

## Summary

Represents distal cis-regulatory elements as unordered counts of TF motifs and trains gradient-boosted (XGBoost) classifiers to predict cell-type- or condition-specific regulatory elements, with SHAP-based per-motif interpretation.

## When to use

Use when you need to classify or regress context-specific cis-regulatory elements (enhancers) from sequence alone across species, cell types, or disease states, and want an interpretable motif-count model rather than a deep-learning sequence model.

## When NOT to use

- The regulatory activity is expected to depend on features beyond the input sequence length, such as open chromatin status, histone marks, or distal interactions outside the scanned region — BOM is constrained to cis-regulatory information within the input sequence.
- The relevant biology depends on motif order, orientation, or spacing grammar — BOM represents sequences as unordered motif counts independent of motif grammar, which remains an important future direction.
- You need to discover novel sequence patterns not captured by current TF motif annotations — BOM's reliance on known TF motifs neglects informative sequences outside existing PWM libraries.

## Inputs

- Four-column BED-like text file (chromosome, start, stop, condition) with enhancer coordinates and cell-type/condition labels
- Genome of choice installed as a BSgenome package (e.g. BSgenome.Mmusculus.UCSC.mm10)
- TF motif PWM file, e.g. gimme.vertebrate.v5.0.meme
- Locally installed FIMO binary (MEME Suite)

## Outputs

- Motif-count feature matrices per dataset
- Trained XGBoost (gradient-boosted) classifiers per cell type/condition
- Performance metrics: F1 and auPR for classification; Pearson correlation for Drosophila STARR-seq regression
- Per-motif SHAP importance values with bar, beeswarm, and waterfall visualizations

## How to apply

Prepare each dataset as a four-column text file (chromosome, start, stop, condition) listing enhancer coordinates and condition labels. Generate FASTA sequences with BagOfMotifs::generateAllFasta() against the appropriate BSgenome (e.g. BSgenome.Mmusculus.UCSC.mm10) and scan them with FIMO using the gimme.vertebrate.v5.0.meme motif set (set FIMO_updated = FALSE for FIMO <5.5.4). Build motif-count matrices and train gradient-boosted classifiers with BagOfMotifs::binModel(), then evaluate with BagOfMotifs::predict_binary_multi() to obtain F1 and auPR for classification (or Pearson correlation for Drosophila STARR-seq regression). Estimate per-motif SHAP values with save_shap_multi() and visualize them with shapPlots_multi() using bar, beeswarm, or waterfall plot types to identify the motifs driving cell-type-specific classification.

## Related tools

- **Bag-of-Motifs (BOM)** (Core R package: generates FASTA, builds motif-count matrices, trains XGBoost models, computes SHAP motif importances and visualizations) — https://github.com/ewonglab/BOM_package
- **FIMO** (Scans generated enhancer sequences for occurrences of TF motifs from the gimme.vertebrate.v5.0.meme PWM collection to produce motif annotations for counting)
- **GimmeMotifs** (Source of the gimme.vertebrate.v5.0.meme motif collection used as the motifs_path input to BOM's FIMO scan) — https://github.com/vanheeringen-lab/gimmemotifs

## Examples

```
generateAllFasta(bedDir = "path/to/input", genome = "Mmusculus"); BagOfMotifs::runFIMO(motifs_path = motifs_path, FIMO_path = FIMO_path); BagOfMotifs::binModel(); BagOfMotifs::predict_binary_multi()
```

## Evaluation signals

- Benchmark metrics reproduce reported values within reasonable tolerance: ENCODE F1 ~0.92 and auPR ~0.98, blood F1 ~0.90, zebrafish F1 ~0.96, Drosophila STARR-seq Pearson ~0.78/0.42, AML F1 ~0.99
- Motif-count matrices are non-negative integer counts consistent with FIMO hits per sequence, with one row per input region and one column per motif in the PWM file
- SHAP bar/beeswarm/waterfall plots return per-motif contributions whose top-ranked motifs are biologically consistent with the cell-type condition being classified
- The pipeline runs end-to-end with the same commands on all six datasets (mouse ENCODE, human blood, zebrafish, Drosophila, Arabidopsis, AML), demonstrating cross-species generalization of the motif-count representation

## Limitations

- BOM is constrained to cis-regulatory information within the input sequence; open chromatin status, histone marks, and enhancer activity can be influenced by regions beyond the sequence length used.
- Reliance on known TF motifs neglects informative sequences not captured by current motif annotations, and TFs do not bind all their motifs.
- It is unknown how much of the remaining genome regions may share similar motif composition yet not share regulatory activity.
- FIMO version matters: FIMO_updated = FALSE must be set when FIMO version is <5.5.4; missing R dependencies listed in the README may cause errors.

## Evidence

- [abstract] We present Bag-of-Motifs (BOM), a computational framework that represents distal cis-regulatory elements as unordered counts of transcription factor (TF) motifs.: "We present Bag-of-Motifs (BOM), a computational framework that represents distal cis-regulatory elements as unordered counts of transcription factor (TF) motifs."
- [readme] BOM pipeline commands: "# Generate FASTA and annotate motifs
generateAllFasta(bedDir = "path/to/input", genome = "Mmusculus")"
- [readme] BOM SHAP interpretation: "Through the use of SHapley Additive exPlanations (SHAP), BOM helps identify the important TF binding motifs that contribute to the classification."
- [intro] Motif order independence: "representing each distal regulatory sequence as an unordered set of motif counts, independent of motif order, orientation, or spacing"
- [readme] XGBoost model basis: "BAG-of-Motifs (BOM) is an R package that uses XGBoost to construct predictive models of cis-regulatory sequences between cell states."
- [abstract] Interpretability advantage over deep learning: "Despite its simplicity, BOM outperforms more complex deep-learning models while using fewer parameters."
