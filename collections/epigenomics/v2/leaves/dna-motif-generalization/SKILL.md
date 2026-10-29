---
name: dna-motif-generalization
description: Use when you need to enumerate all degenerate IUPAC motifs of length
  L whose exact instances all occur in a positive DNA dataset, or to reimplement/validate
  lattice-based motif generalization from counted L-mers.
when_to_use_negative:
- When a heuristic or restricted search space is acceptable and exhaustiveness is
  not required — this skill's value is exactly that no motif is excluded a priori
- When motifs are variable-length or gapped (DiNAMO builds fixed-length IUPAC motifs
  of length L in scanning or fixed-position mode)
- When input is protein sequence or already-degenerate consensus strings rather than
  raw DNA from which exact L-mers must be counted
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_3295
tools:
- name: DiNAMO
  role: 'Implements the exhaustive IUPAC lattice construction: counts L-mers, builds
    the lattice from the hashtable, and scores degenerate motifs over-represented
    in the positive vs negative FASTA datasets'
  repo: https://github.com/bonsai-team/DiNAMO
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-018-2215-1
    title: 'DiNAMO: highly sensitive DNA motif discovery in high-throughput sequencing
      data'
  source_content_hash: sha256:8ea3047752bc96b6af6f3e8d2191af418aa4a317f2b4c95ef91d4e4459797d43
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bonsai-team/DiNAMO
  tools:
  - DiNAMO
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# dna-motif-generalization

## Summary

Reconstructs DiNAMO's exhaustive construction of degenerate IUPAC motifs by generalizing exact DNA L-mers position-by-position along the 4-level IUPAC character lattice (A,C,G,T → K,M,R,S,W,Y → B,D,H,V → N), keeping only motifs whose every exact instance is present in the hashtable of counted L-mers, so the motif space is never restricted a priori.

## When to use

Use when you need to enumerate all degenerate IUPAC motifs of length L whose exact instances all occur in a positive DNA dataset, or to reimplement/validate lattice-based motif generalization from counted L-mers.

## When NOT to use

- When a heuristic or restricted search space is acceptable and exhaustiveness is not required — this skill's value is exactly that no motif is excluded a priori
- When motifs are variable-length or gapped (DiNAMO builds fixed-length IUPAC motifs of length L in scanning or fixed-position mode)
- When input is protein sequence or already-degenerate consensus strings rather than raw DNA from which exact L-mers must be counted

## Inputs

- Hashtable of counted L-mers from the positive dataset (occurrence counts per L-mer of length L)
- Two input FASTA files: positive dataset (signal.fa) and negative dataset (control.fa)
- Motif length L (e.g. -l 6)
- Optional maximum number of degenerate letters d (threshold for stopping generalization)

## Outputs

- A lattice of IUPAC motifs of length L whose all exact instances are present in the positive dataset
- Per-motif summed occurrence counts (exact instances summed from the hashtable) feeding contingency tables for MI and Fisher's exact test

## How to apply

Load the hashtable mapping each L-mer present in the positive dataset (and its counts from both signal and control FASTA files) to occurrences, then create bottom-level lattice nodes from all exact DNA L-mers in the hashtable. For each motif and each position, query the hashtable for alternative nucleotides at that position and merge them into IUPAC symbols following the 4-level character lattice (two-letter symbols K,M,R,S,W,Y; three-letter symbols B,D,H,V; then N) — e.g. generalize CACT with TACT into YACT only because both words are present. Repeat the generalization position by position until the degeneracy threshold d (the -d parameter, the maximum number of degenerate letters) is reached, emitting only edges/motifs whose instances are all in the hashtable. For every generated IUPAC motif, sum the hashtable counts of all its exact instances to populate its contingency table for later MI and Fisher's exact test filtering.

## Related tools

- **DiNAMO** (Implements the exhaustive IUPAC lattice construction: counts L-mers, builds the lattice from the hashtable, and scores degenerate motifs over-represented in the positive vs negative FASTA datasets) — https://github.com/bonsai-team/DiNAMO

## Examples

```
dinamo -pf signal.fa -nf control.fa -l 6 -d 3 -o motifs.out
```

## Evaluation signals

- The produced lattice contains exactly the set of all IUPAC motifs of length L whose instances are all present in the hashtable — no missing and no extra motifs
- Spot-check the YAST counter-example (from the paper's Fig. 2): it must be absent because its instances are not all present in the hashtable
- Generalization like CACT+TACT→YACT is emitted only when both exact words are present in the hashtable; partially supported merges are rejected
- For every emitted IUPAC motif, the summed count equals the sum of hashtable counts over all its exact instances
- Output is written in MEME format listing over-represented motifs found in the positive vs negative dataset

## Limitations

- Generalization stops at the degeneracy threshold d; without -d the number of degenerate letters is capped by the implementation defaults
- DiNAMO targets fixed-length motifs (-l); it is implemented in C++ using Sparsepp and boost, and is designed to run on a desktop machine with less than 8 GB of RAM, but very large genomes/motif lengths increase memory and runtime (Table 3 in the paper reports execution time and memory)
- Lattice construction only enumerates motifs; significance requires downstream MI-based lattice simplification and Fisher's exact test with Holm–Bonferroni adjustment

## Evidence

- [methods] From the hashtable of L-mers, we generate all IUPAC motifs of length L for which all instances are present in the hashtable.: "From the hashtable of L-mers, we generate all IUPAC motifs of length L for which all instances are present in the hashtable."
- [methods] At the end of the process, the lattice contains exactly the set of all IUPAC motifs for which all instances are present in: "At the end of the process, the lattice contains exactly the set of all IUPAC motifs for which all instances are present in"
- [methods] The first step of the algorithm consists in counting the number of occurrences of each existing L-mer in the two input files: "The first step of the algorithm consists in counting the number of occurrences of each existing L-mer in the two input files"
- [abstract] DiNAMO software can serve as a tool to search for degenerate motifs in an exact manner using IUPAC models.: "DiNAMO software can serve as a tool to search for degenerate motifs in an exact manner using IUPAC models."
- [readme] DiNAMO takes as input two fasta files, which contain respectively the positive dataset (signal.fa) and the negative dataset (control.fa).: "DiNAMO takes as input two fasta files, which contain respectively the positive dataset (signal.fa) and the negative dataset (control.fa)."
