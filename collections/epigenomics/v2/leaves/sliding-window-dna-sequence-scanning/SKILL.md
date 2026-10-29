---
name: sliding-window-dna-sequence-scanning
description: Use when you have multi-FASTA ChIP-seq peak sequences (positive) plus
  a matched negative control set and need to identify over-represented degenerate
  IUPAC motifs of a chosen length L anywhere within the sequences.
when_to_use_negative:
- Motifs occur only at a known fixed offset relative to the sequence end — use DiNAMO's
  fixed-position mode (-p) instead of scanning mode, since scanning parses all windows
  unnecessarily
- You need gapped or position-specific motifs beyond IUPAC degenerate encoding (e.g.
  EM-based PWM discovery with MEME), which this exact IUPAC lattice approach does
  not produce
- 'You lack a negative control dataset, since the method is discriminative: it detects
  over-representation by comparing signal against control counts in the Fisher''s
  exact test'
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: DiNAMO
  role: Primary tool; run in scanning (sliding-window) mode to count L-mers, build
    the IUPAC lattice, and report over-represented motifs in MEME format
  repo: https://github.com/bonsai-team/DiNAMO
- name: JASPAR
  role: Reference database of transcription factor frequency matrices used to validate
    the recovered top motif
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-018-2215-1
    title: 'DiNAMO: highly sensitive DNA motif discovery in high-throughput sequencing
      data'
  source_content_hash: sha256:c9d8a9a4baa4d10c8b947e7c0cfaf0edaecbe4cced01e3b1ece09cf6c2f380d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bonsai-team/DiNAMO
  tools:
  - DiNAMO
  - JASPAR
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# sliding-window-dna-sequence-scanning

## Summary

Applies DiNAMO in scanning (sliding-window) mode to exhaustively discover over-represented IUPAC motifs in ChIP-seq peak sequences, comparing positive (signal.fa) against negative (control.fa) datasets to recover transcription factor binding site motifs and secondary co-factor motifs.

## When to use

Use when you have multi-FASTA ChIP-seq peak sequences (positive) plus a matched negative control set and need to identify over-represented degenerate IUPAC motifs of a chosen length L anywhere within the sequences.

## When NOT to use

- Motifs occur only at a known fixed offset relative to the sequence end — use DiNAMO's fixed-position mode (-p) instead of scanning mode, since scanning parses all windows unnecessarily
- You need gapped or position-specific motifs beyond IUPAC degenerate encoding (e.g. EM-based PWM discovery with MEME), which this exact IUPAC lattice approach does not produce
- You lack a negative control dataset, since the method is discriminative: it detects over-representation by comparing signal against control counts in the Fisher's exact test

## Inputs

- Positive dataset in multi-FASTA format (e.g. signal.fa containing ChIP-seq peak sequences such as GATA1 14,351 peaks, SOX2 4526, OCT4 3761, STAT3 2546, or KLF1 945)
- Negative control dataset in multi-FASTA format (control.fa)
- Motif length parameter L (e.g. 6 or 7)
- Optional: maximum number of degenerate letters (-d, e.g. 3) and Fisher's exact test P-value threshold (-t, default 0.05)

## Outputs

- dinamo_motifs.txt (or -o file) containing over-represented IUPAC motifs in MEME format
- Motifs with Holm–Bonferroni-adjusted Fisher's exact test P-value below 0.05, including secondary/co-factor motifs

## How to apply

Run DiNAMO in its default scanning mode with the command pattern 'dinamo -pf signal.fa -nf control.fa -l L', where all L-mer windows (e.g. L=7 in the ChIP-seq case study, L=6 in the documented example) are parsed one base at a time; optionally cap degenerate letters with '-d' (e.g. up to 3 degenerate letters). Internally, DiNAMO counts all L-mer occurrences in both files, stores them in a hashtable, and builds the lattice of IUPAC motifs whose instances are all present in the positive dataset. It simplifies this lattice using mutual information, then filters motifs with Fisher's exact test using Holm–Bonferroni adjustment at a default P-value threshold of 0.05 ('-t' overrides the threshold), with optional clustering of overlapping motifs sharing at least half the motif size. Secondary motifs are found by masking instances of primary motifs directly in the lattice. Output is written in MEME format for comparison against JASPAR frequency matrices.

## Related tools

- **DiNAMO** (Primary tool; run in scanning (sliding-window) mode to count L-mers, build the IUPAC lattice, and report over-represented motifs in MEME format) — https://github.com/bonsai-team/DiNAMO
- **JASPAR** (Reference database of transcription factor frequency matrices used to validate the recovered top motif)

## Examples

```
dinamo -pf signal.fa -nf control.fa -l 6
```

## Evaluation signals

- Output is a well-formed MEME-format motif file (parseable per meme-suite.org/doc/meme-format.html) containing only motifs with Holm–Bonferroni-adjusted Fisher's exact test P-value below 0.05
- The top reported motif matches the expected transcription factor's frequency matrix in the JASPAR database (as done in the paper for GATA1, SOX2, OCT4, STAT3, KLF1)
- Every reported IUPAC motif of length L has all its A/C/G/T string instances present in the positive dataset (the lattice invariant: 'the lattice contains exactly the set of all IUPAC motifs for which all instances are present')
- On the paper's ChIP-seq datasets, scanning mode with L=7 and up to 3 degenerate letters recovers the expected TF motif plus multiple secondary co-factor motifs
- Runs to completion on a desktop machine with < 8 GB of RAM per the paper's resource report

## Limitations

- Requires a negative control dataset — the discriminative Fisher's exact test (with Holm–Bonferroni adjustment) cannot assess over-representation without it
- Degeneracy must be bounded (maximum number of degenerate letters, '-d'), constraining the motif space searched
- No software version is stated for DiNAMO in the context; execution time and memory figures (Table 3) and supplementary materials (Additional files 1–3) are not retrievable from the provided text

## Evidence

- [readme] DiNAMO scanning-mode workflow on ChIP-seq peaks: "The DiNAMO software implements an exhaustive algorithm to detect over-represented IUPAC motifs in a set of DNA sequences."
- [readme] Input specification for scanning mode: "DiNAMO takes as input two fasta files, which contain respectively the positive dataset (signal.fa) and the negative dataset (control.fa)."
- [methods] First algorithm step in scanning mode: "The first step of the algorithm consists in counting the number of occurrences of each existing L-mer in the two input files"
- [methods] IUPAC lattice construction invariant: "From the hashtable of L-mers, we generate all IUPAC motifs of length L for which all instances are present in the hashtable."
- [methods] Statistical filtering threshold and correction: "The Holm–Bonferroni method [25] is applied to adjust the P-values and counteract the problem of multiple comparisons."
- [results] Validation against known TF motifs: "comparing them against the frequency matrices of the JASPAR database [34]"
- [readme] Fixed-position alternative to scanning mode: "It has two modes: *scanning* mode (default), where all windows are parsed, or *fixed-position* mode (see optional parameter *-p )"
