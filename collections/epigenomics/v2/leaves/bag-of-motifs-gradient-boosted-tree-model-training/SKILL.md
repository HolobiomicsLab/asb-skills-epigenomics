---
name: bag-of-motifs-gradient-boosted-tree-model-training
description: Use when you have condition-annotated CRE coordinates (chromosome, start,
  stop, condition BED) from scATAC-seq or similar, and need to classify cell-type-specific
  distal regulatory elements from sequence-derived TF motif counts.
when_to_use_negative:
- The task requires capture of motif grammar (order, orientation, spacing) — BOM represents
  sequences as unordered motif counts and integration with motif grammar remains a
  future direction
- Regulatory activity depends on sequence outside the input region — BOM is constrained
  to cis-regulatory information within the input sequence and cannot model distal
  open-chromatin or histone-mark context
- You need features absent from current motif annotations — reliance on known TF motifs
  neglects informative sequences not captured by motif databases
edam_operation: http://edamontology.org/operation_0267
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_3308
tools:
- name: Bag-of-Motifs (BOM)
  role: R package providing generateAllFasta, runFIMO wrapper, binModel (XGBoost motif-count
    model training), predict_binary_multi, save_shap_multi, and shapPlots_multi
  repo: github.com/ewonglab/BOM_package
- name: FIMO
  role: Motif scanning of generated FASTA sequences against the gimme.vertebrate.v5.0.meme
    library to produce motif hits for counting
  repo: https://meme-suite.org/meme/doc/install.html
- name: GimmeMotifs (gimmemotifs)
  role: Source of the gimme.vertebrate.v5.0.meme motif library used for sequence scanning
  repo: github.com/vanheeringen-lab/gimmemotifs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-65362-2
    title: Motif-based models accurately predict cell type-specific distal regulatory
      elements
  source_content_hash: sha256:07616cd84e23060a1e15b8487c10356043880655ac9bcea62fd316b47f577334
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BSgenome.Mmusculus.UCSC.mm10
  - Bag-of-Motifs (BOM)
  - FIMO
  - GimmeMotifs (gimmemotifs)
  edam_operation: http://edamontology.org/operation_0267
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# bag-of-motifs-gradient-boosted-tree-model-training

## Summary

Train multiclass gradient-boosted-tree (XGBoost) classifiers with the Bag-of-Motifs (BOM) R package, representing distal CREs as unordered TF motif count vectors and interpreting the models with SHAP to assign cell-type specificity (e.g., seven shared adult heart cell types across human and mouse).

## When to use

Use when you have condition-annotated CRE coordinates (chromosome, start, stop, condition BED) from scATAC-seq or similar, and need to classify cell-type-specific distal regulatory elements from sequence-derived TF motif counts.

## When NOT to use

- The task requires capture of motif grammar (order, orientation, spacing) — BOM represents sequences as unordered motif counts and integration with motif grammar remains a future direction
- Regulatory activity depends on sequence outside the input region — BOM is constrained to cis-regulatory information within the input sequence and cannot model distal open-chromatin or histone-mark context
- You need features absent from current motif annotations — reliance on known TF motifs neglects informative sequences not captured by motif databases

## Inputs

- Condition-annotated BED file with four columns (chromosome, start, stop, condition)
- Genome assembly, e.g. BSgenome.Mmusculus.UCSC.mm10 for mouse mm10
- TF motif library in MEME format, e.g. gimme.vertebrate.v5.0.meme
- FIMO binary (meme-suite) with FIMO >= 5.5.4 recommended

## Outputs

- FASTA sequences per condition from generateAllFasta
- FIMO motif scan hits per sequence
- Trained XGBoost multiclass models per cell type (binModel)
- Multiclass predictions and per-cell-type auROC performance (predict_binary_multi)
- SHAP feature attributions (save_shap_multi) and bar/beeswarm/waterfall plots (shapPlots_multi)
- Cross-species SHAP-aggregated cell-type assignments

## How to apply

Generate FASTA sequences from the four-column condition-annotated BED with BagOfMotifs::generateAllFasta() using the mouse mm10 genome (BSgenome.Mmusculus.UCSC.mm10), then scan them against the gimme.vertebrate.v5.0.meme motif library with BagOfMotifs::runFIMO() (set FIMO_updated = FALSE for FIMO versions older than 5.5.4). Build unordered TF motif count vectors and train gradient-boosted-tree multiclass models per cell type with BagOfMotifs::binModel(), then obtain multiclass predictions and per-cell-type auROCs with BagOfMotifs::predict_binary_multi(). Rationale: representing each distal regulatory sequence as unordered motif counts, independent of motif order, orientation, or spacing, combined with XGBoost, captures combinatorial TF motif contributions while remaining interpretable; the article reports mean auROCs of 0.90 on human and 0.85 on mouse heart CREs. Compute SHAP feature attributions with save_shap_multi() and aggregate them to assign cells to cell types cross-species, verifying against reported rates (83% overall of 55,834 human cells; 89% ventricular cardiomyocytes), then visualize with shapPlots_multi(plotType = 'bar'/'beeswarm'/'waterfall').

## Related tools

- **Bag-of-Motifs (BOM)** (R package providing generateAllFasta, runFIMO wrapper, binModel (XGBoost motif-count model training), predict_binary_multi, save_shap_multi, and shapPlots_multi) — github.com/ewonglab/BOM_package
- **FIMO** (Motif scanning of generated FASTA sequences against the gimme.vertebrate.v5.0.meme library to produce motif hits for counting) — https://meme-suite.org/meme/doc/install.html
- **GimmeMotifs (gimmemotifs)** (Source of the gimme.vertebrate.v5.0.meme motif library used for sequence scanning) — github.com/vanheeringen-lab/gimmemotifs

## Examples

```
library(BagOfMotifs); generateAllFasta(bedDir="path/to/input", genome="Mmusculus"); BagOfMotifs::runFIMO(motifs_path=motifs_path, FIMO_path=FIMO_path); BagOfMotifs::binModel(); BagOfMotifs::predict_binary_multi(); save_shap_multi(); shapPlots_multi(plotType="beeswarm")
```

## Evaluation signals

- Mean per-cell-type auROC should approximate the reported values: 0.90 on human CREs and 0.85 on mouse CREs for the seven shared adult heart cell types
- SHAP-aggregated cross-species assignment should classify ~83% of human cells (46,223/55,834), with ventricular cardiomyocytes at ~89% concordance
- Input BED must parse as exactly four columns: chromosome, start, stop, condition, before generateAllFasta
- FIMO version must be >= 5.5.4 with the default parser (or FIMO_updated = FALSE for older versions) or scan parsing will fail
- All required R dependencies (foreach, doParallel, tidyr, dplyr, xgboost, rsample, cvAUC, pROC, ggplot2, yardstick, cowplot, shapviz, gggenes, shades, GenomicRanges, IRanges, GenomicFeatures, GenomeInfoDb, Biostrings, BSgenome) must be installed before running

## Limitations

- BOM is constrained to cis-regulatory information within the input sequence; open chromatin status, histone marks, and enhancer activity can be influenced by regions beyond the sequence length used in BOM
- Reliance on known TF motifs neglects informative sequences not captured by current motif annotations, and TFs do not bind all their motifs
- Unknown how much of the remaining genome shares similar motif composition yet does not share regulatory activity
- Missing R package dependencies may cause errors during execution; install the listed required packages before proceeding

## Evidence

- [intro] BOM represents each distal regulatory sequence as unordered motif counts: "representing each distal regulatory sequence as an unordered set of motif counts, independent of motif"
- [abstract] BOM achieves high predictive accuracy and outperforms deep learning: "Despite its simplicity, BOM outperforms more complex deep-learning models while using fewer parameters."
- [readme] Workflow: FASTA generation, FIMO scan, model training, prediction, SHAP, plots: "generateAllFasta(bedDir = "path/to/input", genome = "Mmusculus")
BagOfMotifs::runFIMO(motifs_path = motifs_path, FIMO_path = FIMO_path)"
- [readme] SHAP interpretation and visualization in BOM: "Through the use of SHapley Additive exPlanations (SHAP), BOM helps identify the important TF binding motifs that contribute to the classification."
- [discussion] BOM constrained to cis-regulatory information within input sequence: "BOM is constrained to cis-regulatory information within the input sequence."
