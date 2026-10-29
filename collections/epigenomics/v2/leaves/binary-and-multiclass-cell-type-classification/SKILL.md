---
name: binary-and-multiclass-cell-type-classification
description: Use when snATAC-seq-derived distal CRE BED files (chromosome, start,
  stop, condition) must be assigned to their cell type of origin using motif-count
  features rather than sequence-based deep learning.
when_to_use_negative:
- The question requires regulatory information beyond the input sequence (e.g. open
  chromatin status or enhancer activity influenced by regions outside the CRE), since
  BOM is constrained to cis-regulatory information within the input sequence.
- The regulatory code depends on motif order, orientation, or spacing (motif grammar),
  which the unordered bag-of-motifs representation ignores.
- Sequences of interest contain TF binding patterns not captured by current motif
  annotations in the provided MEME file, since reliance on known TF motifs neglects
  informative sequences.
edam_operation: http://edamontology.org/operation_0436
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3173
tools:
- name: Bag-of-Motifs (BOM)
  role: R package providing generateAllFasta, runFIMO, binModel and predict_binary_multi
    to build and evaluate the motif-count XGBoost classifiers
  repo: https://github.com/ewonglab/BOM_package
- name: FIMO
  role: Scans the CRE FASTA sequences against gimme.vertebrate.v5.0.meme to produce
    motif occurrences counted per sequence
  repo: https://meme-suite.org/meme/doc/install.html
- name: XGBoost
  role: Gradient-boosted tree algorithm underlying the binary and multiclass cell-type
    classification models
- name: GimmeMotifs
  role: Source of the gimme.vertebrate.v5.0.meme motif database used for scanning
  repo: https://github.com/vanheeringen-lab/gimmemotifs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-65362-2
    title: Motif-based models accurately predict cell type-specific distal regulatory
      elements
  source_content_hash: sha256:4bc57b6ea5436a9a350c86d5cbec6b5de2a1fcb9221ac4452251b0c151adbff0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ewonglab/BOM_package
  tools:
  - Bag-of-Motifs (BOM)
  - FIMO
  - GimmeMotifs
  - XGBoost
  edam_operation: http://edamontology.org/operation_0436
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# binary-and-multiclass-cell-type-classification

## Summary

Trains XGBoost gradient-boosted tree models on bag-of-motifs (unordered motif counts) representations of distal CREs to classify cell-type-specific regulatory elements, both pairwise binary and multiclass, as implemented in the BOM R package.

## When to use

Use when snATAC-seq-derived distal CRE BED files (chromosome, start, stop, condition) must be assigned to their cell type of origin using motif-count features rather than sequence-based deep learning.

## When NOT to use

- The question requires regulatory information beyond the input sequence (e.g. open chromatin status or enhancer activity influenced by regions outside the CRE), since BOM is constrained to cis-regulatory information within the input sequence.
- The regulatory code depends on motif order, orientation, or spacing (motif grammar), which the unordered bag-of-motifs representation ignores.
- Sequences of interest contain TF binding patterns not captured by current motif annotations in the provided MEME file, since reliance on known TF motifs neglects informative sequences.

## Inputs

- Four-column text BED file with columns (chromosome, start, stop, condition) specifying distal CREs per cell type
- Reference genome package BSgenome.Mmusculus.UCSC.mm10 (mouse mm10)
- Motif database gimme.vertebrate.v5.0.meme (shipped in BagOfMotifs extdata)
- Installed FIMO binary (MEME Suite)

## Outputs

- Per-sequence motif count feature table for distal CREs
- Trained binary and 17-class XGBoost models
- Held-out prediction metrics (auROC, auPR, precision, recall, F1) written to bom_model_metrics.csv
- SHAP-based motif importance scores and bar/beeswarm/waterfall plots

## How to apply

Generate FASTA sequences from four-column BED inputs (chromosome, start, stop, condition) against the mouse mm10 genome with BagOfMotifs::generateAllFasta(bedDir, genome = "Mmusculus"), then scan sequences for motifs from gimme.vertebrate.v5.0.meme using BagOfMotifs::runFIMO(motifs_path, FIMO_path), setting FIMO_updated = FALSE when the installed FIMO version is < 5.5.4. Count motif occurrences per distal CRE and train the binary and multiclass (17-class) XGBoost models with BagOfMotifs::binModel(), then predict held-out labels and compute performance with BagOfMotifs::predict_binary_multi(), writing the reproduced metrics to bom_model_metrics.csv. Motif importance can be interpreted downstream with save_shap_multi() and shapPlots_multi(). Accept the result if held-out binary performance reaches auROC ≈ 0.98, auPR ≈ 0.98 and ≈ 93% CRE assignment accuracy, and multiclass performance reaches precision ≈ 0.99, recall ≈ 0.88 and auPR ≈ 0.99.

## Related tools

- **Bag-of-Motifs (BOM)** (R package providing generateAllFasta, runFIMO, binModel and predict_binary_multi to build and evaluate the motif-count XGBoost classifiers) — https://github.com/ewonglab/BOM_package
- **FIMO** (Scans the CRE FASTA sequences against gimme.vertebrate.v5.0.meme to produce motif occurrences counted per sequence) — https://meme-suite.org/meme/doc/install.html
- **XGBoost** (Gradient-boosted tree algorithm underlying the binary and multiclass cell-type classification models)
- **GimmeMotifs** (Source of the gimme.vertebrate.v5.0.meme motif database used for scanning) — https://github.com/vanheeringen-lab/gimmemotifs

## Examples

```
generateAllFasta(bedDir = "path/to/input", genome = "Mmusculus"); BagOfMotifs::runFIMO(motifs_path = motifs_path, FIMO_path = FIMO_path); BagOfMotifs::binModel(); BagOfMotifs::predict_binary_multi()
```

## Evaluation signals

- Held-out binary classification on 2,489 sequences reaches auROC ≈ 0.98 and auPR ≈ 0.98 with ≈ 93% of CREs assigned to their correct cell type (average precision 0.93, recall 0.92, F1 0.92)
- Multiclass model reaches precision ≈ 0.99, recall ≈ 0.88 and auPR ≈ 0.99
- Reproduced metrics file bom_model_metrics.csv is produced and matches the expected ranges
- FIMO runs with FIMO_updated = FALSE when the installed FIMO version is < 5.5.4, avoiding parsing errors
- SHAP outputs (bar, beeswarm, waterfall plots) identify motif importance consistent with the trained models

## Limitations

- BOM is constrained to cis-regulatory information within the input sequence; open chromatin status, histone marks, and enhancer activity can be influenced by regions beyond the sequence length used
- Reliance on known TF motifs neglects informative sequences not captured by current motif annotations, and TFs do not bind all their motifs
- Unknown how much of the remaining genome regions may share similar motif composition yet not share regulatory activity
- Integration with motif grammar (motif order/orientation/spacing) is an important future direction not yet done

## Evidence

- [results] On 2,489 held-out sequences, BOM correctly assigned 93% of CREs to their cell type of origin, with average precision, recall and F1 of 0.93, 0.92 and 0.92 (auROC = 0.98; auPR = 0.98): [evidence span withheld — anchor status: not_found]
- [methods] workflow FIMO step: [evidence span withheld — anchor status: not_found]
- [readme] README quick start: "generateAllFasta(bedDir = "path/to/input", genome = "Mmusculus")
BagOfMotifs::runFIMO(motifs_path = motifs_path, FIMO_path = FIMO_path)"
- [readme] README package description: "BAG-of-Motifs (BOM) is an R package that uses XGBoost to construct predictive models of cis-regulatory sequences between cell states."
- [discussion] discussion limitation: "BOM is constrained to cis-regulatory information within the input sequence."
- [abstract] abstract claim: "This minimalist representation, combined with gradient-boosted trees, enables the accurate prediction of cell-type-specific enhancers across mouse, human, zebrafish, and Arabidopsis datasets."
