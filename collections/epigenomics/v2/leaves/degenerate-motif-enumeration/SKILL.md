---
name: degenerate-motif-enumeration
description: Use when you need to discover over-represented degenerate DNA motifs
  exhaustively — e.g. from ChIP-seq peak sequences or position-specific sequencing-error
  motifs — without restricting the motif space to a predefined consensus or PWM family.
when_to_use_negative:
- When only a small set of candidate consensus motifs is of interest — a direct scan
  with known motifs avoids lattice construction entirely
- When motifs must be modeled as full position-specific frequency matrices rather
  than IUPAC degenerate words (DiNAMO searches IUPAC models, not PWMs)
- 'When memory-constrained environments cannot hold the L-mer hashtable: although
  DiNAMO runs on a desktop machine with < 8 GB RAM, very large genomes with long L
  may exceed this'
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_3511
- http://edamontology.org/topic_0622
tools:
- name: DiNAMO
  role: Implements the exhaustive L-mer counting, IUPAC lattice construction, MI-based
    simplification, and Fisher's exact test filtering for degenerate motif discovery
  repo: https://github.com/bonsai-team/DiNAMO
- name: RSAT
  role: Prior art for exact-string motif discovery approaches that DiNAMO's exact
    enumeration generalizes
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-018-2215-1
    title: 'DiNAMO: highly sensitive DNA motif discovery in high-throughput sequencing
      data'
  source_content_hash: sha256:a63ce0ae9d08333b2dcb682da9b47b23394fb658677ab4e3dfdb216cb89bd458
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bonsai-team/DiNAMO
  tools:
  - DiNAMO
  - RSAT
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_3511
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# degenerate-motif-enumeration

## Summary

Exhaustively enumerates all length-L IUPAC degenerate DNA motifs whose exact L-mer instances are all present in a positive dataset, using a hashtable of L-mer counts and a 4-level character lattice (A,C,G,T → K,M,R,S,W,Y → B,D,H,V → N), as implemented in DiNAMO. This guarantees no restriction on the motif search space while keeping enumeration tractable.

## When to use

Use when you need to discover over-represented degenerate DNA motifs exhaustively — e.g. from ChIP-seq peak sequences or position-specific sequencing-error motifs — without restricting the motif space to a predefined consensus or PWM family.

## When NOT to use

- When only a small set of candidate consensus motifs is of interest — a direct scan with known motifs avoids lattice construction entirely
- When motifs must be modeled as full position-specific frequency matrices rather than IUPAC degenerate words (DiNAMO searches IUPAC models, not PWMs)
- When memory-constrained environments cannot hold the L-mer hashtable: although DiNAMO runs on a desktop machine with < 8 GB RAM, very large genomes with long L may exceed this

## Inputs

- Positive dataset (signal) in FASTA format, e.g. ChIP-seq peak sequences
- Negative/control dataset in FASTA format for count comparison
- Motif length L (CLI parameter -l, e.g. -l 6)
- Optional maximum number of degenerate letters d (CLI parameter -d)

## Outputs

- Hashtable mapping each exact L-mer to its occurrence counts in signal and control datasets
- A lattice of IUPAC motifs of length L, containing exactly all motifs whose instances are all present in the hashtable
- Per-motif contingency tables (signal/control counts summed over exact instances) feeding MI and Fisher's exact test
- Over-represented motifs output in MEME format

## How to apply

Count occurrences of every L-mer in the positive (and control) input files and store them in a hashtable (DiNAMO uses Sparsepp). Build bottom-level lattice nodes from the exact DNA L-mers in the hashtable, then generalize position by position: for each motif and each position, query the hashtable for alternative nucleotides present at that position and merge them into IUPAC symbols following the character lattice (e.g. CACT and TACT merge into YACT only because both words are present), stopping when the degeneracy threshold -d is reached. The resulting lattice contains exactly the set of all IUPAC motifs of length L whose instances are all present in the positive dataset; for each generated motif, sum the hashtable counts of all its exact instances to fill its contingency table. Only afterwards rank/simplify motifs with mutual information and apply Fisher's exact test with Holm–Bonferroni correction — enumeration itself is deliberately significance-agnostic so that weak but real motifs (implanted at frequencies as low as 0.5%) are not lost.

## Related tools

- **DiNAMO** (Implements the exhaustive L-mer counting, IUPAC lattice construction, MI-based simplification, and Fisher's exact test filtering for degenerate motif discovery) — https://github.com/bonsai-team/DiNAMO
- **RSAT** (Prior art for exact-string motif discovery approaches that DiNAMO's exact enumeration generalizes)

## Examples

```
dinamo -pf signal.fa -nf control.fa -l 6
```

## Evaluation signals

- Verify the produced lattice contains exactly the set of all IUPAC motifs of length L whose instances are all present in the hashtable — spot-check that counter-examples like the paper's Fig. 2 YAST motif (whose instances are not all present) are absent
- Check that every emitted degenerate motif's instances all exist in the positive dataset hashtable (no edge/motif is emitted with missing instances)
- Confirm per-node contingency tables equal the sum of hashtable counts over all exact instances of each motif
- Confirm enumeration happens before significance filtering: motifs should not be pruned by MI or Fisher's exact test during lattice construction
- Sanity-check implanted-motif recovery: on synthetic benchmarks, motifs implanted at frequencies down to 0.5% should still be enumerated (DiNAMO was evaluated with nCC at 5%–0.5% implantation frequencies)

## Limitations

- Enumeration is limited to fixed motif length L given by -l; separate runs are needed per length
- The degeneracy cap -d (maximum number of degenerate letters) bounds the lattice size; motifs requiring more degenerate positions are not generated
- The lattice contains all constructible motifs, not only significant ones — downstream MI simplification and Fisher's exact test (with Holm–Bonferroni adjustment) are required to select motifs
- DiNAMO searches IUPAC degenerate words, so motif models richer than IUPAC (e.g. full PWMs) are out of scope

## Evidence

- [methods] From the hashtable of L-mers, we generate all IUPAC motifs of length L for which all instances are present in the hashtable.: "From the hashtable of L-mers, we generate all IUPAC motifs of length L for which all instances are present in the hashtable."
- [methods] At the end of the process, the lattice contains exactly the set of all IUPAC motifs for which all instances are present in: "At the end of the process, the lattice contains exactly the set of all IUPAC motifs for which all instances are present in"
- [methods] The first step of the algorithm consists in counting the number of occurrences of each existing L-mer in the two input files: "The first step of the algorithm consists in counting the number of occurrences of each existing L-mer in the two input files"
- [readme] DiNAMO takes as input two fasta files, which contain respectively the positive dataset (signal.fa) and the negative dataset (control.fa). You should specify also the motif length '-l'.: "DiNAMO takes as input two fasta files, which contain respectively the positive dataset (signal.fa) and the negative dataset (control.fa). You should specify also the motif length '-l'."
- [intro] It is possible to efficiently search for weakly represented IUPAC motifs in a large signal dataset, compared to the control dataset, without any restriction on the motif: "it is possible to efficiently search for weakly represented IUPAC motifs in a large signal dataset, compared to the control dataset, without any restriction on the motif"
