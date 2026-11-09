---
name: iupac-motif-lattice-simplification
description: Use when you have an IUPAC motif lattice built from a positive and a
  negative FASTA dataset and need to score, rank, and prune over-represented degenerate
  motifs without bias toward occurrence count.
when_to_use_negative:
- You need statistical significance assessment rather than dominance-based pruning
  — DiNAMO applies Fisher's exact test with Holm–Bonferroni adjustment as a separate
  downstream filtering step
- The input is raw unfiltered reads rather than a lattice whose motifs have all instances
  present in the positive dataset — the final lattice contains exactly the IUPAC motifs
  whose instances are all present in the positive dataset, so it must be built first
- You need position-specific motif search via expectation maximization (e.g. MEME-style)
  instead of exhaustive discriminative IUPAC enumeration
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0080
tools:
- name: DiNAMO
  role: Implements the exhaustive IUPAC lattice construction, MI-based lattice simplification,
    and subsequent Fisher's exact test filtering; input is two FASTA files plus motif
    length, output is over-represented motifs in MEME format
  repo: https://github.com/bonsai-team/DiNAMO
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-018-2215-1
    title: 'DiNAMO: highly sensitive DNA motif discovery in high-throughput sequencing
      data'
  source_content_hash: sha256:da2f1210ac139a13e69dbd52f7eb46338361872626daa75b1a08c03504dddd6d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bonsai-team/DiNAMO
  tools:
  - DiNAMO
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# iupac-motif-lattice-simplification

## Summary

Simplifies the DiNAMO IUPAC motif lattice by ranking all candidate degenerate motifs with a mutual-information (MI) objective function computed from per-node 2x2 contingency tables, then greedily retaining non-dominated motifs. MI captures motif–dataset dependency independently of occurrence count, unlike Fisher's exact test P-value, so short degenerate ancestors do not outrank their specific descendants.

## When to use

Use when you have an IUPAC motif lattice built from a positive and a negative FASTA dataset and need to score, rank, and prune over-represented degenerate motifs without bias toward occurrence count.

## When NOT to use

- You need statistical significance assessment rather than dominance-based pruning — DiNAMO applies Fisher's exact test with Holm–Bonferroni adjustment as a separate downstream filtering step
- The input is raw unfiltered reads rather than a lattice whose motifs have all instances present in the positive dataset — the final lattice contains exactly the IUPAC motifs whose instances are all present in the positive dataset, so it must be built first
- You need position-specific motif search via expectation maximization (e.g. MEME-style) instead of exhaustive discriminative IUPAC enumeration

## Inputs

- IUPAC motif lattice built from the positive dataset, where each vertex carries a 2x2 contingency table (counts of motif occurrence in positive vs negative datasets)
- Positive dataset FASTA (e.g. signal.fa) and negative dataset FASTA (e.g. control.fa) used to build the contingency tables
- Motif length L (set via the '-l' parameter) used during lattice construction

## Outputs

- Ranked list of retained (non-dominated) IUPAC motifs with their MI scores, written to mi_selected_motifs.tsv
- Simplified lattice with dominated descendants and ancestors eliminated

## How to apply

Load the lattice produced by the lattice-construction stage, where each vertex carries a 2x2 contingency table (a = positive sequences containing the motif, b = negative sequences containing the motif, c and d = remaining sequences). For every node, normalize the contingency table over the total sequence count to estimate joint probabilities p(O_i, C_j), where O is binary motif presence/absence and C is the positive/negative condition, and compute MI(O;C) = sum over i in {0,1}, j in {P,N} of p(O_i,C_j) * log2(p(O_i,C_j) / (p(O_i)*p(C_j))). Sort all motifs by decreasing MI, then iteratively take the highest-MI non-deleted motif, add it to the result list, and delete all of its descendants and ancestors since those are dominated. Continue until every motif has been selected or deleted, and write the retained motifs with their MI scores. Rationale: MI captures the dependency between a motif and a dataset independently from its number of occurrences, unlike Fisher's exact test P-value.

## Related tools

- **DiNAMO** (Implements the exhaustive IUPAC lattice construction, MI-based lattice simplification, and subsequent Fisher's exact test filtering; input is two FASTA files plus motif length, output is over-represented motifs in MEME format) — https://github.com/bonsai-team/DiNAMO

## Examples

```
dinamo -pf signal.fa -nf control.fa -l 6 -o mi_selected_motifs.tsv
```

## Evaluation signals

- Dominance invariant: each kept motif has MI greater than each of its descendants and is not dominated by any ancestor
- MI values recomputed from the stored contingency tables match the stored MI scores within numerical tolerance
- The retained set is disjoint from all deleted (dominated) motifs: every lattice motif is either selected or a descendant/ancestor of a selected motif
- MI ranking is not correlated with the number of motif occurrences — short, frequent ancestors of a high-MI motif should not displace it

## Limitations

- The MI objective alone does not provide a significance test; a separate Fisher's exact test with Holm–Bonferroni correction (default p-value threshold 0.05 via '-t') is applied downstream
- The lattice only contains IUPAC motifs whose instances are all present in the positive dataset, so motifs partially present in the positive set are never scored
- Execution time and memory (Table 3) are bounded but large datasets can be heavy; DiNAMO is designed for desktop machines with less than 8 GB of RAM

## Evidence

- [methods] MI objective and dominance pruning: "the MI (mutual information) captures the dependency between a motif and a dataset independently from its number of occurrences."
- [methods] Lattice simplification step: "we eliminate useless motifs of the lattice that could not improve previously selected motifs, based on their MI."
- [intro] MI as objective function: "It uses mutual information (MI) as an objective function to search for over-represented degenerate motifs"
- [methods] Contingency table per lattice node: "each vertex of the IUPAC lattice has a contingency table that contains the counts for the files"
- [readme] DiNAMO CLI usage: "dinamo -pf signal.fa  -nf control.fa -l 6"
