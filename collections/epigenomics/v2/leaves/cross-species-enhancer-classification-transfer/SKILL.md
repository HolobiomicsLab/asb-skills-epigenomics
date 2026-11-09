---
name: cross-species-enhancer-classification-transfer
description: Use when you have cell-type-specific CRE models trained in one species
  (e.g., mouse cardiomyocyte/erythroid) and want to classify homologous CREs from
  another species, or to test whether predictions depend on sequence alignment.
when_to_use_negative:
- The goal is within-species enhancer discovery from scratch rather than transferring
  an already-trained cross-species model — use the standard BOM training workflow
  (binModel) instead
- Input is already a motif-count feature table rather than genomic coordinates needing
  sequence extraction (generateAllFasta) and FIMO scanning
- The elements of interest depend on regulatory information beyond the input sequence
  window, since BOM is constrained to cis-regulatory information within the input
  sequence
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_0622
tools:
- name: Bag-of-Motifs (BOM)
  role: R package providing generateAllFasta, runFIMO, binModel, and predict_binary_multi
    used to extract target-species CRE sequences, scan motifs, and apply mouse-trained
    XGBoost models cross-species
  repo: https://github.com/ewonglab/BOM_package
- name: FIMO
  role: Scans human CRE sequences for TF binding motifs from gimme.vertebrate.v5.0.meme
    to build motif-count representations
  repo: https://meme-suite.org/meme/doc/install.html
- name: GimmeMotifs
  role: Source of the gimme.vertebrate.v5.0.meme motif library used for FIMO scanning
  repo: https://github.com/vanheeringen-lab/gimmemotifs
- name: UCSC liftOver
  role: Maps correctly predicted human CRE coordinates to mouse at minMatch 0.95 and
    0.6 to test dependence on sequence alignment
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-65362-2
    title: Motif-based models accurately predict cell type-specific distal regulatory
      elements
  source_content_hash: sha256:0882e594eed71317892ebe477f461e411c7d13f80edfa47dc11c4cfd976fabe4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ewonglab/BOM_package
  tools:
  - Bag-of-Motifs (BOM)
  - FIMO
  - GimmeMotifs
  - UCSC liftOver
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# cross-species-enhancer-classification-transfer

## Summary

Applies mouse-trained Bag-of-Motifs (BOM) models to classify homologous CREs from another species (e.g., human fetal cardiomyocyte and erythroblast elements), demonstrating that motif-count-based prediction transfers across species even when no detectable sequence alignment exists.

## When to use

Use when you have cell-type-specific CRE models trained in one species (e.g., mouse cardiomyocyte/erythroid) and want to classify homologous CREs from another species, or to test whether predictions depend on sequence alignment.

## When NOT to use

- The goal is within-species enhancer discovery from scratch rather than transferring an already-trained cross-species model — use the standard BOM training workflow (binModel) instead
- Input is already a motif-count feature table rather than genomic coordinates needing sequence extraction (generateAllFasta) and FIMO scanning
- The elements of interest depend on regulatory information beyond the input sequence window, since BOM is constrained to cis-regulatory information within the input sequence

## Inputs

- Mouse-trained BOM models (from BagOfMotifs::binModel())
- Target-species CRE BED-like text file with four columns: chromosome, start, stop, condition
- Target-species genome build (e.g., BSgenome equivalent of the genome of choice, analogous to BSgenome.Mmusculus.UCSC.mm10)
- gimme.vertebrate.v5.0.meme motif library
- FIMO installation path
- liftOver chain files for human-to-mouse coordinate mapping (UCSC)

## Outputs

- Per-CRE classification results from predict_binary_multi() with correct-classification rates (expected ~64.5% cardiomyocyte, ~62.9% erythroblast)
- liftOver alignment summary quantifying correctly predicted CREs with no detectable mouse-human alignment (expected ~90.1% cardiomyocyte, ~96.6% erythroblast at minMatch = 0.95)
- cross_species_predictions.csv compiling classification rates and liftOver alignment results

## How to apply

Load mouse-trained BOM models (built with BagOfMotifs::binModel() on XGBoost over unordered TF motif counts) and format the target-species CRE coordinates as a four-column text file (chromosome, start, stop, condition). Extract target CRE sequences with generateAllFasta using the matching genome build (e.g., the human fetal genome instead of Mmusculus), then scan them for motifs with FIMO via BagOfMotifs::runFIMO using gimme.vertebrate.v5.0.meme (set FIMO_updated = FALSE for FIMO < 5.5.4). Classify each element with BagOfMotifs::predict_binary_multi() against the mouse-trained models and compute correct-classification rates. Map correctly predicted CRE coordinates back to the training species with UCSC liftOver, first at minMatch = 0.95 then at minMatch = 0.6, to quantify how many predictions lack detectable cross-species alignment — this tests whether transfer relies on motif composition rather than conserved sequence. Rationale: BOM represents each distal element as unordered motif counts independent of motif order, orientation, or spacing, so motif-based codes can generalize across species boundaries.

## Related tools

- **Bag-of-Motifs (BOM)** (R package providing generateAllFasta, runFIMO, binModel, and predict_binary_multi used to extract target-species CRE sequences, scan motifs, and apply mouse-trained XGBoost models cross-species) — https://github.com/ewonglab/BOM_package
- **FIMO** (Scans human CRE sequences for TF binding motifs from gimme.vertebrate.v5.0.meme to build motif-count representations) — https://meme-suite.org/meme/doc/install.html
- **GimmeMotifs** (Source of the gimme.vertebrate.v5.0.meme motif library used for FIMO scanning) — https://github.com/vanheeringen-lab/gimmemotifs
- **UCSC liftOver** (Maps correctly predicted human CRE coordinates to mouse at minMatch 0.95 and 0.6 to test dependence on sequence alignment)

## Examples

```
library(BagOfMotifs); generateAllFasta(bedDir="path/to/human_CREs", genome="Human"); BagOfMotifs::runFIMO(motifs_path=motifs_path, FIMO_path=FIMO_path); BagOfMotifs::predict_binary_multi()
```

## Evaluation signals

- Correct-classification rates of human fetal CREs by mouse-trained models fall near the reported values: 64.5% (cardiomyocyte) and 62.9% (erythroblast)
- Fraction of correctly predicted CREs with no detectable mouse-human alignment at liftOver minMatch = 0.95 approaches 90.1% (cardiomyocyte) and 96.6% (erythroblast)
- Input file conforms to the required four-column format (chromosome, start, stop, condition) before generateAllFasta
- FIMO version compatibility handled correctly: FIMO_updated = FALSE for FIMO < 5.5.4
- Outputs compiled into cross_species_predictions.csv with both classification rates and liftOver alignment summaries

## Limitations

- Unknown how much of the remaining genome may share similar motif composition yet not share regulatory activity
- BOM is constrained to cis-regulatory information within the input sequence; open chromatin status, histone marks, and enhancer activity can be influenced by regions beyond the sequence length used
- Reliance on known TF motifs neglects informative sequences not captured by current motif annotations, and TFs do not bind all their motifs
- FIMO version below 5.5.4 requires FIMO_updated = FALSE, otherwise motif scanning may fail

## Evidence

- [results] Mouse-trained BOM models correctly identified 64.5% of human cardiomyocyte and 62.9% of human erythroblast CREs: [evidence span withheld — anchor status: not_found]
- [abstract] This minimalist representation, combined with gradient-boosted trees, enables the accurate prediction of cell-type-specific enhancers across mouse, human, zebrafish, and Arabidopsis datasets.: "This minimalist representation, combined with gradient-boosted trees, enables the accurate prediction of cell-type-specific enhancers across mouse, human, zebrafish, and Arabidopsis datasets."
- [readme] Input is a text file with four columns (chromosome, start, stop, condition): "Input is a text file with four columns (chromosome, start, stop, condition)"
- [methods] generateAllFasta genome of choice: "library("BSgenome.Mmusculus.UCSC.mm10") # or genome of choice"
- [discussion] BOM is constrained to cis-regulatory information within the input sequence: "BOM is constrained to cis-regulatory information within the input sequence. The status of open chromatin, histone marks, and enhancer activity can be influenced by regions beyond the sequence length"
