---
name: rna-secondary-structure-masking
description: Use when predicting RNA:dsDNA triplex interactions with 3plex/Triplexator-style
  scanning and you want to exclude ssRNA nucleotides unlikely to be available for
  Hoogsteen pairing, or when benchmarking whether structure-aware masking improves
  TPX prediction AUC for lncRNAs with validated binding.
when_to_use_negative:
- The analysis goal does not require RNA accessibility filtering — ss0 (cutoff 0)
  leaves the sequence unmasked and is the correct reference when structure masking
  is not being evaluated
- Predicting protein-mediated RNA:DNA interactions, where intramolecular RNA pairing
  does not preclude binding
- The lncRNA lacks reliable secondary-structure input (e.g. no RNAplfold probabilities
  computed), making cut-off-based masking meaningless
edam_operation: http://edamontology.org/operation_0278
edam_topics:
- http://edamontology.org/topic_0097
- http://edamontology.org/topic_0099
tools:
- name: RNAplfold
  role: Computes per-nucleotide RNA single-strandedness/unpaired probabilities used
    to mask paired nucleotides before triplex scanning
- name: ViennaRNA
  role: Package providing RNAplfold for RNA secondary structure prediction
- name: 3plex
  role: Applies the single_strandedness_cutoff (-s) to exclude masked ssRNA nucleotides
    from the PATO triplex scan
  repo: https://github.com/molinerisLab/3plex
- name: pROC
  role: Computes ROC curves and AUC values per lncRNA x masking-level combination
    to evaluate whether masking improves prediction
- name: bedtools
  role: Generates length-matched shuffled genomic negatives for the AUC comparison
    across masking levels
provenance:
  source_task_ids:
  - task_002
  - task_006
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2023.05.016
    title: 3plex enables deep computational investigation of triplex forming lncRNAs
  source_content_hash: sha256:eb04e8db9baa970a65e8d424abe041f5848349344ad558de90abdb30d8ebfa86
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/molinerisLab/3plex
  repo_url: https://github.com/molinerisLab/3plex
  tools:
  - 3plex
  - LongTarget
  - RNAplfold
  - Triplexator
  - ViennaRNA
  - bedtools
  - pROC
  edam_operation: http://edamontology.org/operation_0278
  edam_topics:
  - http://edamontology.org/topic_0097
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# rna-secondary-structure-masking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Masks paired (low-accessibility) RNA nucleotides prior to triplex (TPX) scanning by deriving per-nucleotide single-strandedness probabilities with RNAplfold (ViennaRNA, -u option, window length 8) and applying cut-offs ss10/ss20/ss50, so that ssRNA regions likely involved in intramolecular base pairing are excluded from triplex-forming oligo (TFO) search.

## When to use

Use when predicting RNA:dsDNA triplex interactions with 3plex/Triplexator-style scanning and you want to exclude ssRNA nucleotides unlikely to be available for Hoogsteen pairing, or when benchmarking whether structure-aware masking improves TPX prediction AUC for lncRNAs with validated binding peaks.

## When NOT to use

- The analysis goal does not require RNA accessibility filtering — ss0 (cutoff 0) leaves the sequence unmasked and is the correct reference when structure masking is not being evaluated
- Predicting protein-mediated RNA:DNA interactions, where intramolecular RNA pairing does not preclude binding
- The lncRNA lacks reliable secondary-structure input (e.g. no RNAplfold probabilities computed), making cut-off-based masking meaningless

## Inputs

- lncRNA transcript sequence in FASTA format (e.g. longest transcript from GENCODE v32)
- RNAplfold per-nucleotide unpaired probabilities (-u 8, window length 8)
- Positive RNA:DNA interaction peak sets (e.g. Top1000, IDR conservative) and matched genomic negatives
- Masking cut-off specification (ss0, ss10, ss20, ss50)

## Outputs

- Masked RNA sequence variants (ss10, ss20, ss50) with low-accessibility nucleotides masked
- Per-nucleotide single-strandedness/unpaired probability track from RNAplfold
- TPX predictions per masking level scored with thermal-stability-based scoring
- AUC comparison table (e.g. ss_masking_auc_comparison.csv, 7 lncRNAs x 4 masking levels) with AUC values within [0, 1]

## How to apply

Compute per-nucleotide RNA single-strandedness with RNAplfold from the ViennaRNA package using the -u option with a window of length 8 centred on each position (3plex defaults: --RNAplfold_unpaired_window 8, window_size 200, span 150). Generate four sequence variants: ss0 (unmasked reference) and masked versions in which nucleotides with unpaired probability below the cut-off are masked, such that the cut-offs remove 10%, 20% or 50% of the sequence (ss10, ss20, ss50); in 3plex this is controlled with -s/--single_strandedness_cutoff. Masking is applied BEFORE the PATO-based triplex scan, since a triplex interaction requires a stretch of ssRNA not involved in any other hydrogen bond. Run the scan under each masking level on the same positive peaks (e.g. Top1000 RNA:DNA interaction peaks) and bedtools-shuffled length-matched negatives, then compute ROC curves and AUC per lncRNA x masking-level combination (e.g. with pROC v1.17.0.1). Compare masked AUCs against ss0 to decide per lncRNA whether secondary-structure filtering helps — the paper found it improves prediction only for a subset of lncRNAs (MEG3 in human; Meg3 and lncSmad7 in mouse, 3 of 7 TPX-validated lncRNAs).

## Related tools

- **RNAplfold** (Computes per-nucleotide RNA single-strandedness/unpaired probabilities used to mask paired nucleotides before triplex scanning)
- **ViennaRNA** (Package providing RNAplfold for RNA secondary structure prediction)
- **3plex** (Applies the single_strandedness_cutoff (-s) to exclude masked ssRNA nucleotides from the PATO triplex scan) — https://github.com/molinerisLab/3plex
- **pROC** (Computes ROC curves and AUC values per lncRNA x masking-level combination to evaluate whether masking improves prediction)
- **bedtools** (Generates length-matched shuffled genomic negatives for the AUC comparison across masking levels)

## Examples

```
docker run -u `id -u`:`id -g` -it --rm -v $PWD:$PWD imolineris/3plex:v1.0 $PWD/ssRNA.fa $PWD/dsDNA.fa $PWD/results_3plex -s 20
```

## Evaluation signals

- The comparison table contains one row per lncRNA x masking level (7 x 4) with AUC values within [0, 1]
- ss0 (cut-off 0) reproduces the unmasked reference; masked variants remove exactly 10%, 20% or 50% of the sequence
- Masking-induced AUC improvement should appear only for a subset of lncRNAs (MEG3 in human, Meg3 and lncSmad7 in mouse), not uniformly
- 3plex is run with --RNAplfold_unpaired_window 8 (default) so masking probabilities correspond to the paper's window-of-8 single-strandedness
- Ranking of masking levels vs ss0 should be internally consistent when re-run on the same positive peaks and shuffled negatives

## Limitations

- Secondary-structure masking improved prediction only for a subset of lncRNAs: 3 out of 7 TPX-validated lncRNAs (MEG3, Meg3, lncSmad7); for the others it did not increase AUC
- The selected cut-offs are coarse and defined as fractions of the sequence masked (10%, 20%, 50%), not absolute probability thresholds tuned per transcript
- Masking is applied before scanning, so nucleotides masked by RNAplfold cannot participate in any predicted triplex, which may remove true positive TFOs in structured regions

## Evidence

- [results] 3 out of 7 TPX-validated lncRNAs (MEG3 in human and Meg3 and lncSmad7 in mouse) showed an improvement in AUC: [evidence span withheld — anchor status: not_found]
