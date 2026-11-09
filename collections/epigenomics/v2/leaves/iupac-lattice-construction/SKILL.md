---
name: iupac-lattice-construction
description: Use when you need to enumerate all degenerate IUPAC motifs of a given
  length L (e.g., 6-mers for TFBS discovery) from counted L-mers of DNA sequences,
  ensuring no candidate motif is missed.
when_to_use_negative:
- You are searching for motifs with unrestricted positions in sequences where a scanning-window
  enumeration is too costly and a heuristic/EM approach (MEME, Weeder, HOMER) is acceptable
  — this skill is specific to exact IUPAC enumeration
- Motifs are not of fixed length L, or you cannot tolerate the degeneracy cap d —
  the lattice is built only for motifs of a fixed length bounded by the degenerate-letter
  threshold
- You need PSSM/frequency-matrix motif models rather than discrete IUPAC consensus
  motifs
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_3125
tools:
- name: DiNAMO
  role: Implements the hashtable counting and IUPAC lattice construction via exhaustive
    generalization along the 4-level character lattice; released as binaries for Windows,
    OS X and Linux
  repo: https://github.com/bonsai-team/DiNAMO
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-018-2215-1
    title: 'DiNAMO: highly sensitive DNA motif discovery in high-throughput sequencing
      data'
  source_content_hash: sha256:1c0897ab50e425ec83afb2783166ad240441d8dd96dd745b6deb987717d09c16
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bonsai-team/DiNAMO
  tools:
  - DiNAMO
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_3125
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# iupac-lattice-construction

## Summary

Constructs the exact lattice of degenerate IUPAC motifs of length L by generalizing all L-mers observed in a positive dataset position by position along the 4-level IUPAC character hierarchy, keeping only motifs whose every exact instance exists in the L-mer hashtable. This is DiNAMO's core mechanism for exploring the full motif space without heuristic restriction.

## When to use

Use when you need to enumerate all degenerate IUPAC motifs of a given length L (e.g., 6-mers for TFBS discovery) from counted L-mers of DNA sequences, ensuring no candidate motif is missed.

## When NOT to use

- You are searching for motifs with unrestricted positions in sequences where a scanning-window enumeration is too costly and a heuristic/EM approach (MEME, Weeder, HOMER) is acceptable — this skill is specific to exact IUPAC enumeration
- Motifs are not of fixed length L, or you cannot tolerate the degeneracy cap d — the lattice is built only for motifs of a fixed length bounded by the degenerate-letter threshold
- You need PSSM/frequency-matrix motif models rather than discrete IUPAC consensus motifs

## Inputs

- Positive dataset in FASTA format (e.g., signal.fa)
- Negative/control dataset in FASTA format (e.g., control.fa)
- Motif length L (CLI parameter -l, e.g., -l 6)
- Optional maximum number of degenerate letters d (CLI parameter -d)

## Outputs

- Hashtable mapping each L-mer present in the positive dataset to its occurrence counts in both datasets
- Lattice of IUPAC motifs of length L containing exactly the set of all IUPAC motifs whose instances are all present in the positive dataset
- Per-node contingency tables with summed occurrence counts for each IUPAC motif

## How to apply

Count occurrences of every L-mer in the positive (and control) FASTA files and store the counts in a hashtable (DiNAMO uses Sparsepp). Seed the bottom lattice level with all exact DNA L-mers present in the hashtable. For each motif and each position, query the hashtable for alternative nucleotides and merge them into IUPAC symbols following the character lattice A,C,G,T → K,M,R,S,W,Y → B,D,H,V → N, only merging when both constituent words are present (e.g., CACT and TACT generalize to YACT, but the YAST counter-example must not be produced). Repeat generalization position by position until the degeneracy threshold d (maximum number of degenerate letters, parameter -d) is reached, and sum the hashtable counts of all exact instances for each generated IUPAC motif to build per-node contingency tables used later for mutual information and Fisher's exact test filtering.

## Related tools

- **DiNAMO** (Implements the hashtable counting and IUPAC lattice construction via exhaustive generalization along the 4-level character lattice; released as binaries for Windows, OS X and Linux) — https://github.com/bonsai-team/DiNAMO

## Examples

```
dinamo -pf signal.fa -nf control.fa -l 6
```

## Evaluation signals

- The final lattice contains exactly the set of all IUPAC motifs of length L whose instances are all present in the positive dataset — no missing and no spurious nodes
- The counter-example YAST (from the paper's Fig. 2) is absent from the lattice since not all of its exact instances exist in the hashtable
- Every emitted degenerate motif generalization is justified: a merge at a position (e.g., CACT + TACT → YACT) occurs only because both constituent words are present in the hashtable
- Each lattice node's contingency table counts equal the sum of hashtable counts of all its exact instances
- Lattice construction completes on a desktop machine with less than 8 GB of RAM

## Limitations

- Motif length L and maximum degeneracy d must be fixed in advance; the lattice only covers IUPAC motifs of that fixed length and degeneracy bound
- The lattice contains every IUPAC motif with all instances present, including insignificant ones — significance is only assessed afterwards via MI simplification and Fisher's exact test with Holm–Bonferroni correction
- Runtime and memory grow with dataset size and motif length; DiNAMO targets desktop-scale use (< 8 GB RAM per Table 3)

## Evidence

- [methods] From the hashtable of L-mers, we generate all IUPAC motifs of length L for which all instances are present in the hashtable.: "From the hashtable of L-mers, we generate all IUPAC motifs of length L for which all instances are present in the hashtable."
- [methods] At the end of the process, the lattice contains exactly the set of all IUPAC motifs for which all instances are present in: "At the end of the process, the lattice contains exactly the set of all IUPAC motifs for which all instances are present in"
- [methods] The first step of the algorithm consists in counting the number of occurrences of each existing L-mer in the two input files: "The first step of the algorithm consists in counting the number of occurrences of each existing L-mer in the two input files"
- [methods] DiNAMO is implemented in C ++ using the libraries Sparsepp [26] and boost [27].: "DiNAMO is implemented in C ++ using the libraries Sparsepp [26] and boost [27]."
- [readme] You should specify also the motif length '-l'. Optionally, you can provide a maximum number of degenerate letters '-d'.: "You should specify also the motif length '-l'. Optionally, you can provide a maximum number of degenerate letters '-d'."
