---
name: gradient-boosted-tree-interpretation
description: Use when you have a trained motif-count classifier (e.g., XGBoost on
  FIMO-derived motif counts) and need per-feature SHAP attributions to rank motifs
  and link them to lineage-specific TFs.
when_to_use_negative:
- When sequences contain regulatory information not captured by known TF motif annotations
  — BOM's reliance on known motifs neglects informative sequences outside current
  motif libraries
- When enhancer activity depends on regions beyond the input sequence length or on
  open chromatin/histone marks not encoded in motif counts — BOM is constrained to
  cis-regulatory information within the input sequence
- When the input is already a feature table unrelated to motif counts (e.g., gene
  expression or epigenomic signal tracks) — SHAP interpretation of motif-count models
  is not applicable
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_0203
tools:
- name: Bag-of-Motifs (BOM)
  role: R package providing save_shap_multi() for SHAP estimation and shapPlots_multi()
    for bar/beeswarm/waterfall visualization of motif importance in the XGBoost classifier
  repo: https://github.com/ewonglab/BOM_package
- name: FIMO
  role: Scans distal CRE sequences against the gimme.vertebrate.v5.0.meme motif library
    to produce the motif count matrices that serve as SHAP-interpreted features
- name: shapviz
  role: R dependency underlying SHAP value computation and visualization in the BOM
    package
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-65362-2
    title: Motif-based models accurately predict cell type-specific distal regulatory
      elements
  source_content_hash: sha256:b2d894790aae2aae8d5ddf310871e19c0df8305b1e4deb05d060070011423338
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ewonglab/BOM_package
  tools:
  - Bag-of-Motifs (BOM)
  - FIMO
  - Gimme Maelstrom
  - shapviz
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# gradient-boosted-tree-interpretation

## Summary

Interprets gradient-boosted tree (XGBoost) classifiers of cis-regulatory sequences by computing SHAP values per motif feature, revealing which TF binding motifs drive cell-type-specific CRE classification. Grounded in the Bag-of-Motifs (BOM) framework, it converts motif-count models into biologically interpretable motif-importance rankings.

## When to use

Use when you have a trained motif-count classifier (e.g., XGBoost on FIMO-derived motif counts) and need per-feature SHAP attributions to rank motifs and link them to lineage-specific TFs.

## When NOT to use

- When sequences contain regulatory information not captured by known TF motif annotations — BOM's reliance on known motifs neglects informative sequences outside current motif libraries
- When enhancer activity depends on regions beyond the input sequence length or on open chromatin/histone marks not encoded in motif counts — BOM is constrained to cis-regulatory information within the input sequence
- When the input is already a feature table unrelated to motif counts (e.g., gene expression or epigenomic signal tracks) — SHAP interpretation of motif-count models is not applicable

## Inputs

- Trained BOM gradient-boosted tree (XGBoost) models produced by BagOfMotifs::binModel()
- FIMO motif count matrices from scanning sequences with the gimme.vertebrate.v5.0.meme motif library
- Cell-type condition labels (e.g., cardiomyocyte, endothelial, neural crest) from the input BED file with chromosome, start, stop, condition columns
- Optionally, genomic annotations (Mus_musculus.GRCm38.92.gtf.gz) for local explanations at specific loci such as Nkx2-5

## Outputs

- Per-motif SHAP importance values for each cell-type condition (save_shap_multi() output)
- Bar plots, beeswarm plots, and waterfall plots of SHAP explanations via shapPlots_multi()
- Local SHAP explanations (waterfall) attributing classification at individual CREs/loci (e.g., Nkx2-5) to specific motifs
- Qualitative motif-to-TF association rankings (e.g., MEF2C/NKX2-5/GATA for cardiomyocytes; FLI1/JUN-FOS/SOX7-ETV6 for endothelial)

## How to apply

After training BOM models with binModel() on FIMO motif count matrices built from the gimme.vertebrate.v5.0.meme library, run save_shap_multi() to estimate per-motif SHAP values from the gradient-boosted tree classifiers for each cell-type condition. Then use shapPlots_multi() to generate bar plots, shapPlots_multi(plotType = "beeswarm") for beeswarm plots, and shapPlots_multi(plotType = "waterfall") for local explanations of individual loci (e.g., the Nkx2-5 cardiac regulator). Validate the SHAP-derived motif rankings qualitatively against known biology: cardiomyocyte CREs should be dominated by MEF2C, NKX2-5 and GATA motifs, endothelial CREs by FLI1, JUN/FOS and SOX7/ETV6, and neural crest by TFAP2A. The rationale is that BOM's unordered motif-count representation, combined with XGBoost and SHAP, provides direct interpretability without needing per-sequence deep-learning attribution methods.

## Related tools

- **Bag-of-Motifs (BOM)** (R package providing save_shap_multi() for SHAP estimation and shapPlots_multi() for bar/beeswarm/waterfall visualization of motif importance in the XGBoost classifier) — https://github.com/ewonglab/BOM_package
- **FIMO** (Scans distal CRE sequences against the gimme.vertebrate.v5.0.meme motif library to produce the motif count matrices that serve as SHAP-interpreted features)
- **shapviz** (R dependency underlying SHAP value computation and visualization in the BOM package)

## Examples

```
save_shap_multi(); shapPlots_multi(plotType = "waterfall")
```

## Evaluation signals

- SHAP-based motif rankings qualitatively reproduce reported cell-type-specific associations: MEF2C, NKX2-5 and GATA motifs dominate cardiomyocyte CREs; FLI1, JUN/FOS family and SOX7/ETV6 dominate endothelial CREs; TFAP2A appears for neural crest
- Local waterfall explanation at the Nkx2-5 locus shows MEF2/SRF motif dominance, matching the paper's local-explanation results
- Model accuracy verified first via predict_binary_multi() before trusting SHAP attributions
- SHAP plots render for all three plot types (bar, beeswarm, waterfall) without errors for every cell-type condition in the model set

## Limitations

- TFs do not bind all their motifs, so SHAP motif importance may not perfectly map to TF activity
- BOM is constrained to cis-regulatory information within the input sequence; open chromatin, histone marks, and enhancer activity can be influenced by regions beyond the sequence length used
- Reliance on known TF motifs neglects informative sequences not captured by current motif annotations
- Unknown how much of the remaining genome regions may share similar motif composition yet not share regulatory activity

## Evidence

- [results] SHAP analysis showed that motif importance corresponded to lineage-specific TFs: cardiomyocyte CREs were dominated by MEF2C, NKX2-5 and GATA motifs, while endothelial CREs were dominated by FLI1, JUN/FOS family members and SOX7/ETV6 motifs.: [evidence span withheld — anchor status: not_found]
- [readme] README SHAP workflow: "# Estimate SHAP 
save_shap_multi()

# Plots (bar, beeswarm, waterfall)
shapPlots_multi()
shapPlots_multi(plotType = "beeswarm")
shapPlots_multi(plotType = "waterfall")"
- [readme] BOM interprets motif importance with SHAP scores: "It then interprets the importance of each motif in the model using SHAP scores."
- [intro] Motif-centric representation linked to gene expression: "the motif-centric representation yields a vocabulary of regulatory features that is biologically interpretable and can be linked directly to gene expression"
- [discussion] BOM constrained to cis-regulatory information: "BOM is constrained to cis-regulatory information within the input sequence."
