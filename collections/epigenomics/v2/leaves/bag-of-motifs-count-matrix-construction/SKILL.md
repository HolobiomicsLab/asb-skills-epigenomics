---
name: bag-of-motifs-count-matrix-construction
description: Use when the input is a four-column BED file (chromosome, start, stop,
  condition) of cell-type-specific distal CREs from snATAC-seq (e.g., mouse E8.25,
  mm10) and a per-sequence motif-count feature matrix is needed for XGBoost classification.
when_to_use_negative:
- The input is already a numeric feature/count table (e.g., a peak-by-peak chromVAR
  deviation matrix) — motif scanning would be redundant
- The regulatory activity of interest is known to depend on sequences outside the
  provided CRE intervals — BOM is constrained to cis-regulatory information within
  the input sequence
- The task requires modelling motif grammar (order, orientation, spacing) or novel
  motifs absent from current TF motif annotations
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
tools:
- name: Bag-of-Motifs (BOM)
  role: R package providing generateAllFasta() and runFIMO() to build the per-CRE
    motif count matrix from BED input
  repo: github.com/ewonglab/BOM_package
- name: FIMO
  role: Scans CRE FASTA sequences against gimme.vertebrate.v5.0.meme position weight
    matrices to produce motif occurrence calls that are counted per region
- name: GimmeMotifs
  role: Source of the gimme.vertebrate.v5.0.meme motif database used as the motif
    scan reference
  repo: https://github.com/vanheeringen-lab/gimmemotifs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-65362-2
    title: Motif-based models accurately predict cell type-specific distal regulatory
      elements
  source_content_hash: sha256:fee05794381ad069670059798b76fcfaa637b04c95919a889768b42eab1e8689
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bag-of-Motifs (BOM)
  - FIMO
  - GimmeMotifs
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# bag-of-motifs-count-matrix-construction

## Summary

Constructs the BOM feature representation: each distal cis-regulatory element is encoded as an unordered vector of TF motif occurrence counts, obtained by scanning CRE sequences against the gimme.vertebrate.v5.0.meme motif database with FIMO, independent of motif order, orientation, or spacing.

## When to use

Use when the input is a four-column BED file (chromosome, start, stop, condition) of cell-type-specific distal CREs from snATAC-seq (e.g., mouse E8.25, mm10) and a per-sequence motif-count feature matrix is needed for XGBoost classification.

## When NOT to use

- The input is already a numeric feature/count table (e.g., a peak-by-peak chromVAR deviation matrix) — motif scanning would be redundant
- The regulatory activity of interest is known to depend on sequences outside the provided CRE intervals — BOM is constrained to cis-regulatory information within the input sequence
- The task requires modelling motif grammar (order, orientation, spacing) or novel motifs absent from current TF motif annotations

## Inputs

- Four-column BED text file (chromosome, start, stop, condition) of distal cis-regulatory elements
- Mouse mm10 genome (BSgenome.Mmusculus.UCSC.mm10)
- Motif database: gimme.vertebrate.v5.0.meme (shipped in BOM extdata)
- Path to a FIMO installation (MEME Suite)

## Outputs

- Per-CRE FASTA sequences extracted from mm10
- FIMO motif occurrence annotations per CRE sequence
- Motif occurrence count matrix (CREs × motifs) used for XGBoost training

## How to apply

First convert the four-column BED input (chromosome, start, stop, condition) into FASTA sequences against the mouse mm10 genome using BagOfMotifs::generateAllFasta(bedDir, genome = 'Mmusculus'), which requires BSgenome.Mmusculus.UCSC.mm10. Next scan every CRE sequence for motifs from gimme.vertebrate.v5.0.meme using BagOfMotifs::runFIMO(motifs_path, FIMO_path); if the installed FIMO version is < 5.5.4, set FIMO_updated = FALSE because output column layouts differ. Then count motif occurrences per distal CRE to build the count matrix, deliberately ignoring motif order, orientation, and spacing — this minimalist representation is what enables BagOfMotifs::binModel() to train gradient-boosted (XGBoost) binary and 17-class models. Validate the intermediate representation by confirming the resulting model reaches auROC ≈ 0.98, auPR ≈ 0.98 and ≈ 93% correct cell-type assignment on 2,489 held-out sequences.

## Related tools

- **Bag-of-Motifs (BOM)** (R package providing generateAllFasta() and runFIMO() to build the per-CRE motif count matrix from BED input) — github.com/ewonglab/BOM_package
- **FIMO** (Scans CRE FASTA sequences against gimme.vertebrate.v5.0.meme position weight matrices to produce motif occurrence calls that are counted per region)
- **GimmeMotifs** (Source of the gimme.vertebrate.v5.0.meme motif database used as the motif scan reference) — https://github.com/vanheeringen-lab/gimmemotifs

## Examples

```
library(BagOfMotifs); generateAllFasta(bedDir = "path/to/input", genome = "Mmusculus"); BagOfMotifs::runFIMO(motifs_path = motifs_path, FIMO_path = FIMO_path)
```

## Evaluation signals

- Downstream binary model on 2,489 held-out CREs reaches auROC ≈ 0.98 and auPR ≈ 0.98
- Binary model assigns ≈ 93% of held-out CREs to their cell type of origin (avg precision 0.93, recall 0.92, F1 0.92)
- Multiclass (17 cell type) model from the same count matrix reaches precision ≈ 0.99, recall ≈ 0.88, auPR ≈ 0.99
- FIMO runs without error only when FIMO_updated matches the installed version (set FALSE for FIMO < 5.5.4) — mismatched setting is a common silent failure
- Reproduced metrics are written to bom_model_metrics.csv and match the published ranges

## Limitations

- BOM is constrained to cis-regulatory information within the input sequence; open chromatin status, histone marks, and enhancer activity can be influenced by regions beyond the sequence length used
- Reliance on known TF motifs neglects informative sequences not captured by current motif annotations
- TFs do not bind all their motifs, and it is unknown how much of the remaining genome shares similar motif composition yet lacks regulatory activity
- Integration with motif grammar (order, orientation, spacing) remains an unimplemented future direction

## Evidence

- [abstract] We present Bag-of-Motifs (BOM), a computational framework that represents distal cis-regulatory elements as unordered counts of transcription factor (TF) motifs.: "We present Bag-of-Motifs (BOM), a computational framework that represents distal cis-regulatory elements as unordered counts of transcription factor (TF) motifs."
- [methods] generateAllFasta(bedDir = "path/to/input", genome = "Mmusculus"): "generateAllFasta(bedDir = "path/to/input", genome = "Mmusculus")"
- [methods] BagOfMotifs::runFIMO(motifs_path = motifs_path, FIMO_path = FIMO_path): "BagOfMotifs::runFIMO(motifs_path = motifs_path, FIMO_path = FIMO_path)"
- [readme] The tutorial requires the mouse mm10 genome. If you don't have this already:: "The tutorial requires the mouse mm10 genome. If you don't have this already:"
- [readme] Input is a text file with four columns (chromosome, start, stop, condition): "Input is a text file with four columns (chromosome, start, stop, condition)"
- [discussion] BOM is constrained to cis-regulatory information within the input sequence.: "BOM is constrained to cis-regulatory information within the input sequence."
