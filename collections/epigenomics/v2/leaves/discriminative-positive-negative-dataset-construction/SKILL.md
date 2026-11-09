---
name: discriminative-positive-negative-dataset-construction
description: 'Use when preparing inputs for discriminative motif discovery with DiNAMO:
  you have genomic coordinates or event positions (e.g., 3272 monocyte SSE coordinates
  on Hg18) and must build signal.fa and a matched control.fa pair.'
when_to_use_negative:
- The analysis goal is positional motif enrichment with no discriminative comparison
  — a single-sequence-set motif finder would suffice without building a control dataset
- Input is already a motif frequency matrix or curated motif set rather than raw sequence
  coordinates — dataset construction is unnecessary
- Events are not sequence-localized (no genomic coordinates or positions to extract
  sequences around)
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: DiNAMO
  role: Consumes the constructed signal.fa and control.fa pair for exact discriminative
    IUPAC motif discovery
  repo: https://github.com/bonsai-team/DiNAMO
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-018-2215-1
    title: 'DiNAMO: highly sensitive DNA motif discovery in high-throughput sequencing
      data'
  source_content_hash: sha256:31082103d178f4e60e2a19f82b0a56df52011d4dc35e96b219cf1ea46aac1087
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bonsai-team/DiNAMO
  tools:
  - DiNAMO
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# discriminative-positive-negative-dataset-construction

## Summary

Constructs a matched pair of FASTA datasets — a positive (signal) dataset of sequences centered on the events of interest (e.g., systematic sequencing error positions) and a negative (control) dataset from non-event regions — so DiNAMO can perform exact discriminative IUPAC motif discovery against a background.

## When to use

Use when preparing inputs for discriminative motif discovery with DiNAMO: you have genomic coordinates or event positions (e.g., 3272 monocyte SSE coordinates on Hg18) and must build signal.fa and a matched control.fa pair.

## When NOT to use

- The analysis goal is positional motif enrichment with no discriminative comparison — a single-sequence-set motif finder would suffice without building a control dataset
- Input is already a motif frequency matrix or curated motif set rather than raw sequence coordinates — dataset construction is unnecessary
- Events are not sequence-localized (no genomic coordinates or positions to extract sequences around)

## Inputs

- Genomic coordinates of events of interest (e.g., 3272 monocyte sequencing error coordinates on Hg18)
- Reference genome (Human genome Hg18) for sequence extraction
- Non-event genomic regions for the matched negative/control dataset

## Outputs

- signal.fa — positive dataset in FASTA format
- control.fa — matched negative dataset in FASTA format
- DiNAMO output of over-represented IUPAC motifs in MEME format

## How to apply

Extract sequences at the event positions from the reference genome — for the SSE application, take the 3272 monocyte genomic coordinates of sequencing errors on Human genome Hg18 to build the positive dataset. Assemble a matched negative dataset from non-event regions of the same genome so motif over-representation is measured against a realistic background. Save both sets as FASTA files (signal.fa and control.fa), since DiNAMO takes exactly two input FASTA files via -pf and -nf. Then run DiNAMO with the motif length '-l' (optionally '-d' for maximum degenerate letters, '-p' for fixed-position mode when errors occur at specific offsets, and '-t' for the Fisher's exact test P-value threshold, default 0.05); the rationale is that DiNAMO counts L-mers in both files, builds an IUPAC lattice, simplifies it by mutual information, and filters motifs with the Holm–Bonferroni-adjusted Fisher's exact test.

## Related tools

- **DiNAMO** (Consumes the constructed signal.fa and control.fa pair for exact discriminative IUPAC motif discovery) — https://github.com/bonsai-team/DiNAMO

## Examples

```
dinamo -pf signal.fa -nf control.fa -l 6
```

## Evaluation signals

- Both signal.fa and control.fa exist as valid FASTA files with the expected sequence counts (3272 error-position sequences in the SSE case)
- DiNAMO runs to completion on the dataset pair within a desktop profile (< 8 GB of RAM)
- Output motifs are delivered in valid MEME format
- Recovered over-represented IUPAC motifs match the expected biological signal (e.g., NNRGGT on the SSE dataset, adjusted P-value < 10−324)
- No positive-dataset coordinates leak into the negative (control) dataset

## Limitations

- Motif discovery quality depends on how well the control dataset matches the positive dataset's background composition; a poorly matched control can produce spurious motifs
- Fixed-position events require the '-p' offset parameter; using default scanning mode would dilute position-specific signals
- DiNAMO's Fisher's exact test threshold defaults to 0.05 and multiple-testing correction is Holm–Bonferroni; alternative thresholds need the '-t' parameter
- No software version for DiNAMO is stated in the article or README, which may affect reproducibility

## Evidence

- [readme] DiNAMO takes as input two fasta files, which contain respectively the positive dataset (signal.fa) and the negative dataset (control.fa).: "DiNAMO takes as input two fasta files, which contain respectively the positive dataset (signal.fa) and the negative dataset (control.fa)."
- [results] we used the monocyte dataset described in [43]. This compilation contains 3272 genomic coordinates of sequencing errors on the Human genome (Hg18): "we used the monocyte dataset described in [43]. This compilation contains 3272 genomic coordinates of sequencing errors on the Human genome (Hg18)"
- [methods] The first step of the algorithm consists in counting the number of occurrences of each existing L-mer in the two input files: "The first step of the algorithm consists in counting the number of occurrences of each existing L-mer in the two input files"
- [methods] It can be easily installed (binaries for different Linux, MacOS and Windows are available) and used on a desktop machine (< 8 GB of RAM, see Table 3).: "It can be easily installed (binaries for different Linux, MacOS and Windows are available) and used on a desktop machine (< 8 GB of RAM, see Table 3)."
