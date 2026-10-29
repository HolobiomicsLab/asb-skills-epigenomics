---
name: dominance-based-graph-pruning
description: Use when a candidate motif set forms a partial-order lattice (e.g. IUPAC
  generalizations of observed L-mers) and redundant/generalized motifs must be eliminated
  using an objective function computed from 2x2 contingency tables over positive/negative
  datasets.
when_to_use_negative:
- When motifs must be ranked by statistical significance rather than information content
  — DiNAMO computes Fisher's exact test P-values with Holm–Bonferroni adjustment as
  a separate, later filtering step
- When the candidate set is an unordered list without lattice (generalization/specialization)
  relationships, since ancestor/descendant deletion assumes the partial order
- When searching for motifs whose instances are not all present in the positive dataset
  — the DiNAMO lattice only contains motifs whose every instance occurs in the positive
  set
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: DiNAMO
  role: Exhaustive IUPAC motif discovery tool whose lattice construction supplies
    the contingency-table vertices and whose MI-based simplification performs the
    dominance pruning
  repo: https://github.com/bonsai-team/DiNAMO
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-018-2215-1
    title: 'DiNAMO: highly sensitive DNA motif discovery in high-throughput sequencing
      data'
  source_content_hash: sha256:aec58164043f3b8986d8098288878fa013b4e6fc23c129e6605aba9e2afe72a5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bonsai-team/DiNAMO
  tools:
  - DiNAMO
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# dominance-based-graph-pruning

## Summary

Simplifies the DiNAMO IUPAC motif lattice by ranking motifs on mutual information (MI) and pruning dominated ancestors/descendants, retaining only non-dominated over-represented motifs independent of occurrence count.

## When to use

Use when a candidate motif set forms a partial-order lattice (e.g. IUPAC generalizations of observed L-mers) and redundant/generalized motifs must be eliminated using an objective function computed from 2x2 contingency tables over positive/negative datasets.

## When NOT to use

- When motifs must be ranked by statistical significance rather than information content — DiNAMO computes Fisher's exact test P-values with Holm–Bonferroni adjustment as a separate, later filtering step
- When the candidate set is an unordered list without lattice (generalization/specialization) relationships, since ancestor/descendant deletion assumes the partial order
- When searching for motifs whose instances are not all present in the positive dataset — the DiNAMO lattice only contains motifs whose every instance occurs in the positive set

## Inputs

- IUPAC motif lattice from DiNAMO's lattice-construction stage, each vertex annotated with a 2x2 contingency table over the positive dataset (signal FASTA) and negative dataset (control FASTA)
- Positive and negative FASTA files with motif length L used to derive the lattice

## Outputs

- Non-dominated motif list with MI scores (e.g. mi_selected_motifs.tsv), sorted by decreasing MI
- Dominance relationships (deleted ancestors/descendants) implicit in the pruning order

## How to apply

Build (or load) the IUPAC motif lattice whose vertices each carry a 2x2 contingency table: a = positive-dataset sequences containing the motif, b = negative-dataset sequences containing it, c and d = remaining sequences. For each node, normalize the table to joint probabilities p(O_i, C_j) over the total sequence count and compute MI(O;C) = sum over i in {0,1}, j in {P,N} of p(O_i,C_j) * log2(p(O_i,C_j) / (p(O_i)*p(C_j))), where O is motif presence/absence and C is positive-vs-negative condition. Sort all motifs by decreasing MI, then iteratively take the highest-MI non-deleted motif into the result list and delete all of its descendants and ancestors, since those are dominated. Continue until every motif is selected or deleted, and write retained motifs with MI scores (e.g. to mi_selected_motifs.tsv). Rationale: MI captures the dependency between a motif and a dataset independently from its number of occurrences, unlike Fisher's exact test P-value.

## Related tools

- **DiNAMO** (Exhaustive IUPAC motif discovery tool whose lattice construction supplies the contingency-table vertices and whose MI-based simplification performs the dominance pruning) — https://github.com/bonsai-team/DiNAMO

## Examples

```
dinamo -pf signal.fa -nf control.fa -l 6 -o mi_selected_motifs.tsv
```

## Evaluation signals

- Dominance invariant: every kept motif has MI strictly greater than each of its descendants and is not dominated by any ancestor
- MI values recomputed from the stored 2x2 contingency tables match the stored scores within numerical tolerance
- Every lattice motif appears in exactly one of the two states at termination: selected into the result list or deleted as dominated
- Result is ordered by decreasing MI and each retained motif's instances are all present in the positive dataset
- Downstream Fisher's exact test with Holm–Bonferroni adjustment (default p-value threshold 0.05) can be applied to the retained motifs without contradiction

## Limitations

- MI comparison eliminates motifs that 'could not improve previously selected motifs' — genuinely useful but lower-MI relatives of a top motif are pruned along with dominated ones
- MI ranking is occurrence-count-independent by design, so frequency-based prioritization must come from later Fisher's exact test filtering
- Memory/runtime constraints apply to the exhaustive lattice: DiNAMO is designed for desktop machines with less than 8 GB of RAM

## Evidence

- [methods] MI ranking and ancestor/descendant deletion: "the MI (mutual information) captures the dependency between a motif and a dataset independently from its number of occurrences."
- [methods] Lattice vertices carry contingency tables: "each vertex of the IUPAC lattice has a contingency table that contains the counts for the files"
- [methods] Simplification step is the pruning operation: "we eliminate useless motifs of the lattice that could not improve previously selected motifs, based on their MI."
- [methods] Lattice scope restricted to positive dataset: "At the end of the process, the lattice contains exactly the set of all IUPAC motifs for which all instances are present in"
- [readme] Concrete CLI invocation for the upstream tool: "dinamo -pf signal.fa  -nf control.fa -l 6"
