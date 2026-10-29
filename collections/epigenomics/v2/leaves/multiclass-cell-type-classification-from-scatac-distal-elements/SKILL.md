---
name: multiclass-cell-type-classification-from-scatac-distal-elements
description: Use when scATAC-seq peak/CRE data must be assigned to cell types from
  sequence alone — e.g., multiclass classification of cell-type-specific distal elements
  or cross-species cell-type assignment using motif counts.
when_to_use_negative:
- Input is already a feature/counts matrix rather than genomic coordinates of CREs
  — BOM requires a BED file of genomic intervals to extract sequences
- The regulatory activity is expected to depend on motif grammar (order, orientation,
  spacing) — BOM's representation is unordered motif counts independent of motif order,
  orientation, or spacing
- Informative sequences not captured by known TF motif annotations drive the classification
  — reliance on known motifs neglects unannotated informative sequences
edam_operation: http://edamontology.org/operation_1777
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3173
tools:
- name: Bag-of-Motifs (BOM)
  role: R package providing binModel() for motif-count feature construction and XGBoost
    multiclass model training, predict_binary_multi() for prediction/auROC, save_shap_multi()
    for SHAP estimation, and shapPlots_multi() for visualization
  repo: https://github.com/ewonglab/BOM_package
- name: FIMO
  role: Scans generated FASTA sequences against the gimme.vertebrate.v5.0.meme motif
    library to produce motif occurrences for count-vector construction
- name: BSgenome.Mmusculus.UCSC.mm10
  role: Mouse mm10 genome used by generateAllFasta to extract CRE sequences from BED
    coordinates
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-65362-2
    title: Motif-based models accurately predict cell type-specific distal regulatory
      elements
  source_content_hash: sha256:170b675b9421516ea450d50825f3c5d0d4dbb7b56918008c7590193c7f3013bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ewonglab/BOM_package
  tools:
  - BSgenome.Mmusculus.UCSC.mm10
  - Bag-of-Motifs (BOM)
  - FIMO
  edam_operation: http://edamontology.org/operation_1777
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# multiclass-cell-type-classification-from-scatac-distal-elements

## Summary

Train multiclass Bag-of-Motifs (BOM) models on seven shared adult heart cell types using unordered TF motif counts from distal cis-regulatory elements, then use SHAP-aggregated attributions to assign single cells to cell types, including cross-species transfer between human and mouse heart.

## When to use

Use when scATAC-seq peak/CRE data must be assigned to cell types from sequence alone — e.g., multiclass classification of cell-type-specific distal elements or cross-species cell-type assignment using motif counts.

## When NOT to use

- Input is already a feature/counts matrix rather than genomic coordinates of CREs — BOM requires a BED file of genomic intervals to extract sequences
- The regulatory activity is expected to depend on motif grammar (order, orientation, spacing) — BOM's representation is unordered motif counts independent of motif order, orientation, or spacing
- Informative sequences not captured by known TF motif annotations drive the classification — reliance on known motifs neglects unannotated informative sequences

## Inputs

- Four-column BED file (chromosome, start, stop, condition) of condition-annotated distal regulatory elements
- Reference genome BSgenome.Mmusculus.UCSC.mm10
- TF motif library gimme.vertebrate.v5.0.meme
- FIMO executable (>= 5.5.4 recommended)

## Outputs

- Multiclass cell-type predictions and per-cell-type auROC performance metrics
- SHAP feature attributions per motif feature (save_shap_multi output)
- SHAP-aggregated single-cell cell-type assignments with concordance rates
- SHAP bar, beeswarm, and waterfall plots

## How to apply

Start from a four-column BED file (chromosome, start, stop, condition) and generate FASTA sequences with generateAllFasta against the mouse mm10 genome (BSgenome.Mmusculus.UCSC.mm10). Scan the sequences against the gimme.vertebrate.v5.0.meme motif library using FIMO (>= 5.5.4 with the default parser; set FIMO_updated = FALSE for older versions), then build unordered TF motif count vectors and train gradient-boosted-tree multiclass models per cell type with BagOfMotifs::binModel(). Evaluate per-cell-type auROC with BagOfMotifs::predict_binary_multi() — expected mean auROCs are ~0.90 on human CREs and ~0.85 on mouse CREs. Compute SHAP attributions with save_shap_multi(), aggregate them to assign cells to cell types, and verify assignment concordance (e.g., 83% overall for 55,834 human cells, 89% for ventricular cardiomyocytes), rendering bar, beeswarm, and waterfall plots via shapPlots_multi().

## Related tools

- **Bag-of-Motifs (BOM)** (R package providing binModel() for motif-count feature construction and XGBoost multiclass model training, predict_binary_multi() for prediction/auROC, save_shap_multi() for SHAP estimation, and shapPlots_multi() for visualization) — https://github.com/ewonglab/BOM_package
- **FIMO** (Scans generated FASTA sequences against the gimme.vertebrate.v5.0.meme motif library to produce motif occurrences for count-vector construction)
- **BSgenome.Mmusculus.UCSC.mm10** (Mouse mm10 genome used by generateAllFasta to extract CRE sequences from BED coordinates)

## Examples

```
library(BagOfMotifs); generateAllFasta(bedDir = "path/to/input", genome = "Mmusculus"); BagOfMotifs::runFIMO(motifs_path = motifs_path, FIMO_path = FIMO_path); BagOfMotifs::binModel(); BagOfMotifs::predict_binary_multi(); save_shap_multi(); shapPlots_multi(plotType = "beeswarm")
```

## Evaluation signals

- Mean auROC reaches ~0.90 on human CREs and ~0.85 on mouse CREs across the seven shared adult heart cell types
- SHAP-aggregated assignment correctly classifies 83% (46,223/55,834) of human cells
- Ventricular cardiomyocytes show the highest cross-species assignment concordance at 89%
- FIMO version compatibility check: FIMO >= 5.5.4 uses the default parser; FIMO_updated = FALSE must be set for older versions
- SHAP outputs render successfully in all three plot types: shapPlots_multi(), shapPlots_multi(plotType = 'beeswarm'), and shapPlots_multi(plotType = 'waterfall')

## Limitations

- BOM is constrained to cis-regulatory information within the input sequence; open chromatin status, histone marks, and enhancer activity can be influenced by regions beyond the sequence length used
- Reliance on known TF motifs neglects informative sequences not captured by current motif annotations, and TFs do not bind all their motifs
- Unknown how much of the remaining genome regions may share similar motif composition yet not share regulatory activity

## Evidence

- [results] Multiclass BOM models trained on seven shared adult heart cell types achieved mean auROCs of 0.90 on human CREs and 0.85 on mouse CREs: [evidence span withheld — anchor status: not_found]
- [results] SHAP-aggregated assignment rates: [evidence span withheld — anchor status: not_found]
- [readme] FASTA generation and genome: "generateAllFasta(bedDir = "path/to/input", genome = "Mmusculus")"
- [other] FIMO version note: [evidence span withheld — anchor status: not_found]
- [abstract] BOM representation: "We present Bag-of-Motifs (BOM), a computational framework that represents distal cis-regulatory elements as unordered counts of transcription factor (TF) motifs."
- [readme] SHAP interpretation: "Through the use of SHapley Additive exPlanations (SHAP), BOM helps identify the important TF binding motifs that contribute to the classification."
