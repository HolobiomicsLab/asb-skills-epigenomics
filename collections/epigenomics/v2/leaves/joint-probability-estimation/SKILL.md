---
name: joint-probability-estimation
description: Use when each candidate degenerate IUPAC motif carries a 2x2 contingency
  table (a,b,c,d counts in positive/negative datasets) and you need an occurrence-count-independent
  score such as MI for ranking or lattice simplification.
when_to_use_negative:
- You only need a statistical significance call rather than a rank/score — use DiNAMO's
  Fisher's exact test with Holm–Bonferroni adjustment instead, which is the paper's
  downstream filtering step
- The lattice has not been built yet and L-mer counts/contingency tables are unavailable
  — MI estimation cannot proceed without the per-motif contingency tables
- Motifs are not represented as degenerate IUPAC models with all instances present
  in the positive dataset (e.g. PWM/EM approaches like MEME), since the contingency-table-based
  MI is defined for this exact IUPAC lattice
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: DiNAMO
  role: Implements the exhaustive IUPAC lattice construction whose vertices carry
    the contingency tables from which joint probabilities and MI are computed; MI
    is used to simplify the lattice
  repo: https://github.com/bonsai-team/DiNAMO
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-018-2215-1
    title: 'DiNAMO: highly sensitive DNA motif discovery in high-throughput sequencing
      data'
  source_content_hash: sha256:a70efca271ab611e7e862142247630590a9fe2c8d5546019e456acacbe440fda
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bonsai-team/DiNAMO
  tools:
  - DiNAMO
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# joint-probability-estimation

## Summary

Estimate joint probabilities p(O_i, C_j) between binary motif presence/absence (O) and dataset condition (C: positive vs negative) from 2x2 contingency tables attached to each vertex of the IUPAC motif lattice, then compute mutual information MI(O;C) to score motifs independently of their occurrence count.

## When to use

Use when each candidate degenerate IUPAC motif carries a 2x2 contingency table (a,b,c,d counts in positive/negative datasets) and you need an occurrence-count-independent score such as MI for ranking or lattice simplification.

## When NOT to use

- You only need a statistical significance call rather than a rank/score — use DiNAMO's Fisher's exact test with Holm–Bonferroni adjustment instead, which is the paper's downstream filtering step
- The lattice has not been built yet and L-mer counts/contingency tables are unavailable — MI estimation cannot proceed without the per-motif contingency tables
- Motifs are not represented as degenerate IUPAC models with all instances present in the positive dataset (e.g. PWM/EM approaches like MEME), since the contingency-table-based MI is defined for this exact IUPAC lattice

## Inputs

- IUPAC motif lattice where each vertex carries a 2x2 contingency table (a, b, c, d counts over positive and negative FASTA datasets)
- Total number of sequences in the positive (signal.fa) and negative (control.fa) datasets

## Outputs

- MI scores for every lattice node (motif -> MI value)
- Motif list sorted by decreasing MI
- Retained non-dominated motifs written to mi_selected_motifs.tsv with their MI scores

## How to apply

Load the IUPAC motif lattice produced by the lattice-construction stage, where every vertex carries a 2x2 contingency table (a = sequences in the positive dataset containing the motif, b = sequences in the negative dataset containing the motif, c and d = remaining sequences in each dataset). For each node, estimate joint probabilities p(O_i, C_j) by normalizing contingency table entries over the total number of sequences in both datasets, then compute MI(O;C) = sum over i in {0,1}, j in {P,N} of p(O_i,C_j) * log2(p(O_i,C_j) / (p(O_i) * p(C_j))). The rationale, per DiNAMO, is that MI captures the dependency between a motif and a dataset independently from the motif's number of occurrences, unlike Fisher's exact test P-value, which biases toward frequent motifs. Sort all motifs by decreasing MI and use the scores in the lattice-simplification step: iteratively keep the highest-MI non-deleted motif and delete its descendants and ancestors, since they are dominated. Recompute MI from the contingency tables and check that retained motifs satisfy the dominance invariant (each kept motif has MI greater than each of its descendants and is not dominated by any ancestor) within numerical tolerance.

## Related tools

- **DiNAMO** (Implements the exhaustive IUPAC lattice construction whose vertices carry the contingency tables from which joint probabilities and MI are computed; MI is used to simplify the lattice) — https://github.com/bonsai-team/DiNAMO

## Examples

```
dinamo -pf signal.fa -nf control.fa -l 6
```

## Evaluation signals

- MI values recomputed from the contingency tables match the stored scores within numerical tolerance
- Dominance invariant holds: each kept motif has MI greater than each of its descendants and is not dominated by any ancestor
- Every motif in the lattice has been either selected or deleted when the iteration terminates
- Joint probabilities p(O_i, C_j) sum to 1 over the four contingency cells after normalization by total sequence count
- Output file mi_selected_motifs.tsv contains motifs paired with their MI scores

## Limitations

- MI alone does not provide significance: DiNAMO additionally computes Fisher's exact test P-values with Holm–Bonferroni adjustment for each selected motif
- The lattice (and hence the estimable probabilities) contains exactly the IUPAC motifs whose instances are all present in the positive dataset, so motifs with no positive-dataset instances cannot be scored
- DiNAMO is implemented in C++ (Sparsepp, boost) and targets desktop use (< 8 GB RAM); supplementary Algorithms 1–2 and Table S1 needed for exact implementation details are referenced but not retrievable from the provided context

## Evidence

- [methods] MI score from contingency table: "the MI (mutual information) captures the dependency between a motif and a dataset independently from its number of occurrences."
- [methods] contingency table per lattice vertex: "each vertex of the IUPAC lattice has a contingency table that contains the counts for the files"
- [methods] lattice simplification by MI: "we eliminate useless motifs of the lattice that could not improve previously selected motifs, based on their MI."
- [full_text] workflow step from source card: [evidence span withheld — anchor status: not_found]
- [readme] input format from README: "DiNAMO takes as input two fasta files, which contain respectively the positive dataset (signal.fa) and the negative dataset (control.fa)."
