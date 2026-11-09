---
name: nucleotide-annotation-fraction-quantification
description: Use when reproducing or sanity-checking the motif-scanning sensitivity
  of a BOM/FIMO CRE classification workflow, e.g. verifying that ~83% of CRE nucleotides
  are annotated at q<=0.5 before ablation or retraining.
when_to_use_negative:
- Input is already a per-sequence motif-count feature table or trained model — this
  skill only quantifies raw annotation coverage from coordinates and a motif library.
- The analysis requires motif order, orientation, or spacing information — BOM's representation
  is explicitly unordered counts of motifs, independent of grammar.
- The genome or motif library differs from mm10/gimme.vertebrate.v5.0.meme without
  revalidating the coverage expectation — the ~83% figure is dataset- and library-specific.
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
tools:
- name: FIMO
  role: Scans CRE sequences for TF motif occurrences at controlled q-value thresholds
    to produce the annotations whose nucleotide coverage is quantified
- name: BagOfMotifs (BOM)
  role: R package whose runFIMO wrapper invokes FIMO and whose extdata provides gimme.vertebrate.v5.0.meme
    and mm10-based CRE workflow used for the coverage check
  repo: github.com/ewonglab/BOM_package
- name: BSgenome.Mmusculus.UCSC.mm10
  role: Genome assembly used to extract CRE sequences from the four-column coordinate
    files prior to scanning
- name: GimmeMotifs
  role: Source of the gimme.vertebrate.v5.0.meme motif library used as the scanning
    reference
  repo: vanheeringen-lab/gimmemotifs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-65362-2
    title: Motif-based models accurately predict cell type-specific distal regulatory
      elements
  source_content_hash: sha256:b1192f837f771d66d95ff66f38e95a243d9c04341b1afcdac42ed70d3299f750
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
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# nucleotide-annotation-fraction-quantification

## Summary

Quantifies the fraction of CRE nucleotides covered by at least one FIMO motif annotation at escalating q-value cutoffs (0.1, 0.3, 0.5), using the gimme.vertebrate.v5.0.meme library and BOM pipeline. It serves as a sensitivity check that motif-scan sensitivity is sufficient before motif-count features are used for classification.

## When to use

Use when reproducing or sanity-checking the motif-scanning sensitivity of a BOM/FIMO CRE classification workflow, e.g. verifying that ~83% of CRE nucleotides are annotated at q<=0.5 before ablation or retraining.

## When NOT to use

- Input is already a per-sequence motif-count feature table or trained model — this skill only quantifies raw annotation coverage from coordinates and a motif library.
- The analysis requires motif order, orientation, or spacing information — BOM's representation is explicitly unordered counts of motifs, independent of grammar.
- The genome or motif library differs from mm10/gimme.vertebrate.v5.0.meme without revalidating the coverage expectation — the ~83% figure is dataset- and library-specific.

## Inputs

- CRE coordinate file (four-column text: chromosome, start, stop, condition) for mouse E8.25 or human cell-line datasets
- gimme.vertebrate.v5.0.meme motif library (shipped with BagOfMotifs extdata)
- Mouse mm10 genome (BSgenome.Mmusculus.UCSC.mm10)
- Locally installed FIMO (MEME Suite) binary path

## Outputs

- Per-threshold motif-annotation coverage fractions (fraction of CRE nucleotides covered by >=1 motif at q=0.1, 0.3, 0.5)
- FIMO motif annotation tracks over CRE sequences at each q-value cutoff
- Validated input for downstream per-sequence motif-count feature construction (binModel) and auROC sensitivity comparison

## How to apply

Load the CRE coordinate files (four-column text: chromosome, start, stop, condition) for the mouse E8.25 or human cell-line datasets and generate sequences with generateAllFasta against BSgenome.Mmusculus.UCSC.mm10. Run FIMO via BagOfMotifs::runFIMO with motifs_path pointing to gimme.vertebrate.v5.0.meme and FIMO_path to the local FIMO installation. Repeat the scan at q-value cutoffs 0.1, 0.3, and 0.5, and at each threshold compute the fraction of CRE nucleotides overlapped by at least one motif annotation. The key decision point is the q<=0.5 threshold, where the annotation fraction should reach approximately 83%; lower cutoffs trade coverage for stricter motif confidence. Only after confirming coverage should the annotations be converted to per-sequence motif counts for downstream ablation of overlapping motif annotations.

## Related tools

- **FIMO** (Scans CRE sequences for TF motif occurrences at controlled q-value thresholds to produce the annotations whose nucleotide coverage is quantified)
- **BagOfMotifs (BOM)** (R package whose runFIMO wrapper invokes FIMO and whose extdata provides gimme.vertebrate.v5.0.meme and mm10-based CRE workflow used for the coverage check) — github.com/ewonglab/BOM_package
- **BSgenome.Mmusculus.UCSC.mm10** (Genome assembly used to extract CRE sequences from the four-column coordinate files prior to scanning)
- **GimmeMotifs** (Source of the gimme.vertebrate.v5.0.meme motif library used as the scanning reference) — vanheeringen-lab/gimmemotifs

## Examples

```
generateAllFasta(bedDir = "path/to/input", genome = "Mmusculus"); BagOfMotifs::runFIMO(motifs_path = motifs_path, FIMO_path = FIMO_path)
```

## Evaluation signals

- At q<=0.5, the fraction of CRE nucleotides covered by at least one motif annotation reaches approximately 83%.
- Coverage fractions are monotonically non-decreasing as the q-value cutoff relaxes from 0.1 to 0.3 to 0.5.
- FIMO runs complete for all three thresholds without missing-value errors across every sequence in the CRE coordinate file.
- Resulting per-threshold annotation tables can be converted to per-sequence motif counts and fed to binModel without schema failures.
- Downstream auROC values computed with predict_binary_multi match the reported baseline before any overlapping-motif ablation is applied.

## Limitations

- The reliance on known TF motifs neglects informative sequences not captured by current motif annotations.
- TFs do not bind all their motifs, so annotation coverage does not imply actual TF occupancy.
- BOM is constrained to cis-regulatory information within the input sequence; open chromatin status, histone marks, and enhancer activity can be influenced by regions beyond the sequence length used.
- Seemingly redundant (overlapping) motif annotations removed in ablation may carry biologically meaningful information (e.g., composite binding sites), so aggressive de-duplication reduces auROC by 0.32.

## Evidence

- [results] Repeat the FIMO scans at q-value cutoffs 0.1, 0.3, and 0.5, and at each threshold compute the fraction of CRE nucleotides covered by at least one motif annotation: [evidence span withheld — anchor status: not_found]
- [readme] runFIMO invocation: "BagOfMotifs::runFIMO(
	motifs_path = motifs_path,
	FIMO_path   = FIMO_path
)"
- [methods] motif library path: "motifs_path <- paste0(extdata_path,"/gimme.vertebrate.v5.0.meme")"
- [abstract] BOM representation rationale: "This minimalist representation, combined with gradient-boosted trees, enables the accurate prediction of cell-type-specific enhancers across mouse, human, zebrafish, and Arabidopsis datasets."
