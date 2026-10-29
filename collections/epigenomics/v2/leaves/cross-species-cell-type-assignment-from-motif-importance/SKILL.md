---
name: cross-species-cell-type-assignment-from-motif-importance
description: Use when you have cell-type-annotated CRE BED intervals from two species
  sharing homologous cell types (e.g. seven adult heart cell types in human and mouse)
  and need to classify cell-type-specific regulatory sequences or assign single cells
  across species using motif-based models.
when_to_use_negative:
- Input is already a motif count or feature matrix — BOM's pipeline starts from genomic
  intervals and a BSgenome, so pre-computed features skip generateAllFasta/runFIMO
  and need a different path
- The task requires motif grammar (order, orientation, or spacing) information — BOM
  represents sequences as unordered motif counts and integration with motif grammar
  is described as an important future direction
- Cell types are not shared/homologous between the training and target species, since
  the cross-species transfer relies on seven shared adult heart cell types
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0203
tools:
- name: Bag-of-Motifs (BOM)
  role: 'Core R package: generates FASTA from BED, counts TF motifs, trains XGBoost
    multiclass models, computes SHAP attributions, and renders interpretation plots'
  repo: https://github.com/ewonglab/BOM_package
- name: FIMO
  role: Scans generated FASTA sequences against the gimme.vertebrate.v5.0.meme motif
    library to produce motif occurrences for counting
  repo: https://meme-suite.org/meme/doc/install.html
- name: BSgenome.Mmusculus.UCSC.mm10
  role: Mouse mm10 genome assembly used by generateAllFasta to extract CRE sequences
- name: GimmeMotifs
  role: Source of the gimme.vertebrate.v5.0.meme vertebrate motif library used for
    FIMO scanning
  repo: https://github.com/vanheeringen-lab/gimmemotifs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-65362-2
    title: Motif-based models accurately predict cell type-specific distal regulatory
      elements
  source_content_hash: sha256:5293d5ae53f354d880e79f20d9256cdb0ed86194d511edc4922681913dc95dc7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ewonglab/BOM_package
  tools:
  - BSgenome.Mmusculus.UCSC.mm10
  - Bag-of-Motifs (BOM)
  - FIMO
  - GimmeMotifs
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Cross-species cell-type assignment from motif-importance models

## Summary

Train multiclass Bag-of-Motifs (BOM) gradient-boosted-tree models on TF motif counts from distal cis-regulatory elements of shared cell types in one species, then use SHAP-aggregated feature attributions to assign single cells to cell types in another species' heart, checking generalization against reported auROCs (0.90 human, 0.85 mouse) and assignment rates (83% overall).

## When to use

Use when you have cell-type-annotated CRE BED intervals from two species sharing homologous cell types (e.g. seven adult heart cell types in human and mouse) and need to classify cell-type-specific regulatory sequences or assign single cells across species using motif-based models.

## When NOT to use

- Input is already a motif count or feature matrix — BOM's pipeline starts from genomic intervals and a BSgenome, so pre-computed features skip generateAllFasta/runFIMO and need a different path
- The task requires motif grammar (order, orientation, or spacing) information — BOM represents sequences as unordered motif counts and integration with motif grammar is described as an important future direction
- Cell types are not shared/homologous between the training and target species, since the cross-species transfer relies on seven shared adult heart cell types

## Inputs

- Four-column BED file (chromosome, start, stop, condition) with cell-type-annotated distal CREs
- Reference genome as BSgenome object (BSgenome.Mmusculus.UCSC.mm10)
- Motif library: gimme.vertebrate.v5.0.meme (packaged in BOM extdata)
- FIMO binary (MEME Suite, >= 5.5.4)
- Single-cell ATAC-derived cell-to-condition assignments for SHAP-based cross-species cell assignment

## Outputs

- Trained multiclass XGBoost models per cell type
- Cross-species multiclass predictions and per-cell-type auROC performance metrics
- SHAP feature attribution tables per motif per model (save_shap_multi())
- Cell-type assignments for single cells with assignment-rate statistics
- SHAP bar, beeswarm, and waterfall plots

## How to apply

Generate FASTA sequences from a four-column BED (chromosome, start, stop, condition) with BagOfMotifs::generateAllFasta() against the mouse mm10 genome (BSgenome.Mmusculus.UCSC.mm10), then scan sequences against the gimme.vertebrate.v5.0.meme motif library using FIMO (>= 5.5.4 with the default parser; set FIMO_updated = FALSE for older versions). Build unordered TF motif count vectors and train gradient-boosted-tree (XGBoost) multiclass models per cell type with BagOfMotifs::binModel(), then evaluate cross-species generalization with BagOfMotifs::predict_binary_multi(), comparing mean auROC to the reported benchmarks of 0.90 on human CREs and 0.85 on mouse CREs. Compute SHAP feature attributions with save_shap_multi(), aggregate them across motifs to assign each cell to its highest-attribution cell type, and verify the reported rates (83% overall of 55,834 human cells; 89% for ventricular cardiomyocytes). Render bar, beeswarm, and waterfall interpretation plots with shapPlots_multi(plotType = 'beeswarm') and shapPlots_multi(plotType = 'waterfall'). The rationale is that the motif-centric, order-independent representation captures a predictive sequence code that transfers across species for homologous cell types.

## Related tools

- **Bag-of-Motifs (BOM)** (Core R package: generates FASTA from BED, counts TF motifs, trains XGBoost multiclass models, computes SHAP attributions, and renders interpretation plots) — https://github.com/ewonglab/BOM_package
- **FIMO** (Scans generated FASTA sequences against the gimme.vertebrate.v5.0.meme motif library to produce motif occurrences for counting) — https://meme-suite.org/meme/doc/install.html
- **BSgenome.Mmusculus.UCSC.mm10** (Mouse mm10 genome assembly used by generateAllFasta to extract CRE sequences)
- **GimmeMotifs** (Source of the gimme.vertebrate.v5.0.meme vertebrate motif library used for FIMO scanning) — https://github.com/vanheeringen-lab/gimmemotifs

## Examples

```
library(BagOfMotifs); library("BSgenome.Mmusculus.UCSC.mm10"); generateAllFasta(bedDir = "path/to/input", genome = "Mmusculus"); BagOfMotifs::runFIMO(motifs_path = motifs_path, FIMO_path = FIMO_path); BagOfMotifs::binModel(); BagOfMotifs::predict_binary_multi(); save_shap_multi()
```

## Evaluation signals

- Mean auROC of the multiclass BOM models should approach the reported benchmarks: 0.90 on human CREs and 0.85 on mouse CREs across the seven shared adult heart cell types
- SHAP-aggregated cell assignment should reproduce ~83% correct assignment (46,223/55,834 human cells) overall
- Ventricular cardiomyocytes should show the highest cross-species concordance at ~89%
- FIMO version compatibility check: FIMO >= 5.5.4 uses the default parser; older versions require FIMO_updated = FALSE, and a version mismatch will corrupt motif scan output
- All required R dependencies (xgboost, shapviz, BSgenome, GenomicRanges, etc.) must be installed before running, as the README warns missing packages cause errors

## Limitations

- BOM is constrained to cis-regulatory information within the input sequence; open chromatin status, histone marks, and enhancer activity can be influenced by regions beyond the sequence length used
- Reliance on known TF motifs neglects informative sequences not captured by current motif annotations, and TFs do not bind all their motifs
- Unknown how much of the remaining genome regions may share similar motif composition yet not share regulatory activity
- Performance of sequence-based deep learning comparison methods is likely limited by the amount of training data available for many cell states

## Evidence

- [results] Multiclass BOM models trained on seven shared adult heart cell types achieved mean auROCs of 0.90 on human CREs and 0.85 on mouse CREs: [evidence span withheld — anchor status: not_found]
- [results] SHAP assignment rates: [evidence span withheld — anchor status: not_found]
- [methods] FASTA generation call: "generateAllFasta(bedDir = "path/to/input", genome = "Mmusculus")"
- [abstract] BOM representation rationale: "We present Bag-of-Motifs (BOM), a computational framework that represents distal cis-regulatory elements as unordered counts of transcription factor (TF) motifs."
- [readme] Tutorial genome requirement: "The tutorial requires the mouse mm10 genome. If you don't have this already:
BiocManager::install("BSgenome.Mmusculus.UCSC.mm10")"
- [discussion] Cis-only constraint: "BOM is constrained to cis-regulatory information within the input sequence."
