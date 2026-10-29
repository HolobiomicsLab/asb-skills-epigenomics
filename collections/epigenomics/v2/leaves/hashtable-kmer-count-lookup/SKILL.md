---
name: hashtable-kmer-count-lookup
description: Use when performing exhaustive, restriction-free motif discovery over
  DNA sequences where degenerate IUPAC motifs must be built from exact constituent
  instances, requiring per-L-mer occurrence counts in signal and control files.
when_to_use_negative:
- When the motif space should be seeded from a position-specific scoring matrix or
  consensus rather than built exhaustively from exact instances present in the data
- When only motifs at known fixed positions relative to sequence ends are of interest,
  since hashtable-backed lattice generation enumerates all windows (scanning mode)
  unless -p is used
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3168
tools:
- name: DiNAMO
  role: Implements the L-mer counting, Sparsepp hashtable storage, and hashtable-driven
    IUPAC lattice construction
  repo: https://github.com/bonsai-team/DiNAMO
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-018-2215-1
    title: 'DiNAMO: highly sensitive DNA motif discovery in high-throughput sequencing
      data'
  source_content_hash: sha256:26fadfaf850c9ff76c5aa6b576f9ee953a5fe951e3f9dd85090c4c220b1792a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bonsai-team/DiNAMO
  tools:
  - DiNAMO
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# hashtable-kmer-count-lookup

## Summary

Count every L-mer present in positive and negative DNA sequence datasets and store occurrence counts in a hashtable, providing the substrate for exhaustive IUPAC motif lattice construction in DiNAMO. The hashtable allows constant-time lookup of exact motif instances during degenerate motif generalization.

## When to use

Use when performing exhaustive, restriction-free motif discovery over DNA sequences where degenerate IUPAC motifs must be built from exact constituent instances, requiring per-L-mer occurrence counts in signal and control files.

## When NOT to use

- When the motif space should be seeded from a position-specific scoring matrix or consensus rather than built exhaustively from exact instances present in the data
- When only motifs at known fixed positions relative to sequence ends are of interest, since hashtable-backed lattice generation enumerates all windows (scanning mode) unless -p is used

## Inputs

- Positive dataset FASTA file (signal.fa), e.g. ChIP-seq peak sequences
- Negative dataset FASTA file (control.fa)
- Motif length parameter L (e.g. -l 6)

## Outputs

- Hashtable mapping each L-mer present in the input files to its occurrence counts per dataset
- Per-motif contingency tables (counts in positive vs negative datasets) derived by summing exact-instance counts

## How to apply

Count the occurrences of each existing L-mer (length L, e.g. L=6) in both input FASTA files (positive/signal and negative/control). Store the resulting L-mers and their per-dataset counts in a sparse hashtable (DiNAMO uses the Sparsepp C++ library). During lattice construction, query the hashtable at each motif position to find alternative nucleotides present in the data and merge them into IUPAC symbols only when all constituent exact words are present in the hashtable (e.g. CACT and TACT generalize to YACT, but YACT is only emitted if all four Y-instances exist). For every generated IUPAC motif, sum the hashtable counts of all its exact instances to fill the contingency table used for mutual information scoring and Fisher's exact test.

## Related tools

- **DiNAMO** (Implements the L-mer counting, Sparsepp hashtable storage, and hashtable-driven IUPAC lattice construction) — https://github.com/bonsai-team/DiNAMO

## Examples

```
dinamo -pf signal.fa -nf control.fa -l 6
```

## Evaluation signals

- The produced IUPAC lattice contains exactly the set of all IUPAC motifs of length L whose instances are all present in the hashtable — spot-check counter-examples such as the YAST motif from the paper's Fig. 2, which must be absent
- No generalized motif is emitted unless every constituent exact L-mer is present in the hashtable (e.g. YACT only when CACT and TACT and the other Y-instances all occur)
- Summed exact-instance counts for each IUPAC motif match independent recounting of the input FASTA files
- Counts are recorded separately for the positive (signal.fa) and negative (control.fa) datasets so contingency tables can be built

## Limitations

- The hashtable only covers L-mers actually present in the input datasets; motifs with missing exact instances are never generated, by design
- Memory and runtime scale with dataset size; DiNAMO targets desktop machines (< 8 GB RAM per Table 3)
- Default P-value threshold is 0.05 (changeable with -t); no software version is stated in the provided context

## Evidence

- [methods] Counting and storage of L-mers: "The first step of the algorithm consists in counting the number of occurrences of each existing L-mer in the two input files"
- [methods] Lattice construction from hashtable: "From the hashtable of L-mers, we generate all IUPAC motifs of length L for which all instances are present in the hashtable."
- [methods] Lattice completeness invariant: "At the end of the process, the lattice contains exactly the set of all IUPAC motifs for which all instances are present in"
- [methods] Contingency tables from hashtable counts: "each vertex of the IUPAC lattice has a contingency table that contains the counts for the files"
- [readme] DiNAMO CLI usage: "dinamo -pf signal.fa  -nf control.fa -l 6"
