---
name: cell-type-classification-performance-evaluation
description: Use when you have trained cell-type CRE classifiers (e.g., BOM/XGBoost
  on motif counts) and need to quantify prediction accuracy, such as fraction of elements
  assigned to the correct cell type or cross-species transfer rates.
when_to_use_negative:
- Input is already a feature/model output table and no cell-type labels or classifier
  predictions are involved — evaluation of classification rates does not apply.
- The goal is motif importance interpretation or SHAP visualization rather than accuracy
  assessment — use save_shap_multi()/shapPlots_multi() instead.
- Elements are proximal promoters or the regulatory activity depends on chromatin
  context beyond the input sequence, since BOM is constrained to cis-regulatory information
  within the input sequence.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
tools:
- name: Bag-of-Motifs (BOM)
  role: Provides binModel() for training and predict_binary_multi() for classifying
    CREs and computing correct-classification rates
  repo: https://github.com/ewonglab/BOM_package
- name: FIMO
  role: Scans CRE sequences for TF motifs used as classification features (via BagOfMotifs::runFIMO)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-65362-2
    title: Motif-based models accurately predict cell type-specific distal regulatory
      elements
  source_content_hash: sha256:858b8a2cea53c236cca2563304ad3f8efd329479a4551e4c26bdd77f0d590748
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ewonglab/BOM_package
  tools:
  - Bag-of-Motifs (BOM)
  - FIMO
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cell-type-classification-performance-evaluation

## Summary

Evaluates motif-based (Bag-of-Motifs/XGBoost) classifiers of cell-type-specific distal regulatory elements by computing correct-classification rates on held-out or cross-species CRE sets, quantifying how often BOM models assign each element to its true cell type.

## When to use

Use when you have trained cell-type CRE classifiers (e.g., BOM/XGBoost on motif counts) and need to quantify prediction accuracy, such as fraction of elements assigned to the correct cell type or cross-species transfer rates.

## When NOT to use

- Input is already a feature/model output table and no cell-type labels or classifier predictions are involved — evaluation of classification rates does not apply.
- The goal is motif importance interpretation or SHAP visualization rather than accuracy assessment — use save_shap_multi()/shapPlots_multi() instead.
- Elements are proximal promoters or the regulatory activity depends on chromatin context beyond the input sequence, since BOM is constrained to cis-regulatory information within the input sequence.

## Inputs

- BED-like text file with four columns: chromosome, start, stop, condition (cell type label)
- Mouse-trained BOM models built with BagOfMotifs::binModel()
- FIMO motif scan results generated with runFIMO against gimme.vertebrate.v5.0.meme
- Genome build for sequence extraction (e.g., BSgenome.Mmusculus.UCSC.mm10 or genome of choice)
- LiftOver chain files for human-to-mouse coordinate mapping

## Outputs

- Per-CRE class predictions from predict_binary_multi()
- Correct-classification rates per cell type (e.g., 64.5% cardiomyocyte, 62.9% erythroid)
- liftOver alignment summary: fraction of correctly predicted CREs with no detectable mouse-human alignment at minMatch 0.95
- Compiled results table (cross_species_predictions.csv)

## How to apply

Run BagOfMotifs::predict_binary_multi() to classify each input CRE (formatted as chromosome, start, stop, condition) against the trained cardiomyocyte/erythroid models and compute correct-classification rates. In the cross-species setting, apply mouse-trained BOM models to human fetal CRE sequences extracted with generateAllFasta from the appropriate genome build and scanned with FIMO using gimme.vertebrate.v5.0.meme (set FIMO_updated = FALSE for FIMO < 5.5.4). Judge success against the reported benchmarks: mouse-trained models correctly identified 64.5% of human cardiomyocyte and 62.9% of human erythroblast CREs. Optionally, map correctly predicted elements with UCSC liftOver (minMatch = 0.95, then 0.6) to quantify the fraction of correct predictions lacking detectable mouse-human alignment (90.1% cardiomyocyte, 96.6% erythroid), demonstrating motif-based prediction beyond sequence alignment. Compile rates and liftOver summaries into a results table such as cross_species_predictions.csv.

## Related tools

- **Bag-of-Motifs (BOM)** (Provides binModel() for training and predict_binary_multi() for classifying CREs and computing correct-classification rates) — https://github.com/ewonglab/BOM_package
- **FIMO** (Scans CRE sequences for TF motifs used as classification features (via BagOfMotifs::runFIMO))

## Examples

```
library(BagOfMotifs); library("BSgenome.Mmusculus.UCSC.mm10"); BagOfMotifs::runFIMO(motifs_path = motifs_path, FIMO_path = FIMO_path); BagOfMotifs::predict_binary_multi()
```

## Evaluation signals

- Correct-classification rate for human cardiomyocyte CREs approximates 64.5% under mouse-trained models
- Correct-classification rate for human erythroblast CREs approximates 62.9% under mouse-trained models
- Fraction of correctly predicted CREs lacking detectable mouse-human alignment (liftOver minMatch 0.95) is 90.1% (cardiomyocyte) and 96.6% (erythroblast)
- Every input CRE carries exactly one predicted class from predict_binary_multi() that can be compared to its condition label
- Results table (e.g., cross_species_predictions.csv) contains both correct-classification rates and the liftOver alignment summary

## Limitations

- Unknown how much of the remaining genome regions may share similar motif composition yet not share regulatory activity
- BOM is constrained to cis-regulatory information within the input sequence; open chromatin status, histone marks, and enhancer activity can be influenced by regions beyond the sequence length used
- Reliance on known TF motifs neglects informative sequences not captured by current motif annotations, and TFs do not bind all their motifs

## Evidence

- [results] Mouse-trained BOM models correctly identified 64.5% of human cardiomyocyte and 62.9% of human erythroblast CREs: [evidence span withheld — anchor status: not_found]
- [results] Motif-based cross-species prediction beyond sequence alignment: [evidence span withheld — anchor status: not_found]
- [methods] Prediction and performance workflow step: "BagOfMotifs::predict_binary_multi()"
- [readme] BOM is an R package using XGBoost to classify cis-regulatory sequences: "BAG-of-Motifs (BOM) is an R package that uses XGBoost to construct predictive models of cis-regulatory sequences between cell states."
- [abstract] Combining motif counts with gradient-boosted trees enables accurate prediction of cell-type-specific enhancers: "This minimalist representation, combined with gradient-boosted trees, enables the accurate prediction of cell-type-specific enhancers across mouse, human, zebrafish, and Arabidopsis datasets."
