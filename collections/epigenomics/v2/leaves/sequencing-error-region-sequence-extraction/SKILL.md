---
name: sequencing-error-region-sequence-extraction
description: Use when you have genomic coordinates of systematic sequencing errors
  (e.g. the 3272 monocyte SSE positions on Hg18) and need paired signal/control FASTA
  inputs for fixed-position IUPAC motif discovery.
when_to_use_negative:
- The goal is de novo scanning-mode motif discovery in ChIP-seq peaks rather than
  motifs at a specific position relative to sequencing errors (DiNAMO's default scanning
  mode, not fixed-position mode, is appropriate there)
- The error coordinates and genome build do not match (e.g. Hg18 coordinates lifted
  onto another assembly), which would corrupt the extracted sequences
- No control dataset can be assembled — DiNAMO is a discriminative method requiring
  both a positive and a negative FASTA input
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0092
tools:
- name: DiNAMO
  role: Discovers over-represented IUPAC motifs from the extracted signal.fa/control.fa
    pair in fixed-position mode, outputting motifs in MEME format
  repo: https://github.com/bonsai-team/DiNAMO
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-018-2215-1
    title: 'DiNAMO: highly sensitive DNA motif discovery in high-throughput sequencing
      data'
  source_content_hash: sha256:007b421957e598571faf8f212dd37a3256704db690528aab9bb83100692d54b7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bonsai-team/DiNAMO
  tools:
  - DiNAMO
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# sequencing-error-region-sequence-extraction

## Summary

Builds a positive FASTA dataset by extracting genomic sequences at systematic sequencing error (SSE) positions and a matched control FASTA from non-error regions, so DiNAMO can discover degenerate IUPAC motifs that induce systematic sequencing errors.

## When to use

Use when you have genomic coordinates of systematic sequencing errors (e.g. the 3272 monocyte SSE positions on Hg18) and need paired signal/control FASTA inputs for fixed-position IUPAC motif discovery.

## When NOT to use

- The goal is de novo scanning-mode motif discovery in ChIP-seq peaks rather than motifs at a specific position relative to sequencing errors (DiNAMO's default scanning mode, not fixed-position mode, is appropriate there)
- The error coordinates and genome build do not match (e.g. Hg18 coordinates lifted onto another assembly), which would corrupt the extracted sequences
- No control dataset can be assembled — DiNAMO is a discriminative method requiring both a positive and a negative FASTA input

## Inputs

- Genomic coordinates of systematic sequencing errors (e.g. 3272 monocyte SSE coordinates on Hg18)
- Reference genome build matching the error coordinates (Human genome Hg18)
- Non-error genomic regions for the matched negative/control dataset
- DiNAMO parameters: motif length L (-l), optional max degenerate letters (-d), optional Fisher's exact test P-value threshold (-t), optional fixed-position offset (-p)

## Outputs

- Positive dataset FASTA (signal.fa) containing sequences at the error positions
- Matched negative/control dataset FASTA (control.fa) from non-error regions
- Over-represented IUPAC motifs in MEME format (DiNAMO output), e.g. the NNRGGT motif with adjusted P-value < 10^-324

## How to apply

Take the list of genomic coordinates of sequencing errors — in the reference study, 3272 monocyte SSE positions mapped on the Human genome (Hg18) — and extract the sequence at each error position to assemble the positive dataset (signal.fa). Assemble a matched negative dataset (control.fa) from non-error regions of the same genome build, preserving comparable sequence composition. Run DiNAMO on this pair in fixed-position mode (-p) with motif length set by -l, optionally limiting degenerate letters with -d and the Fisher's exact test P-value threshold with -t. DiNAMO counts L-mers into a hashtable, builds the IUPAC motif lattice, simplifies it by mutual information, and filters motifs with the Holm-Bonferroni-adjusted Fisher's exact test; output motifs are delivered in MEME format for comparison against error-associated motifs reported in the literature.

## Related tools

- **DiNAMO** (Discovers over-represented IUPAC motifs from the extracted signal.fa/control.fa pair in fixed-position mode, outputting motifs in MEME format) — https://github.com/bonsai-team/DiNAMO

## Examples

```
dinamo -pf signal.fa -nf control.fa -l 6 -p <offset> -d <i> -t <f> -o sse_motifs.txt
```

## Evaluation signals

- DiNAMO completes on the SSE dataset within the desktop profile (< 8 GB of RAM as reported in Table 3)
- Recovered over-represented IUPAC motifs match the degenerate error-associated motifs reported in the article's SSE results
- The top motif reproduces/refines the published GGT motif — expected NNRGGT with Holm-Bonferroni-adjusted Fisher's exact test P-value < 10^-324
- Output file parses as valid MEME format motifs (http://meme-suite.org/doc/meme-format.html)
- Positive dataset contains one sequence per error coordinate on the correct genome build (Hg18), and control sequences derive from non-error regions

## Limitations

- DiNAMO is discriminative: it requires both a positive and a negative dataset, so motif discovery fails without a well-matched control
- Fixed-position mode only processes motifs at a specific offset (-p) related to the end of each sequence, so offsets must be consistent with how the error sequences were extracted
- DiNAMO runs on a desktop machine but is bounded by memory (< 8 GB RAM in the article's tests); the article's supplementary Table 3 with execution time/memory is not retrievable from this context

## Evidence

- [results] This compilation contains 3272 genomic coordinates of sequencing errors on the Human genome (Hg18): "we used the monocyte dataset described in [43]. This compilation contains 3272 genomic coordinates of sequencing errors on the Human genome (Hg18)"
- [readme] DiNAMO takes as input two fasta files, which contain respectively the positive dataset (signal.fa) and the negative dataset (control.fa): "DiNAMO takes as input two fasta files, which contain respectively the positive dataset (signal.fa) and the negative dataset (control.fa). You should specify also the motif length '-l'."
- [readme] Fixed-position mode restricts motifs to a specific offset relative to the end of each sequence: "*fixed-position* mode (see optional parameter *-p* ), where only motifs occurring at a specific position in the sequences are taken into account."
- [methods] The Fisher's exact test P-values are Holm-Bonferroni adjusted before filtering motifs: [evidence span withheld — anchor status: not_found]
- [readme] Output motifs are delivered in MEME format: "The output file contains the over-represented motifs found in the positive dataset compared to the negative dataset, in the MEME format"
