---
name: multi-species-enhancer-benchmarking
description: Use when asked to reproduce or extend classification of cell-type-specific
  enhancers across species, cell types, or disease states using BED coordinates plus
  condition labels and the BOM pipeline.
when_to_use_negative:
- The task requires modeling motif grammar (order, orientation, or spacing) — BOM
  represents sequences as unordered motif counts and grammar integration is noted
  as future work
- The regulatory activity may depend on sequence beyond the input regions or on open
  chromatin/histone marks — BOM is constrained to cis-regulatory information within
  the input sequence
- The target sequences contain TF binding patterns absent from known motif annotations
  (e.g. gimme.vertebrate.v5.0.meme) — reliance on known motifs neglects informative
  sequences not captured by current annotations
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
tools:
- name: Bag-of-Motifs (BOM)
  role: R package used to generate FASTA, build motif-count matrices, train XGBoost
    classifiers, evaluate predictions, and compute/plot SHAP motif importances
  repo: github.com/ewonglab/BOM_package
- name: FIMO
  role: Scans generated FASTA sequences for TF motif occurrences using the gimme.vertebrate.v5.0.meme
    position weight matrices
- name: GimmeMotifs
  role: Source of the gimme.vertebrate.v5.0.meme motif collection used as the motif
    database for FIMO scanning
  repo: https://github.com/vanheeringen-lab/gimmemotifs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-65362-2
    title: Motif-based models accurately predict cell type-specific distal regulatory
      elements
  source_content_hash: sha256:427c2d82ab088c33c7aa7f41fa5ae1dd4fef847e6b97f6d218e07a22dd1f0f44
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bag-of-Motifs (BOM)
  - BagOfMotifs (BOM)
  - FIMO
  - GimmeMotifs
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# multi-species-enhancer-benchmarking

## Summary

Reproduce BOM's cross-context benchmarking of motif-count-based cis-regulatory element classification across human, zebrafish, Drosophila, Arabidopsis, and AML datasets, verifying that unordered TF motif counts plus gradient-boosted trees generalize beyond the mouse benchmark.

## When to use

Use when asked to reproduce or extend classification of cell-type-specific enhancers across species, cell types, or disease states using BED coordinates plus condition labels and the BOM pipeline.

## When NOT to use

- The task requires modeling motif grammar (order, orientation, or spacing) — BOM represents sequences as unordered motif counts and grammar integration is noted as future work
- The regulatory activity may depend on sequence beyond the input regions or on open chromatin/histone marks — BOM is constrained to cis-regulatory information within the input sequence
- The target sequences contain TF binding patterns absent from known motif annotations (e.g. gimme.vertebrate.v5.0.meme) — reliance on known motifs neglects informative sequences not captured by current annotations

## Inputs

- Four-column BED-like text file (chromosome, start, stop, condition) per dataset with enhancer coordinates and cell-type condition labels
- Species-appropriate BSgenome package (e.g. BSgenome.Mmusculus.UCSC.mm10)
- gimme.vertebrate.v5.0.meme motif file (bundled in BOM extdata)
- FIMO binary from MEME Suite

## Outputs

- F1 and auPR metrics per classification dataset (or Pearson correlation for Drosophila STARR-seq regression)
- Trained XGBoost motif-count classifiers per dataset
- Per-motif SHAP values and bar/beeswarm/waterfall SHAP plots

## How to apply

Prepare each dataset as a four-column text file (chromosome, start, stop, condition) with enhancer coordinates and cell-type condition labels. Generate FASTA sequences with BagOfMotifs::generateAllFasta() against the appropriate BSgenome package for each species, then scan motifs with BagOfMotifs::runFIMO() using gimme.vertebrate.v5.0.meme (set FIMO_updated = FALSE for FIMO <5.5.4). Build motif-count matrices and train gradient-boosted (XGBoost) classifiers with BagOfMotifs::binModel(), evaluate with BagOfMotifs::predict_binary_multi() to obtain F1 and auPR (or Pearson correlation for the Drosophila STARR-seq regression), and compute per-motif SHAP values with save_shap_multi() visualized via shapPlots_multi() in bar, beeswarm, or waterfall form. Validate against reported benchmark values: ENCODE F1 ~0.92/auPR ~0.98, blood F1 ~0.90, zebrafish F1 ~0.96, Drosophila Pearson ~0.78/0.42, AML F1 ~0.99.

## Related tools

- **Bag-of-Motifs (BOM)** (R package used to generate FASTA, build motif-count matrices, train XGBoost classifiers, evaluate predictions, and compute/plot SHAP motif importances) — github.com/ewonglab/BOM_package
- **FIMO** (Scans generated FASTA sequences for TF motif occurrences using the gimme.vertebrate.v5.0.meme position weight matrices)
- **GimmeMotifs** (Source of the gimme.vertebrate.v5.0.meme motif collection used as the motif database for FIMO scanning) — https://github.com/vanheeringen-lab/gimmemotifs

## Examples

```
generateAllFasta(bedDir = "path/to/input", genome = "Mmusculus"); BagOfMotifs::runFIMO(motifs_path = motifs_path, FIMO_path = FIMO_path); BagOfMotifs::binModel(); BagOfMotifs::predict_binary_multi()
```

## Evaluation signals

- Computed ENCODE dataset F1 ~0.92 and auPR ~0.98 within reasonable tolerance
- Blood dataset F1 ~0.90, zebrafish F1 ~0.96, and AML F1 ~0.99 match reported values
- Drosophila STARR-seq regression Pearson correlation ~0.78/0.42 reproduced
- Motif-count matrices are generated against the correct species-specific BSgenome for each dataset
- SHAP output renders in all three plot types (bar, beeswarm, waterfall) without error via shapPlots_multi()

## Limitations

- Unknown how much of the remaining genome may share similar motif composition yet not share regulatory activity
- BOM is constrained to cis-regulatory information within the input sequence; open chromatin, histone marks, and enhancer activity can be influenced by regions beyond the sequence length used
- Reliance on known TF motifs neglects informative sequences not captured by current motif annotations, and TFs do not bind all their motifs
- FIMO_updated = FALSE must be set when the installed FIMO version is <5.5.4

## Evidence

- [abstract] This minimalist representation, combined with gradient-boosted trees, enables the accurate prediction of cell-type-specific enhancers across mouse, human, zebrafish, and Arabidopsis datasets.: "This minimalist representation, combined with gradient-boosted trees, enables the accurate prediction of cell-type-specific enhancers across mouse, human, zebrafish, and Arabidopsis datasets."
- [methods] generateAllFasta call: "generateAllFasta(bedDir = "path/to/input", genome = "Mmusculus")"
- [discussion] BOM constrained to input sequence: "BOM is constrained to cis-regulatory information within the input sequence."
- [readme] BOM package description: "BAG-of-Motifs (BOM) is an R package that uses XGBoost to construct predictive models of cis-regulatory sequences between cell states. It then interprets the importance of each motif in the model"
- [readme] Input format from README: "Input is a text file with four columns (chromosome, start, stop, condition)"
