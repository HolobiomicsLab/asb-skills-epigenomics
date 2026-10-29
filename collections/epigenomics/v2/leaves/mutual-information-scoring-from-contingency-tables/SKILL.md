---
name: mutual-information-scoring-from-contingency-tables
description: Use when ranking over-represented degenerate IUPAC motifs from a positive/control
  FASTA pair where occurrence-count-biased statistics would favor long or highly degenerate
  motifs; each candidate motif has a contingency table of counts (a, b, c, d).
when_to_use_negative:
- When a count-normalized enrichment statistic tied to occurrence frequency is explicitly
  required, e.g. comparing motif significance via Fisher's exact test P-value alone,
  since MI is deliberately independent of occurrence count
- When motifs cannot be represented in the IUPAC degenerate alphabet or when the analysis
  requires position-specific frequency matrices rather than discrete degenerate consensus
  motifs
- When only one dataset (no positive/control pair) is available, as MI here is defined
  over the contingency between motif presence and the positive-vs-negative condition
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0080
tools:
- name: DiNAMO
  role: Implements the exhaustive IUPAC lattice construction, computes contingency
    tables and MI per lattice node, ranks motifs by MI, and applies Fisher's exact
    test filtering; input via -pf signal.fa and -nf control.fa
  repo: https://github.com/bonsai-team/DiNAMO
- name: RSAT
  role: Cited as a prior exact-string motif discovery approach that DiNAMO's IUPAC/MI
    method is contrasted against
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-018-2215-1
    title: 'DiNAMO: highly sensitive DNA motif discovery in high-throughput sequencing
      data'
  source_content_hash: sha256:56cdfc5ad3c99864a89a84a881473a9dd22b220d11ff185fe3a9fc671b36141f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bonsai-team/DiNAMO
  tools:
  - DiNAMO
  - RSAT
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# mutual-information-scoring-from-contingency-tables

## Summary

Scores and ranks degenerate IUPAC motifs by mutual information (MI) computed from 2x2 contingency tables (motif presence/absence vs positive/negative dataset), so motif quality is captured independently of occurrence count — unlike Fisher's exact test P-value, which is biased by occurrence number.

## When to use

Use when ranking over-represented degenerate IUPAC motifs from a positive/control FASTA pair where occurrence-count-biased statistics would favor long or highly degenerate motifs; each candidate motif has a contingency table of counts (a, b, c, d).

## When NOT to use

- When a count-normalized enrichment statistic tied to occurrence frequency is explicitly required, e.g. comparing motif significance via Fisher's exact test P-value alone, since MI is deliberately independent of occurrence count
- When motifs cannot be represented in the IUPAC degenerate alphabet or when the analysis requires position-specific frequency matrices rather than discrete degenerate consensus motifs
- When only one dataset (no positive/control pair) is available, as MI here is defined over the contingency between motif presence and the positive-vs-negative condition

## Inputs

- IUPAC motif lattice with a 2x2 contingency table (a, b, c, d) at each vertex, from the lattice-construction stage
- Positive (signal) dataset in FASTA format, e.g. signal.fa
- Negative (control) dataset in FASTA format, e.g. control.fa
- Motif length L (parameter -l) and optional maximum number of degenerate letters (parameter -d)

## Outputs

- Ranked list of retained motifs with their MI scores (e.g. mi_selected_motifs.tsv)
- Per-motif contingency tables supporting the MI values
- Downstream: Fisher's exact test P-values with Holm–Bonferroni adjustment for each selected motif, reported in MEME motif format by DiNAMO

## How to apply

Build or load the IUPAC motif lattice where each vertex carries a 2x2 contingency table (a = positive-dataset sequences containing the motif, b = negative-dataset sequences, c and d = remaining sequences in each dataset). For every lattice node, normalize the table entries over the total number of sequences to estimate joint probabilities p(O_i, C_j), where O is binary motif presence/absence and C is the condition (positive vs negative), then compute MI(O;C) = sum over i in {0,1}, j in {P,N} of p(O_i,C_j) * log2(p(O_i,C_j) / (p(O_i) * p(C_j))). Sort all motifs by decreasing MI and iteratively select the highest-MI non-deleted motif, deleting all of its descendants and ancestors from the candidate list since those are dominated by it. Continue until every motif has been selected or deleted; the MI objective captures the dependency between a motif and a dataset independently from its number of occurrences, so it can be combined afterwards with a Fisher's exact test and Holm–Bonferroni adjustment (default p-value threshold 0.05 in DiNAMO) for final statistical filtering.

## Related tools

- **DiNAMO** (Implements the exhaustive IUPAC lattice construction, computes contingency tables and MI per lattice node, ranks motifs by MI, and applies Fisher's exact test filtering; input via -pf signal.fa and -nf control.fa) — https://github.com/bonsai-team/DiNAMO
- **RSAT** (Cited as a prior exact-string motif discovery approach that DiNAMO's IUPAC/MI method is contrasted against)

## Examples

```
dinamo -pf signal.fa -nf control.fa -l 6 -d 3 -o mi_selected_motifs.tsv
```

## Evaluation signals

- Dominance invariant: each kept motif has MI greater than each of its descendants and is not dominated by any ancestor in the lattice
- MI values recomputed independently from the stored contingency tables match the reported scores within numerical tolerance
- Every lattice motif appears exactly once as either selected or deleted, with deletion traceable to a selected ancestor/descendant
- Selected motifs' MI ranking is independent of occurrence count — a motif with far fewer occurrences can outrank a more frequent one if its contingency association is stronger
- Post-filtering, retained motifs pass Fisher's exact test at threshold 0.05 after Holm–Bonferroni adjustment

## Limitations

- MI alone does not assign statistical significance; DiNAMO still computes Fisher's exact test P-values with Holm–Bonferroni correction on the MI-selected motifs
- The lattice contains exactly the IUPAC motifs whose instances are all present in the positive dataset, so motifs absent from the positive data are never scored
- Selection by iterative ancestor/descendant deletion assumes the lattice dominance relation holds; ties or near-equal MI among related motifs can make the retained set sensitive to traversal order

## Evidence

- [methods] MI objective is independent of occurrence count: "the MI (mutual information) captures the dependency between a motif and a dataset independently from its number of occurrences."
- [methods] Contingency table per lattice vertex: "each vertex of the IUPAC lattice has a contingency table that contains the counts for the files"
- [methods] Lattice simplification by MI: "we eliminate useless motifs of the lattice that could not improve previously selected motifs, based on their MI."
- [intro] MI used as the objective function: "It uses mutual information (MI) as an objective function to search for over-represented degenerate motifs"
- [readme] DiNAMO input modes and CLI: "dinamo -pf signal.fa  -nf control.fa -l 6"
- [methods] Fisher's exact test and Holm–Bonferroni follow-up: [evidence span withheld — anchor status: not_found]
