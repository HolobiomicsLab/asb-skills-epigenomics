---
name: gradient-boosted-tree-classifier-training
description: Use when you have per-sequence motif count matrices (e.g., from FIMO
  scans against gimme.vertebrate.v5.0.meme) and labeled CRE sets (chromosome, start,
  stop, condition) and need to train binary or multiclass classifiers for context-specific
  regulatory elements.
when_to_use_negative:
- Input is already a rich feature table (e.g., k-mer spectra or deep-learning embeddings)
  rather than motif counts — gkmSVM/LS-GKM or deep models may be more appropriate
- The regulatory mechanism depends on motif order, orientation, or spacing, since
  BOM's representation is explicitly independent of these (motif grammar integration
  remains a future direction)
- Analysis requires trans-acting or chromatin-context information beyond the input
  sequence — BOM is constrained to cis-regulatory information within the input sequence
  and relies on known TF motifs
edam_operation: http://edamontology.org/operation_0435
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3169
tools:
- name: Bag-of-Motifs (BOM)
  role: R package providing generateAllFasta, runFIMO, binModel (XGBoost training),
    predict_binary_multi, and SHAP estimation/visualization for motif-count classifiers
  repo: https://github.com/ewonglab/BOM_package
- name: XGBoost
  role: Extreme Gradient Boosting algorithm underlying binModel(); captures combinatorial
    contributions of TF motif counts for classification
- name: FIMO
  role: Scans CRE sequences for TF motifs from the gimme.vertebrate.v5.0.meme library
    to produce the motif occurrence counts used as features
- name: GimmeMotifs
  role: Source of the gimme.vertebrate.v5.0.meme motif library used as the motif PWM
    input
  repo: https://github.com/vanheeringen-lab/gimmemotifs
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-65362-2
    title: Motif-based models accurately predict cell type-specific distal regulatory
      elements
  source_content_hash: sha256:466282023006af063f3b42b80ca232d58d27585abf07e65e5e10c90577a1c3a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ewonglab/BOM_package
  tools:
  - BSgenome.Mmusculus.UCSC.mm10
  - Bag-of-Motifs (BOM)
  - BagOfMotifs
  - BagOfMotifs (BOM)
  - FIMO
  - GimmeMotifs
  - XGBoost
  - binModel
  edam_operation: http://edamontology.org/operation_0435
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# gradient-boosted-tree-classifier-training

## Summary

Trains XGBoost gradient-boosted tree classifiers on motif-count feature matrices derived from distal cis-regulatory element (CRE) sequences, as implemented in the BagOfMotifs (BOM) R package, to predict cell-type- or context-specific regulatory elements and quantify per-motif importance via SHAP.

## When to use

Use when you have per-sequence motif count matrices (e.g., from FIMO scans against gimme.vertebrate.v5.0.meme) and labeled CRE sets (chromosome, start, stop, condition) and need to train binary or multiclass classifiers for context-specific regulatory elements.

## When NOT to use

- Input is already a rich feature table (e.g., k-mer spectra or deep-learning embeddings) rather than motif counts — gkmSVM/LS-GKM or deep models may be more appropriate
- The regulatory mechanism depends on motif order, orientation, or spacing, since BOM's representation is explicitly independent of these (motif grammar integration remains a future direction)
- Analysis requires trans-acting or chromatin-context information beyond the input sequence — BOM is constrained to cis-regulatory information within the input sequence and relies on known TF motifs

## Inputs

- Four-column text/BED files (chromosome, start, stop, condition) with CRE coordinates and condition labels
- Motif library PWMs (gimme.vertebrate.v5.0.meme, shipped with BagOfMotifs)
- Reference genome BSgenome package (e.g., BSgenome.Mmusculus.UCSC.mm10)
- FIMO motif scan results over CRE sequences (motif occurrence counts per sequence)

## Outputs

- Trained binary and multiclass XGBoost classifier models per dataset
- Held-out performance metrics (auROC, auPR, precision, recall, F1, assignment accuracy) e.g., written to bom_model_metrics.csv
- Per-motif SHAP importance values from save_shap_multi()
- SHAP bar, beeswarm, and waterfall plots from shapPlots_multi()

## How to apply

Prepare each dataset as a four-column text file (chromosome, start, stop, condition), generate FASTA with BagOfMotifs::generateAllFasta(bedDir, genome = "Mmusculus") against the appropriate BSgenome package (e.g., BSgenome.Mmusculus.UCSC.mm10), and scan motifs with BagOfMotifs::runFIMO using gimme.vertebrate.v5.0.meme (set FIMO_updated = FALSE if FIMO version is < 5.5.4). Count motif occurrences per distal CRE to build the unordered bag-of-motifs feature matrix, then train binary and multiclass gradient-boosted tree models with BagOfMotifs::binModel() and evaluate held-out predictions with BagOfMotifs::predict_binary_multi(). Interpret motif contributions with save_shap_multi() and visualize with shapPlots_multi() (bar, beeswarm, waterfall). Rationale: the unordered motif-count representation combined with XGBoost captures combinatorial TF motif contributions while remaining interpretable, and BOM outperforms deep-learning models with fewer parameters.

## Related tools

- **Bag-of-Motifs (BOM)** (R package providing generateAllFasta, runFIMO, binModel (XGBoost training), predict_binary_multi, and SHAP estimation/visualization for motif-count classifiers) — https://github.com/ewonglab/BOM_package
- **XGBoost** (Extreme Gradient Boosting algorithm underlying binModel(); captures combinatorial contributions of TF motif counts for classification)
- **FIMO** (Scans CRE sequences for TF motifs from the gimme.vertebrate.v5.0.meme library to produce the motif occurrence counts used as features)
- **GimmeMotifs** (Source of the gimme.vertebrate.v5.0.meme motif library used as the motif PWM input) — https://github.com/vanheeringen-lab/gimmemotifs

## Examples

```
library(BagOfMotifs); generateAllFasta(bedDir = "path/to/input", genome = "Mmusculus"); BagOfMotifs::runFIMO(motifs_path = motifs_path, FIMO_path = FIMO_path); BagOfMotifs::binModel(); BagOfMotifs::predict_binary_multi()
```

## Evaluation signals

- Mouse E8.25 binary task: held-out performance should reach auROC ≈ 0.98, auPR ≈ 0.98, and ≈ 93% CRE assignment accuracy (precision/recall/F1 ≈ 0.93/0.92/0.92 on 2,489 held-out sequences)
- Mouse E8.25 multiclass (17-class) task: precision ≈ 0.99, recall ≈ 0.88, auPR ≈ 0.99
- Cross-dataset benchmarks: ENCODE F1 ~0.92 and auPR ~0.98, blood F1 ~0.90, zebrafish F1 ~0.96, Drosophila STARR-seq Pearson ~0.78/0.42, AML F1 ~0.99
- Motif-annotation coverage sanity check: fraction of CRE nucleotides covered by at least one motif annotation should reach approximately 83% at q <= 0.5
- SHAP ablation check: removing overlapping motif annotations should reproduce the reported 0.32 auROC reduction (auroc_sensitivity_results.csv)

## Limitations

- BOM is constrained to cis-regulatory information within the input sequence; open chromatin status, histone marks, and enhancer activity can be influenced by regions beyond the sequence length used
- Reliance on known TF motifs neglects informative sequences not captured by current motif annotations, and TFs do not bind all their motifs
- Seemingly redundant (overlapping) motif annotations carry biologically meaningful information — removing them reduces auROC by 0.32, so deduplication is not safe
- Unknown how much of the remaining genome regions may share similar motif composition yet not share regulatory activity

## Evidence

- [abstract] This minimalist representation, combined with gradient-boosted trees, enables the accurate prediction of cell-type-specific enhancers across mouse, human, zebrafish, and Arabidopsis datasets.: "This minimalist representation, combined with gradient-boosted trees, enables the accurate prediction of cell-type-specific enhancers across mouse, human, zebrafish, and Arabidopsis datasets."
- [readme] BOM is an R package that uses XGBoost to construct predictive models of cis-regulatory sequences between cell states.: "BAG-of-Motifs (BOM) is an R package that uses XGBoost to construct predictive models of cis-regulatory sequences between cell states."
- [results] Removing overlapping motif annotations reduced BOM's auROC by 0.32, indicating that seemingly redundant motif annotation may reflect biologically meaningful information.: [evidence span withheld — anchor status: not_found]
- [methods] Generate FASTA and annotate motifs generateAllFasta(bedDir = "path/to/input", genome = "Mmusculus"): "generateAllFasta(bedDir = "path/to/input", genome = "Mmusculus")"
- [discussion] BOM is constrained to cis-regulatory information within the input sequence.: "BOM is constrained to cis-regulatory information within the input sequence."
