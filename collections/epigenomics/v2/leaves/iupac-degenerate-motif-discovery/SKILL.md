---
name: iupac-degenerate-motif-discovery
description: Use when a pair of positive (signal.fa) and negative (control.fa) DNA
  FASTA files is available and the goal is to recover over-represented degenerate
  IUPAC motifs, e.g. ChIP-seq peak TF binding sites or motifs inducing systematic
  sequencing errors.
when_to_use_negative:
- When no matched negative control dataset exists, since DiNAMO is a discriminative
  method requiring a positive/control FASTA pair
- When the motif of interest is position-variable but must be modeled as a PWM/probability
  matrix rather than an IUPAC consensus (e.g. MEME-style EM discovery is expected
  instead)
- When searching only exact (non-degenerate) string motifs — the paper notes exact-string
  tools like RSAT exist, and DiNAMO's advantage lies in degenerate IUPAC motifs
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0102
tools:
- name: DiNAMO
  role: 'Primary executable: exhaustive IUPAC lattice construction, MI simplification,
    Fisher filtering, MEME-format motif output; run in scanning (default, sliding
    windows) or fixed-position (-p) mode'
  repo: https://github.com/bonsai-team/DiNAMO
- name: MEME
  role: Comparison baseline (EM-based method) on the synthetic nCC benchmark and provider
    of the output motif format
- name: RSAT
  role: Reference exact-string motif discovery approach contrasted with DiNAMO's degenerate
    IUPAC search
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_002
  source_papers:
  - doi: 10.1186/s12859-018-2215-1
    title: 'DiNAMO: highly sensitive DNA motif discovery in high-throughput sequencing
      data'
  source_content_hash: sha256:b6259bde23bd77463845098cde4ef4e7bb1a349783bbcc87ae3f059c6a3588cd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bonsai-team/DiNAMO
  tools:
  - DiNAMO
  - MEME
  - RSAT
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# iupac-degenerate-motif-discovery

## Summary

Exhaustive discovery of over-represented degenerate IUPAC DNA motifs between a positive (signal) and negative (control) FASTA dataset using DiNAMO, which builds an IUPAC motif lattice, simplifies it with mutual information, and filters by Holm-Bonferroni-adjusted Fisher's exact test (default P-value threshold 0.05).

## When to use

Use when a pair of positive (signal.fa) and negative (control.fa) DNA FASTA files is available and the goal is to recover over-represented degenerate IUPAC motifs, e.g. ChIP-seq peak TF binding sites or motifs inducing systematic sequencing errors.

## When NOT to use

- When no matched negative control dataset exists, since DiNAMO is a discriminative method requiring a positive/control FASTA pair
- When the motif of interest is position-variable but must be modeled as a PWM/probability matrix rather than an IUPAC consensus (e.g. MEME-style EM discovery is expected instead)
- When searching only exact (non-degenerate) string motifs — the paper notes exact-string tools like RSAT exist, and DiNAMO's advantage lies in degenerate IUPAC motifs

## Inputs

- Positive dataset FASTA (multi-FASTA signal.fa, e.g. ChIP-seq peak sequences or sequences extracted at error coordinates)
- Negative control dataset FASTA (control.fa, matched background sequences)
- Motif length L parameter (e.g. -l 6 or -l 7)
- Optional: maximum number of degenerate letters (-d), Fisher P-value threshold (-t), fixed-position offset (-p)

## Outputs

- Over-represented IUPAC motifs in MEME format (default output dinamo_motifs.txt), each with Holm-Bonferroni-adjusted Fisher's exact test P-value below the threshold
- Optionally clustered motif groups for overlapping motifs

## How to apply

Run the DiNAMO executable on the positive/negative multi-FASTA pair, specifying the motif length L with '-l' (e.g. L=6 or L=7 for ChIP-seq peaks) and optionally the maximum number of degenerate letters '-d' (up to 3 in the ChIP-seq case study). DiNAMO first counts all L-mer occurrences in both files into a hashtable, then generates the lattice of all IUPAC motifs of length L whose instances are all present in the positive dataset. It builds a contingency table per lattice node, simplifies the lattice with mutual information (which captures motif-dataset dependency independently of occurrence count), then computes Fisher's exact test P-values adjusted with the Holm-Bonferroni method and keeps motifs below the threshold (default 0.05, tunable via '-t'). Secondary motifs are detected by masking instances in the lattice, and an optional post-processing clusters overlapping motifs sharing at least half the motif size. Results are written in MEME format and can be compared against JASPAR frequency matrices (TF case) or degenerate error-associated motifs (SSE case).

## Related tools

- **DiNAMO** (Primary executable: exhaustive IUPAC lattice construction, MI simplification, Fisher filtering, MEME-format motif output; run in scanning (default, sliding windows) or fixed-position (-p) mode) — https://github.com/bonsai-team/DiNAMO
- **MEME** (Comparison baseline (EM-based method) on the synthetic nCC benchmark and provider of the output motif format)
- **RSAT** (Reference exact-string motif discovery approach contrasted with DiNAMO's degenerate IUPAC search)

## Examples

```
dinamo -pf signal.fa -nf control.fa -l 7 -d 3 -o dinamo_motifs.txt
```

## Evaluation signals

- Output is a well-formed MEME-format motif file containing only motifs with Holm-Bonferroni-adjusted Fisher's exact test P-value below 0.05 (or the '-t' threshold used)
- On the ChIP-seq case study (GATA1, SOX2, OCT4, STAT3, KLF1), the top motif should align with the corresponding transcription factor's frequency matrix in the JASPAR database
- On the synthetic benchmark (100 motif sets x 6 implantation frequencies from 5% to 0.5%), DiNAMO should achieve the best nucleotide level correlation coefficient (nCC) on degenerate motifs, beating MEME-CHIP and Discrover
- On the monocyte SSE dataset, the top reported motif should be NNRGGT (adjusted P-value < 10^-324), refining the previously reported GGT motif
- The lattice must contain exactly all IUPAC motifs of length L whose instances are all present in the positive dataset — an exhaustive-property invariant of the algorithm

## Limitations

- Requires a desktop with less than 8 GB RAM per the paper's resource profile; Table 3 reports execution time and memory but its contents are not retrievable from the text
- The P-value threshold is on Fisher's exact test with Holm-Bonferroni adjustment; motif ranking relies on mutual information rather than occurrence count, so low-count but highly dependent motifs dominate
- Overlapping-motif clustering only merges motifs with a minimum overlap of half the motif size; secondary motif detection may be needed for co-factor motifs
- Binary availability and compilation depend on the Boost library; the Boost include path may need to be set manually in the Makefile

## Evidence

- [methods] From the hashtable of L-mers, we generate all IUPAC motifs of length L for which all instances are present in the hashtable.: "From the hashtable of L-mers, we generate all IUPAC motifs of length L for which all instances are present in the hashtable."
- [methods] we eliminate useless motifs of the lattice that could not improve previously selected motifs, based on their MI.: "we eliminate useless motifs of the lattice that could not improve previously selected motifs, based on their MI."
- [methods] The Holm–Bonferroni method [25] is applied to adjust the P-values: "The Holm–Bonferroni method [25] is applied to adjust the P-values"
- [readme] DiNAMO takes as input two fasta files, which contain respectively the positive dataset (signal.fa) and the negative dataset (control.fa).: "DiNAMO takes as input two fasta files, which contain respectively the positive dataset (signal.fa) and the negative dataset (control.fa)."
- [results] we implanted motifs at 6 different frequencies: 5%, 4%, 3%, 2%, 1% and 0.5% of the number of initial sequences: "we implanted motifs at 6 different frequencies: 5%, 4%, 3%, 2%, 1% and 0.5% of the number of initial sequences"
