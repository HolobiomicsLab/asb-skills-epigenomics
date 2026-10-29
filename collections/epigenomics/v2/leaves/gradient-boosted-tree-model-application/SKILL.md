---
name: gradient-boosted-tree-model-application
description: Use when you need to classify context-specific cis-regulatory elements
  from sequence using motif-count features with a tree ensemble (XGBoost), e.g. distinguishing
  cardiomyocyte vs erythroid CREs or transferring models across species.
when_to_use_negative:
- When regulatory activity depends on features beyond the input sequence (open chromatin
  status, histone marks, distal enhancer–promoter effects outside the sequence length),
  since BOM is constrained to cis-regulatory information within the input sequence.
- When informative sequences are not captured by known TF motif annotations, as reliance
  on known PWMs neglects unannotated sequence patterns.
- When the goal requires motif grammar (order, orientation, spacing) modeling — motif
  grammar integration is an explicitly unimplemented future direction.
edam_operation: http://edamontology.org/operation_0445
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
tools:
- name: Bag-of-Motifs (BOM)
  role: R package providing binModel(), predict_binary_multi(), save_shap_multi()
    and shapPlots_multi() to build, apply, and interpret XGBoost models on motif counts
  repo: https://github.com/ewonglab/BOM_package
- name: FIMO
  role: Scans extracted CRE sequences for TF motif occurrences (via BagOfMotifs::runFIMO)
    to generate the motif-count features consumed by the tree model
- name: GimmeMotifs
  role: Source of the gimme.vertebrate.v5.0.meme motif library used by FIMO scanning
  repo: https://github.com/vanheeringen-lab/gimmemotifs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-65362-2
    title: Motif-based models accurately predict cell type-specific distal regulatory
      elements
  source_content_hash: sha256:912add4a616d9b029f21f582c1a68a0b3332843872ffd2865079310b0f6066c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ewonglab/BOM_package
  tools:
  - Bag-of-Motifs (BOM)
  - FIMO
  - GimmeMotifs
  edam_operation: http://edamontology.org/operation_0445
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# gradient-boosted-tree-model-application

## Summary

Train and apply gradient-boosted tree (XGBoost) classifiers on unordered TF motif-count features of distal cis-regulatory elements, as implemented in the Bag-of-Motifs (BOM) R package, to classify cell-type-specific CREs and interpret motif importance via SHAP.

## When to use

Use when you need to classify context-specific cis-regulatory elements from sequence using motif-count features with a tree ensemble (XGBoost), e.g. distinguishing cardiomyocyte vs erythroid CREs or transferring models across species.

## When NOT to use

- When regulatory activity depends on features beyond the input sequence (open chromatin status, histone marks, distal enhancer–promoter effects outside the sequence length), since BOM is constrained to cis-regulatory information within the input sequence.
- When informative sequences are not captured by known TF motif annotations, as reliance on known PWMs neglects unannotated sequence patterns.
- When the goal requires motif grammar (order, orientation, spacing) modeling — motif grammar integration is an explicitly unimplemented future direction.

## Inputs

- Text/BED-like file with four columns: chromosome, start, stop, condition
- Genome FASTA via BSgenome package (e.g. BSgenome.Mmusculus.UCSC.mm10, 'or genome of choice')
- Motif library PWM file (gimme.vertebrate.v5.0.meme from BOM extdata)
- Installed FIMO binary path (MEME Suite)
- Chromosome sizes file (e.g. mouse.chrom.sizes.txt) and GTF annotation (e.g. Mus_musculus.GRCm38.92.gtf.gz)

## Outputs

- Trained XGBoost-based BOM model object classifying cell-type-specific CREs
- Per-sequence class predictions and performance metrics from predict_binary_multi()
- Per-motif SHAP importance scores (save_shap_multi())
- SHAP plots: bar, beeswarm, waterfall (shapPlots_multi())

## How to apply

Represent each distal regulatory sequence as an unordered vector of TF motif counts (independent of motif order, orientation, or spacing): format input as a 4-column text file (chromosome, start, stop, condition), extract sequences with generateAllFasta against the appropriate BSgenome (e.g. Mmusculus/mm10), scan motifs with FIMO using gimme.vertebrate.v5.0.meme (set FIMO_updated = FALSE for FIMO < 5.5.4), then train with BagOfMotifs::binModel() which combines motif counts with XGBoost gradient-boosted trees. Apply trained models to new sequences with BagOfMotifs::predict_binary_multi() to obtain binary/multi-class predictions and performance metrics. Interpret the fitted model with save_shap_multi() and visualize per-motif contributions using shapPlots_multi() (bar, beeswarm, waterfall). Rationale: motif counts plus gradient boosting capture combinatorial TF motif contributions while remaining directly interpretable, and mouse-trained models transferred to human fetal cardiomyocyte/erythroblast CREs (64.5% and 62.9% correct classification) even without detectable liftOver alignment.

## Related tools

- **Bag-of-Motifs (BOM)** (R package providing binModel(), predict_binary_multi(), save_shap_multi() and shapPlots_multi() to build, apply, and interpret XGBoost models on motif counts) — https://github.com/ewonglab/BOM_package
- **FIMO** (Scans extracted CRE sequences for TF motif occurrences (via BagOfMotifs::runFIMO) to generate the motif-count features consumed by the tree model)
- **GimmeMotifs** (Source of the gimme.vertebrate.v5.0.meme motif library used by FIMO scanning) — https://github.com/vanheeringen-lab/gimmemotifs

## Examples

```
library(BagOfMotifs); generateAllFasta(bedDir="path/to/input", genome="Mmusculus"); BagOfMotifs::runFIMO(motifs_path=motifs_path, FIMO_path=FIMO_path); BagOfMotifs::binModel(); BagOfMotifs::predict_binary_multi()
```

## Evaluation signals

- Correct-classification rate of held-out/homologous CREs reaches expected ranges: 64.5% for human cardiomyocyte and 62.9% for human erythroblast with mouse-trained models.
- predict_binary_multi() returns per-sequence class assignments consistent with the 'condition' column of the 4-column input.
- SHAP output from save_shap_multi() covers all motifs in the model and shapPlots_multi() renders bar, beeswarm, and waterfall plot types without errors.
- FIMO scan (runFIMO) completes against the specified motif library (gimme.vertebrate.v5.0.meme) and motif counts are finite, non-negative integers per sequence.
- Cross-species check: most correctly predicted human CREs (90.1% cardiomyocyte, 96.6% erythroblast) lack detectable mouse–human alignment at liftOver minMatch = 0.95, confirming motif-based (not alignment-based) prediction.

## Limitations

- Unknown how much of the remaining genome may share similar motif composition yet not share regulatory activity.
- BOM is constrained to cis-regulatory information within the input sequence; open chromatin status, histone marks, and enhancer activity can be influenced by regions beyond the sequence length used.
- Reliance on known TF motifs neglects informative sequences not captured by current motif annotations, and TFs do not bind all their motifs.
- Performance of sequence-based deep learning alternatives is limited by training data availability for many cell states; BOM itself has not yet integrated motif grammar.

## Evidence

- [abstract] This minimalist representation, combined with gradient-boosted trees, enables the accurate prediction of cell-type-specific enhancers across mouse, human, zebrafish, and Arabidopsis datasets.: "This minimalist representation, combined with gradient-boosted trees, enables the accurate prediction of cell-type-specific enhancers across mouse, human, zebrafish, and Arabidopsis datasets."
- [results] Mouse-trained BOM models correctly identified 64.5% of human cardiomyocyte and 62.9% of human erythroblast CREs: [evidence span withheld — anchor status: not_found]
- [intro] BOM represents each distal regulatory sequence as an unordered set of motif counts, independent of motif order, orientation, or spacing: "representing each distal regulatory sequence as an unordered set of motif counts, independent of motif order, orientation, or spacing"
- [readme] devtools::install_github("ewonglab/BOM_package"): "devtools::install_github("ewonglab/BOM_package")"
- [readme] BOM is an R package that uses XGBoost to construct predictive models of cis-regulatory sequences between cell states. It then interprets the importance of each motif in the model using SHAP scores.: "BAG-of-Motifs (BOM) is an R package that uses XGBoost to construct predictive models of cis-regulatory sequences between cell states. It then interprets the importance of each motif in the model"
