---
name: chipseq-overrepresented-motif-discovery
description: Use when you have a positive multi-FASTA of ChIP-seq peak sequences (e.g.
  GATA1, SOX2, OCT4, STAT3, KLF1 peaks) plus a negative control FASTA, and need exhaustive
  IUPAC motif discovery to identify over-represented TF binding sites.
when_to_use_negative:
- There is no negative control dataset — DiNAMO is inherently discriminative and requires
  both a positive (signal.fa) and a negative (control.fa) dataset to compute contingency
  tables and Fisher's exact test P-values
- You need variable-length, gapped, or position-weight-matrix motifs rather than fixed-length
  IUPAC degenerate motifs of a single length L
- The question is motif occurrence at a known fixed offset relative to sequence ends
  (e.g. fixed-position sequencing-error motifs) — that requires DiNAMO's '-p' fixed-position
  mode rather than default scanning mode
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_0749
tools:
- name: DiNAMO
  role: 'Primary motif discovery engine: exhaustively enumerates over-represented
    IUPAC motifs from signal vs control FASTA inputs using a mutual-information-simplified
    lattice and Fisher''s exact test filtering, emitting MEME-format motifs'
  repo: https://github.com/bonsai-team/DiNAMO
- name: JASPAR
  role: Reference database of transcription factor frequency matrices used to validate
    that the top discovered motif corresponds to the expected ChIP-seq factor's binding
    motif
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-018-2215-1
    title: 'DiNAMO: highly sensitive DNA motif discovery in high-throughput sequencing
      data'
  source_content_hash: sha256:82af42df3392229e7be3dc8cb37d22dc54160dc3c87938d4e965bb38ac4fd01a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bonsai-team/DiNAMO
  tools:
  - DiNAMO
  - JASPAR
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# chipseq-overrepresented-motif-discovery

## Summary

Discriminative discovery of over-represented degenerate IUPAC motifs in ChIP-seq peak sequences using DiNAMO's exhaustive lattice algorithm, which compares a positive signal dataset against a negative control dataset and reports motifs passing a Holm–Bonferroni-adjusted Fisher's exact test. Applied in scanning (sliding-window) mode it recovers the expected transcription factor binding motif plus secondary co-factor motifs.

## When to use

Use when you have a positive multi-FASTA of ChIP-seq peak sequences (e.g. GATA1, SOX2, OCT4, STAT3, KLF1 peaks) plus a negative control FASTA, and need exhaustive IUPAC motif discovery to identify over-represented TF binding sites.

## When NOT to use

- There is no negative control dataset — DiNAMO is inherently discriminative and requires both a positive (signal.fa) and a negative (control.fa) dataset to compute contingency tables and Fisher's exact test P-values
- You need variable-length, gapped, or position-weight-matrix motifs rather than fixed-length IUPAC degenerate motifs of a single length L
- The question is motif occurrence at a known fixed offset relative to sequence ends (e.g. fixed-position sequencing-error motifs) — that requires DiNAMO's '-p' fixed-position mode rather than default scanning mode

## Inputs

- Positive dataset in multi-FASTA format (signal.fa) containing ChIP-seq peak sequences for one factor (e.g. GATA1: 14,351 sequences; SOX2: 4,526; OCT4: 3,761; STAT3: 2,546; KLF1: 945)
- Negative control dataset in multi-FASTA format (control.fa)
- Motif length L (integer, via -l)
- Optional: maximum number of degenerate letters i (via -d), Fisher's exact test P-value threshold f (via -t, default 0.05), output file path (via -o)

## Outputs

- Output file of over-represented IUPAC motifs in MEME format (dinamo_motifs.txt), containing only motifs with Holm–Bonferroni-adjusted Fisher's exact test P-value below the threshold
- Optional clusters of overlapping motifs grouped by sequence similarity (minimum overlap of half the motif size)

## How to apply

Run DiNAMO in its default scanning mode with a positive dataset (signal.fa, the ChIP-seq peaks) and a negative control dataset (control.fa), specifying the motif length with '-l' (e.g. L=6 in the documented command) and optionally the maximum number of degenerate letters with '-d' (the paper's ChIP-seq case study used IUPAC motifs of length L=7 with up to 3 degenerate letters). DiNAMO counts L-mer occurrences in both input files, stores them in a hashtable, and builds the lattice of IUPAC motifs whose instances are all present in the positive dataset; it then computes mutual information from contingency tables to simplify the lattice and retain dominant non-dominated motifs, chosen because MI captures motif–dataset dependency independently of occurrence count. Statistical filtering computes Fisher's exact test P-values with Holm–Bonferroni adjustment, keeping only motifs below the default 0.05 threshold; optional post-processing clusters overlapping over-represented motifs based on sequence similarity with a minimum overlap of half the motif size, and secondary motifs are detected by masking instances of the found motif in the lattice. The output is written in MEME format (dinamo_motifs.txt) and the top motif should be validated against the corresponding factor's frequency matrix in the JASPAR database.

## Related tools

- **DiNAMO** (Primary motif discovery engine: exhaustively enumerates over-represented IUPAC motifs from signal vs control FASTA inputs using a mutual-information-simplified lattice and Fisher's exact test filtering, emitting MEME-format motifs) — https://github.com/bonsai-team/DiNAMO
- **JASPAR** (Reference database of transcription factor frequency matrices used to validate that the top discovered motif corresponds to the expected ChIP-seq factor's binding motif)

## Examples

```
dinamo -pf signal.fa -nf control.fa -l 6
```

## Evaluation signals

- Output is a well-formed MEME-format motif file (per http://meme-suite.org/doc/meme-format.html) containing only motifs whose Holm–Bonferroni-adjusted Fisher's exact test P-value is below 0.05
- On GATA1, SOX2, OCT4, STAT3 and KLF1 ChIP-seq peak datasets, the top motif matches the corresponding factor's frequency matrix in the JASPAR database
- Secondary (co-factor) motifs are additionally reported, consistent with the paper's finding of multiple secondary motifs on the full ChIP-seq peak datasets
- The reported motif is a dominant non-dominated lattice node, i.e. it survived mutual-information-based lattice simplification
- Execution fits a desktop machine with less than 8 GB of RAM, per the paper's resource report

## Limitations

- The lattice contains exactly the set of all IUPAC motifs whose instances are all present in the positive dataset — motifs with even one absent instance are excluded by construction
- Fixed-length constraint: a single motif length L must be chosen per run; scanning mode parses all windows of length L one base at a time, so multiple lengths require multiple runs
- Default degeneracy and Fisher P-value threshold (0.05) may need tuning via '-d' and '-t'; the paper's synthetic benchmark showed sensitivity depends on implanted motif frequency (5% down to 0.5%)
- Optional clustering of overlapping motifs depends on sequence similarity with a minimum overlap of half the motif size; without it, redundant overlapping motifs may be reported separately

## Evidence

- [abstract] DiNAMO proves to compare favorably with other existing methods and is robust to noise.: "DiNAMO proves to compare favorably with other existing methods and is robust to noise."
- [methods] L-mer counting is the first algorithmic step on both inputs.: "The first step of the algorithm consists in counting the number of occurrences of each existing L-mer in the two input files"
- [methods] Holm–Bonferroni correction guards against multiple comparisons.: "The Holm–Bonferroni method [25] is applied to adjust the P-values and counteract the problem of multiple comparisons."
- [results] Validation compares discovered motifs against JASPAR frequency matrices.: "comparing them against the frequency matrices of the JASPAR database [34]"
- [readme] README documents the command line and MEME-format output.: "The output file contains the over-represented motifs found in the positive dataset compared to the negative dataset, in the MEME format"
