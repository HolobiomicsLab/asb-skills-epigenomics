---
name: single-cell-atac-enhancer-classification
description: Use when single-cell ATAC peak sets need to be separated into cell-type-specific
  CREs and you need an interpretable, motif-based model that names the TF motifs (e.g.
  MEF2C, NKX2-5, GATA, FLI1) driving the classification.
when_to_use_negative:
- When the question requires regulatory information beyond the input sequence (e.g.
  open chromatin status, histone marks, or distal influences outside the CRE) — BOM
  is constrained to cis-regulatory information within the input sequence.
- When discovering novel sequence patterns unrepresented in current motif annotations
  is the goal — the reliance on known TF motifs neglects informative sequences not
  captured by current motif libraries.
- When motif order, orientation, or spacing (motif grammar) matters — BOM represents
  each sequence as an unordered set of motif counts and integration with motif grammar
  remains a future direction.
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_0203
tools:
- name: Bag-of-Motifs (BOM)
  role: R package that converts distal cis-regulatory elements to motif-count features,
    trains XGBoost classifiers per cell type, and computes SHAP motif importances
  repo: github.com/ewonglab/BOM_package
- name: FIMO
  role: Scans generated CRE FASTA sequences against the gimme.vertebrate.v5.0.meme
    motif library to produce motif hits/counts
- name: Gimme Maelstrom
  role: Cited as the source of the gimme.vertebrate.v5.0.meme motif library used for
    FIMO scanning; its ensemble differential motif approach lacks per-sequence interpretability,
    motivating BOM
  repo: vanheeringen-lab/gimmemotifs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-65362-2
    title: Motif-based models accurately predict cell type-specific distal regulatory
      elements
  source_content_hash: sha256:1c8e80429abed7659aa0928fd540fabbe39051bd3925c4aadf5217bda3cb5c99
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bag-of-Motifs (BOM)
  - FIMO
  - Gimme Maelstrom
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# single-cell-atac-enhancer-classification

## Summary

Classify cell-type-specific distal cis-regulatory elements from single-cell ATAC data by representing each CRE as unordered transcription factor motif counts (via FIMO scanning against gimme.vertebrate.v5.0.meme) and training gradient-boosted tree (XGBoost) models with the BOM R package, then interpreting motif importance with SHAP.

## When to use

Use when single-cell ATAC peak sets need to be separated into cell-type-specific CREs and you need an interpretable, motif-based model that names the TF motifs (e.g. MEF2C, NKX2-5, GATA, FLI1) driving the classification.

## When NOT to use

- When the question requires regulatory information beyond the input sequence (e.g. open chromatin status, histone marks, or distal influences outside the CRE) — BOM is constrained to cis-regulatory information within the input sequence.
- When discovering novel sequence patterns unrepresented in current motif annotations is the goal — the reliance on known TF motifs neglects informative sequences not captured by current motif libraries.
- When motif order, orientation, or spacing (motif grammar) matters — BOM represents each sequence as an unordered set of motif counts and integration with motif grammar remains a future direction.

## Inputs

- Four-column text/BED file of distal regulatory elements (chromosome, start, stop, condition) from single-cell ATAC data
- Reference genome: mouse mm10 (BSgenome.Mmusculus.UCSC.mm10)
- Motif library: gimme.vertebrate.v5.0.meme (shipped in BOM package extdata)
- Chromosome sizes file (mouse.chrom.sizes.txt)
- Gene annotation (Mus_musculus.GRCm38.92.gtf.gz)
- Installed FIMO binary (MEME Suite)

## Outputs

- Per-condition XGBoost (gradient-boosted tree) classifiers of cell-type-specific CREs
- Binary prediction and performance metrics from predict_binary_multi()
- Per-motif SHAP importance values from save_shap_multi()
- SHAP bar, beeswarm, and waterfall plots per cell-type condition from shapPlots_multi()
- Cell-type-specific motif importance rankings (e.g. MEF2C/NKX2-5/GATA for cardiomyocytes, FLI1/JUN-FOS/SOX7-ETV6 for endothelial)

## How to apply

Format input as a four-column BED-like text file (chromosome, start, stop, condition) and generate sequences with generateAllFasta() against the mouse mm10 genome (BSgenome.Mmusculus.UCSC.mm10). Scan motifs with FIMO using the gimme.vertebrate.v5.0.meme library via BagOfMotifs::runFIMO(), then train per-condition XGBoost classifiers on unordered motif-count features with BagOfMotifs::binModel(). Assess performance with predict_binary_multi(), estimate per-motif SHAP values with save_shap_multi(), and visualize with shapPlots_multi() (bar), shapPlots_multi(plotType="beeswarm"), and shapPlots_multi(plotType="waterfall"). The rationale is that BOM's motif-count representation combined with gradient-boosted trees captures combinatorial TF contributions while remaining directly interpretable, outperforming deep-learning models with fewer parameters.

## Related tools

- **Bag-of-Motifs (BOM)** (R package that converts distal cis-regulatory elements to motif-count features, trains XGBoost classifiers per cell type, and computes SHAP motif importances) — github.com/ewonglab/BOM_package
- **FIMO** (Scans generated CRE FASTA sequences against the gimme.vertebrate.v5.0.meme motif library to produce motif hits/counts)
- **Gimme Maelstrom** (Cited as the source of the gimme.vertebrate.v5.0.meme motif library used for FIMO scanning; its ensemble differential motif approach lacks per-sequence interpretability, motivating BOM) — vanheeringen-lab/gimmemotifs

## Examples

```
BagOfMotifs::runFIMO(motifs_path = motifs_path, FIMO_path = FIMO_path); BagOfMotifs::binModel(); BagOfMotifs::predict_binary_multi(); save_shap_multi(); shapPlots_multi(plotType = "waterfall")
```

## Evaluation signals

- predict_binary_multi() classification performance (e.g. AUC) shows the model reliably separates each cell-type condition's CREs from other conditions
- SHAP-derived motif rankings qualitatively reproduce the paper's cell-type associations: MEF2C, NKX2-5 and GATA motifs dominate cardiomyocyte CREs; FLI1, JUN/FOS and SOX7/ETV6 dominate endothelial CREs; TFAP2A for neural crest
- Local SHAP explanations at the cardiac Nkx2-5 locus show MEF2/SRF motif dominance
- shapPlots_multi() produces all three plot types (bar, beeswarm, waterfall) without errors and with plausible per-motif SHAP magnitudes
- FIMO run with gimme.vertebrate.v5.0.meme yields non-empty motif count matrices for all input CREs before model training

## Limitations

- BOM is constrained to cis-regulatory information within the input sequence; open chromatin status, histone marks, and enhancer activity can be influenced by regions beyond the sequence length used
- The reliance on known TF motifs neglects informative sequences not captured by current motif annotations, and TFs do not bind all their motifs
- Unknown how much of the remaining genome regions may share similar motif composition yet not share regulatory activity
- Integration with motif grammar (motif order, orientation, spacing) is not modeled and remains an important future direction
- BOM's linear-free tree ensemble means performance depends on sufficient training data for each cell state, as noted for sequence-based models generally

## Evidence

- [abstract] BOM, combining motif counts with gradient-boosted trees, accurately predicts cell-type-specific enhancers: "This minimalist representation, combined with gradient-boosted trees, enables the accurate prediction of cell-type-specific enhancers across mouse, human, zebrafish, and Arabidopsis datasets."
- [readme] BOM pipeline invocation order: "BagOfMotifs::binModel()

# Prediction and performance
BagOfMotifs::predict_binary_multi()

# Estimate SHAP 
save_shap_multi()"
- [readme] Input format and FIMO/motif library setup: "Input is a text file with four columns (chromosome, start, stop, condition)"
- [results] SHAP motif rankings correspond to lineage TFs: [evidence span withheld — anchor status: not_found]
- [methods] FIMO scanning step in the BOM workflow: "BagOfMotifs::runFIMO(
	motifs_path = motifs_path,
	FIMO_path   = FIMO_path
)"
