---
name: motif-scanning-of-regulatory-sequences
description: Use when you have genomic intervals (chromosome, start, stop, condition
  BED-style input) or extracted FASTA of candidate enhancers and need per-sequence
  TF motif occurrences as features for cell-type-specific CRE classification or cross-species
  transfer.
when_to_use_negative:
- When you need motif order, orientation, or spacing information — BOM's representation
  is explicitly unordered motif counts, so grammar-aware questions are out of scope
  (integration with motif grammar remains a stated future direction)
- When regulatory logic depends on sequences beyond the scanned interval — BOM is
  constrained to cis-regulatory information within the input sequence length
- When the TF motifs of interest are absent from current motif annotations — reliance
  on known TF motifs neglects informative sequences not captured by PWM libraries
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3173
tools:
- name: FIMO
  role: Scans extracted CRE sequences for TF motif occurrences using the gimme.vertebrate.v5.0.meme
    PWM library; invoked via BagOfMotifs::runFIMO()
- name: Bag-of-Motifs (BOM)
  role: 'R package wrapping the workflow: generateAllFasta() extracts sequences from
    genomic intervals, runFIMO() performs the motif scan, binModel()/predict_binary_multi()
    consume the resulting motif counts'
  repo: https://github.com/ewonglab/BOM_package
- name: GimmeMotifs
  role: Source of the gimme.vertebrate.v5.0.meme vertebrate motif library used as
    the scanning reference
  repo: https://github.com/vanheeringen-lab/gimmemotifs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-65362-2
    title: Motif-based models accurately predict cell type-specific distal regulatory
      elements
  source_content_hash: sha256:5bc99c6ad9f3f4ec9624ba7637358c13597eb8ca3abe15351495444694dce83e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ewonglab/BOM_package
  tools:
  - Bag-of-Motifs (BOM)
  - FIMO
  - GimmeMotifs
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# motif-scanning-of-regulatory-sequences

## Summary

Scan distal cis-regulatory element sequences for TF motif occurrences with FIMO using the gimme.vertebrate.v5.0.meme PWM library, producing per-sequence motif counts that feed BOM's gradient-boosted classifiers and enable cross-species CRE prediction independent of sequence alignment.

## When to use

Use when you have genomic intervals (chromosome, start, stop, condition BED-style input) or extracted FASTA of candidate enhancers and need per-sequence TF motif occurrences as features for cell-type-specific CRE classification or cross-species transfer.

## When NOT to use

- When you need motif order, orientation, or spacing information — BOM's representation is explicitly unordered motif counts, so grammar-aware questions are out of scope (integration with motif grammar remains a stated future direction)
- When regulatory logic depends on sequences beyond the scanned interval — BOM is constrained to cis-regulatory information within the input sequence length
- When the TF motifs of interest are absent from current motif annotations — reliance on known TF motifs neglects informative sequences not captured by PWM libraries

## Inputs

- BED-style text file with four columns: chromosome, start, stop, condition
- Genome sequence (e.g. BSgenome.Mmusculus.UCSC.mm10 or genome of choice) for FASTA extraction
- TF motif PWM library: gimme.vertebrate.v5.0.meme (bundled with BagOfMotifs package)
- Installed FIMO binary (MEME Suite) with its filesystem path

## Outputs

- Per-sequence TF motif scan results (FIMO hits) used to build motif-count feature matrices
- FASTA files of CRE sequences per condition from generateAllFasta
- Motif-count feature table consumed by binModel()/predict_binary_multi() for classification and SHAP interpretation

## How to apply

Format the input as a four-column text file (chromosome, start, stop, condition) and extract sequences with BagOfMotifs::generateAllFasta(), specifying the genome of choice (e.g. 'Mmusculus' with BSgenome.Mmusculus.UCSC.mm10). Scan sequences by calling BagOfMotifs::runFIMO() with motifs_path pointing to the package's bundled gimme.vertebrate.v5.0.meme library and FIMO_path set to the local FIMO installation (install from meme-suite.org); set FIMO_updated = FALSE if your FIMO version is below 5.5.4. The resulting motif hits are counted per sequence — unordered, independent of motif order, orientation, or spacing — and these counts become the features for BagOfMotifs::binModel() XGBoost training or predict_binary_multi() classification. The motif-count representation is what allows models trained on mouse embryonic cardiomyocyte/erythroid CREs to classify homologous human fetal CREs (64.5% and 62.9% correct) even when most correctly predicted elements (90.1% and 96.6%) lack detectable human–mouse alignment by UCSC liftOver at minMatch = 0.95.

## Related tools

- **FIMO** (Scans extracted CRE sequences for TF motif occurrences using the gimme.vertebrate.v5.0.meme PWM library; invoked via BagOfMotifs::runFIMO())
- **Bag-of-Motifs (BOM)** (R package wrapping the workflow: generateAllFasta() extracts sequences from genomic intervals, runFIMO() performs the motif scan, binModel()/predict_binary_multi() consume the resulting motif counts) — https://github.com/ewonglab/BOM_package
- **GimmeMotifs** (Source of the gimme.vertebrate.v5.0.meme vertebrate motif library used as the scanning reference) — https://github.com/vanheeringen-lab/gimmemotifs

## Examples

```
BagOfMotifs::runFIMO(motifs_path = paste0(system.file("extdata",package="BagOfMotifs"),"/gimme.vertebrate.v5.0.meme"), FIMO_path = "/path/to/fimo")
```

## Evaluation signals

- FASTA extraction succeeds only if the input is a four-column (chromosome, start, stop, condition) file matching the genome build — sequence lengths should equal interval stop−start
- FIMO runs must use gimme.vertebrate.v5.0.meme; verify FIMO version compatibility (set FIMO_updated = FALSE for FIMO < 5.5.4) or runFIMO will error
- Downstream binModel() should produce a motif-count matrix with one row per input interval and one column per PWM in the library
- Cross-species sanity check: mouse-trained models should correctly classify ~64.5% of human cardiomyocyte and ~62.9% of human erythroblast CREs; deviations suggest genome-build or scan-parameter mismatches
- Alignment-independence check: ≥90% of correctly predicted cross-species elements should lack liftOver (minMatch = 0.95) alignment, confirming prediction is motif-based rather than sequence-homology-based

## Limitations

- TFs do not bind all their motifs, so motif occurrence does not guarantee binding
- The reliance on known TF motifs neglects informative sequences not captured by current motif annotations
- BOM is constrained to cis-regulatory information within the input sequence; open chromatin status, histone marks, and enhancer activity can be influenced by regions beyond the sequence length used
- Unknown how much of the remaining genome regions may share similar motif composition yet not share regulatory activity

## Evidence

- [results] Mouse-trained BOM models correctly identified 64.5% of human cardiomyocyte and 62.9% of human erythroblast CREs: [evidence span withheld — anchor status: not_found]
- [abstract] BOM motif-count representation: "We present Bag-of-Motifs (BOM), a computational framework that represents distal cis-regulatory elements as unordered counts of transcription factor (TF) motifs."
- [readme] runFIMO invocation: "BagOfMotifs::runFIMO(
	motifs_path = motifs_path,
	FIMO_path   = FIMO_path
)"
- [readme] Input format and genome: "Input is a text file with four columns (chromosome, start, stop, condition)"
- [full_text] FIMO version compatibility: [evidence span withheld — anchor status: not_found]
- [methods] Motif library: "motifs_path <- paste0(extdata_path,"/gimme.vertebrate.v5.0.meme")"
- [results] Cross-species workflow: [evidence span withheld — anchor status: not_found]
- [discussion] Sequence constraint limitation: "BOM is constrained to cis-regulatory information within the input sequence. The status of open chromatin, histone marks, and enhancer activity can be influenced by regions beyond the sequence length"
- [discussion] Motif annotation limitation: "the reliance on known TF motifs neglects informative sequences not captured by current motif annotations. Furthermore, TFs do not bind all their motifs"
