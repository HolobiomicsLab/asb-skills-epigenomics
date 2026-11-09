---
name: fimo-motif-scanning
description: Use when you have CRE coordinate BED files (chromosome, start, stop,
  condition) and need motif occurrence annotations from known position weight matrices
  to build motif-count features for a classifier.
when_to_use_negative:
- You need de novo motif discovery from ChIP-seq peaks rather than scanning known
  PWMs (use the GimmeMotifs prediction pipeline instead).
- Your input is already a motif-count or feature table — skip directly to BagOfMotifs::binModel()
  model training.
- The regulatory activity of interest depends on context beyond the input sequence
  length (e.g. distal enhancer activity influenced by regions outside the scanned
  CRE), which motif scanning within the given coordinates cannot capture.
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3517
tools:
- name: FIMO
  role: Scans the generated CRE FASTA sequences against the gimme.vertebrate.v5.0.meme
    PWM library to annotate motif occurrences; invoked through the BagOfMotifs::runFIMO
    wrapper
  repo: https://meme-suite.org/meme/doc/install.html
- name: BagOfMotifs (BOM)
  role: Provides generateAllFasta to build the FASTA input, the runFIMO wrapper, and
    binModel()/predict_binary_multi() to convert scan results into motif counts and
    train/evaluate XGBoost classifiers
  repo: https://github.com/ewonglab/BOM_package
- name: GimmeMotifs
  role: Source of the gimme.vertebrate.v5.0.meme motif library used as the FIMO scanning
    reference
  repo: https://github.com/vanheeringen-lab/gimmemotifs
- name: BSgenome.Mmusculus.UCSC.mm10
  role: Mouse mm10 genome package used by generateAllFasta to extract CRE sequences
    prior to FIMO scanning
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_001
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-65362-2
    title: Motif-based models accurately predict cell type-specific distal regulatory
      elements
  source_content_hash: sha256:e473918a957dfdcd138b3ac124b958fdb4c9cccfef751d3f968cb284b4268c79
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BSgenome.Mmusculus.UCSC.mm10
  - BagOfMotifs
  - BagOfMotifs (BOM)
  - FIMO
  - GimmeMotifs
  - binModel
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# fimo-motif-scanning

## Summary

Scan cis-regulatory element FASTA sequences for transcription factor motif occurrences with FIMO against the gimme.vertebrate.v5.0.meme PWM library, producing the per-sequence motif annotations that the BOM package converts into unordered motif-count features for XGBoost classification.

## When to use

Use when you have CRE coordinate BED files (chromosome, start, stop, condition) and need motif occurrence annotations from known position weight matrices to build motif-count features for a classifier.

## When NOT to use

- You need de novo motif discovery from ChIP-seq peaks rather than scanning known PWMs (use the GimmeMotifs prediction pipeline instead).
- Your input is already a motif-count or feature table — skip directly to BagOfMotifs::binModel() model training.
- The regulatory activity of interest depends on context beyond the input sequence length (e.g. distal enhancer activity influenced by regions outside the scanned CRE), which motif scanning within the given coordinates cannot capture.

## Inputs

- Four-column BED-style text file (chromosome, start, stop, condition) of CRE coordinates with condition labels
- FASTA sequences generated from the BED file against a BSgenome package (e.g. BSgenome.Mmusculus.UCSC.mm10)
- gimme.vertebrate.v5.0.meme motif library (shipped with BagOfMotifs extdata)
- Locally installed FIMO executable (path passed via FIMO_path)

## Outputs

- Per-sequence FIMO motif occurrence annotations (motif hits with coordinates and q-values) for each CRE
- Motif-count feature matrix per CRE used by BagOfMotifs::binModel() for XGBoost training
- Motif-coverage fractions at q-value thresholds 0.1, 0.3, 0.5 for annotation-density sensitivity analysis

## How to apply

First generate FASTA sequences from the four-column input BED files against the appropriate BSgenome (e.g. BSgenome.Mmusculus.UCSC.mm10) using BagOfMotifs::generateAllFasta(bedDir, genome = "Mmusculus"). Then run FIMO via BagOfMotifs::runFIMO(motifs_path = gimme.vertebrate.v5.0.meme, FIMO_path = "/path/to/fimo"), setting FIMO_updated = FALSE when the installed FIMO version is < 5.5.4 because the output parser differs. For sensitivity work, repeat scans at q-value cutoffs 0.1, 0.3, and 0.5 and compute the fraction of CRE nucleotides covered by at least one motif annotation (expected ~83% at q <= 0.5). The rationale is that motif counts, independent of order, orientation, or spacing, feed gradient-boosted trees (BagOfMotifs::binModel()) that capture combinatorial TF contributions; note that removing overlapping motif annotations reduced auROC by 0.32, so overlapping annotations carry biologically meaningful signal.

## Related tools

- **FIMO** (Scans the generated CRE FASTA sequences against the gimme.vertebrate.v5.0.meme PWM library to annotate motif occurrences; invoked through the BagOfMotifs::runFIMO wrapper) — https://meme-suite.org/meme/doc/install.html
- **BagOfMotifs (BOM)** (Provides generateAllFasta to build the FASTA input, the runFIMO wrapper, and binModel()/predict_binary_multi() to convert scan results into motif counts and train/evaluate XGBoost classifiers) — https://github.com/ewonglab/BOM_package
- **GimmeMotifs** (Source of the gimme.vertebrate.v5.0.meme motif library used as the FIMO scanning reference) — https://github.com/vanheeringen-lab/gimmemotifs
- **BSgenome.Mmusculus.UCSC.mm10** (Mouse mm10 genome package used by generateAllFasta to extract CRE sequences prior to FIMO scanning)

## Examples

```
BagOfMotifs::runFIMO(motifs_path = motifs_path, FIMO_path = FIMO_path)
```

## Evaluation signals

- At q-value cutoff 0.5, the fraction of CRE nucleotides covered by at least one motif annotation reaches approximately 83%.
- FIMO_updated is set to FALSE only when the installed FIMO version is < 5.5.4; otherwise the default parser is used.
- Downstream BOM models trained on the resulting motif counts reproduce benchmark metrics (e.g. mouse E8.25 held-out auROC ≈ 0.98, auPR ≈ 0.98, ~93% cell-type assignment accuracy).
- Retraining on non-overlapping motif annotations should reproduce the reported 0.32 auROC reduction, confirming the scan captured overlapping annotations faithfully.
- All scans use the gimme.vertebrate.v5.0.meme library from the BagOfMotifs extdata path, not an ad-hoc PWM set.

## Limitations

- Reliance on known TF motifs neglects informative sequences not captured by current motif annotations.
- TFs do not bind all their motifs, so motif occurrence does not guarantee binding.
- It remains undetermined how much of the remaining genome shares similar motif composition yet does not share regulatory activity; BOM is constrained to cis-regulatory information within the input sequence.

## Evidence

- [intro] Many studies use TF motif enrichment6–10, using tools like FIMO and HOMER, to scan for over-represented motifs using position weight matrices.: "Many studies use TF motif enrichment6–10, using tools like FIMO and HOMER, to scan for over-represented motifs using position weight matrices."
- [methods] runFIMO call with motifs_path and FIMO_path: "BagOfMotifs::runFIMO(
	motifs_path = motifs_path,
	FIMO_path   = FIMO_path
)"
- [readme] FIMO requirement in BOM README: "FIMO is required:
Installation information can be found <a href="https://meme-suite.org/meme/doc/install.html"> here </a>"
- [readme] motif library path from README: "motifs_path <- paste0(extdata_path,"/gimme.vertebrate.v5.0.meme")"
