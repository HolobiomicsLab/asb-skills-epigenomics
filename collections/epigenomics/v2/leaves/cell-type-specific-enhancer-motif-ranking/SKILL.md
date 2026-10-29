---
name: cell-type-specific-enhancer-motif-ranking
description: Use when you have trained BOM/XGBoost classifiers on cell-type-specific
  distal regulatory elements (e.g. mouse E8.25 scATAC-derived CREs) and need per-motif
  SHAP importance rankings plus local explanations at a locus of interest.
when_to_use_negative:
- You need motif grammar (spacing/orientation/order) effects — BOM represents sequences
  as unordered motif counts and grammar integration remains future work
- Regulatory activity depends on regions beyond the input sequence length or on open
  chromatin/histone marks — BOM is constrained to cis-regulatory information within
  the input sequence
- The relevant TFs lack known PWMs in the motif library — reliance on known TF motifs
  neglects informative sequences not captured by current motif annotations
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3169
tools:
- name: Bag-of-Motifs (BOM)
  role: R package providing binModel() (XGBoost training on motif counts), save_shap_multi()
    (SHAP estimation) and shapPlots_multi() (bar/beeswarm/waterfall plots) used to
    rank motif importance
  repo: https://github.com/ewonglab/BOM_package
- name: FIMO
  role: Scans distal regulatory sequences against the gimme.vertebrate.v5.0.meme motif
    library to produce the motif count features fed to the classifier
- name: Gimme Maelstrom
  role: Source of the gimme.vertebrate.v5.0.meme motif library used for FIMO scanning;
    noted in the article as an ensemble motif-detection alternative lacking per-sequence
    interpretability
  repo: https://github.com/vanheeringen-lab/gimmemotifs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-65362-2
    title: Motif-based models accurately predict cell type-specific distal regulatory
      elements
  source_content_hash: sha256:9a83ef6694cd83df5726be6dde4e0601dbd46eda1ad2e0502e2ef15f45e3c5a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ewonglab/BOM_package
  tools:
  - Bag-of-Motifs (BOM)
  - FIMO
  - Gimme Maelstrom
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cell-type-specific-enhancer-motif-ranking

## Summary

Ranks TF binding motifs by their contribution to cell-type-specific cis-regulatory element classification using SHAP values from BOM's gradient-boosted tree (XGBoost) models built on FIMO motif count matrices, and validates that the rankings correspond to known lineage-specific TFs (e.g. MEF2C/NKX2-5/GATA for cardiomyocytes, FLI1/JUN-FOS/SOX7-ETV6 for endothelial cells).

## When to use

Use when you have trained BOM/XGBoost classifiers on cell-type-specific distal regulatory elements (e.g. mouse E8.25 scATAC-derived CREs) and need per-motif SHAP importance rankings plus local explanations at a locus of interest.

## When NOT to use

- You need motif grammar (spacing/orientation/order) effects — BOM represents sequences as unordered motif counts and grammar integration remains future work
- Regulatory activity depends on regions beyond the input sequence length or on open chromatin/histone marks — BOM is constrained to cis-regulatory information within the input sequence
- The relevant TFs lack known PWMs in the motif library — reliance on known TF motifs neglects informative sequences not captured by current motif annotations

## Inputs

- Trained BOM gradient-boosted tree (XGBoost) models from BagOfMotifs::binModel()
- FIMO motif count matrices generated with runFIMO() using gimme.vertebrate.v5.0.meme
- BED file of distal regulatory elements with chromosome, start, stop, condition columns
- Cell-type condition labels (e.g. mouse E8.25 cardiomyocyte, endothelial, neural crest)

## Outputs

- Per-motif SHAP importance rankings for each cell-type condition
- Bar, beeswarm and waterfall SHAP plots from shapPlots_multi()
- Local SHAP explanations at specific loci (e.g. the Nkx2-5 locus showing MEF2/SRF dominance)
- Qualitative mapping of top motifs to lineage-specific TFs (MEF2C, NKX2-5, GATA, FLI1, JUN/FOS, SOX7/ETV6, TFAP2A)

## How to apply

Load the trained BOM models and FIMO motif count matrices produced by BagOfMotifs::binModel() in R, which scanned sequences with FIMO against the gimme.vertebrate.v5.0.meme motif library. Run save_shap_multi() to estimate per-motif SHAP values from the gradient-boosted tree classifiers for each cell-type condition, using the shapviz-based SHAP machinery. Verify the SHAP-derived motif rankings against the reported lineage associations: MEF2C, NKX2-5 and GATA motifs dominating cardiomyocyte CREs; FLI1, JUN/FOS family members and SOX7/ETV6 dominating endothelial CREs; MEF2/SRF dominance at the Nkx2-5 locus; and TFAP2A for neural crest. Visualize and inspect explanations with shapPlots_multi() (bar plots), shapPlots_multi(plotType = "beeswarm") for global distributions, and shapPlots_multi(plotType = "waterfall") for local explanations at individual loci such as the cardiac regulator Nkx2-5. Success is judged by qualitative reproduction of the paper's cell-type-specific motif associations in the resulting rankings.

## Related tools

- **Bag-of-Motifs (BOM)** (R package providing binModel() (XGBoost training on motif counts), save_shap_multi() (SHAP estimation) and shapPlots_multi() (bar/beeswarm/waterfall plots) used to rank motif importance) — https://github.com/ewonglab/BOM_package
- **FIMO** (Scans distal regulatory sequences against the gimme.vertebrate.v5.0.meme motif library to produce the motif count features fed to the classifier)
- **Gimme Maelstrom** (Source of the gimme.vertebrate.v5.0.meme motif library used for FIMO scanning; noted in the article as an ensemble motif-detection alternative lacking per-sequence interpretability) — https://github.com/vanheeringen-lab/gimmemotifs

## Examples

```
save_shap_multi(); shapPlots_multi(); shapPlots_multi(plotType = "beeswarm"); shapPlots_multi(plotType = "waterfall")
```

## Evaluation signals

- Cardiomyocyte CRE SHAP rankings are dominated by MEF2C, NKX2-5 and GATA motifs
- Endothelial top-20 SHAP ranking contains FLI1, JUN/FOS family members and SOX7/ETV6 motifs
- Local waterfall explanation at the Nkx2-5 locus shows MEF2/SRF motif dominance
- Neural crest rankings surface TFAP2A motifs
- shapPlots_multi() generates all three plot types (bar, beeswarm, waterfall) without errors from the saved SHAP objects

## Limitations

- BOM is constrained to cis-regulatory information within the input sequence; open chromatin status, histone marks, and enhancer activity can be influenced by regions beyond the sequence length used in BOM
- Reliance on known TF motifs neglects informative sequences not captured by current motif annotations, and TFs do not bind all their motifs
- Unknown how much of the remaining genome regions may share similar motif composition yet not share regulatory activity
- Integration with motif grammar remains an important future direction not yet done

## Evidence

- [results] SHAP motif rankings reproduce lineage associations: [evidence span withheld — anchor status: not_found]
- [readme] SHAP workflow commands: "# Estimate SHAP 
save_shap_multi()

# Plots (bar, beeswarm, waterfall)
shapPlots_multi()
shapPlots_multi(plotType = "beeswarm")"
- [readme] BOM interpretation principle: "Through the use of SHapley Additive exPlanations (SHAP), BOM helps identify the important TF binding motifs that contribute to the classification."
- [methods] Motif library input: "motifs_path <- paste0(extdata_path,"/gimme.vertebrate.v5.0.meme")"
- [discussion] BOM scope limitation: "BOM is constrained to cis-regulatory information within the input sequence."
