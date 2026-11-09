---
name: constrained-genomic-interval-shuffling
description: Use when testing whether a contiguous set of genomic intervals (e.g.,
  a TLD-clique of >4 TLDs) shows enriched inter-interval contacts, by building position-randomized
  null cliques on the same chromosome.
when_to_use_negative:
- Intervals span multiple chromosomes, since the permutation constrains the clique
  start position to the same chromosome
- The analysis question concerns TLD boundaries themselves rather than inter-TLD contact
  enrichment within a clique
- A clique contains 4 or fewer TLDs, since the pipeline retains only cliques with
  more than 4 TLDs
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: deDoc2
  role: Provides the predefined single-cell TLD predictions used as input to the TLD-clique
    identification pipeline adopted from Paulsen et al. (with TAD substituted by TLD)
  repo: https://github.com/zengguangjie/deDoc2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/advs.202300366
    title: DeDoc2 Identifies and Characterizes the Hierarchy and Dynamics of Chromatin
      TAD‐Like Domains in the Single Cells
  source_content_hash: sha256:231c3378768aa58dc1eeed66ad302b56a7c495f5ff6bd4cd7be459280cfca25e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zengguangjie/deDoc2
  tools:
  - deDoc2
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# constrained-genomic-interval-shuffling

## Summary

Generates permuted control sets of genomic interval groups (e.g., TLD-cliques) by randomly relocating the group start position on the same chromosome while preserving interval sizes and inter-interval genomic distances, enabling observed-vs-expected enrichment tests over 250 permutations.

## When to use

Use when testing whether a contiguous set of genomic intervals (e.g., a TLD-clique of >4 TLDs) shows enriched inter-interval contacts, by building position-randomized null cliques on the same chromosome.

## When NOT to use

- Intervals span multiple chromosomes, since the permutation constrains the clique start position to the same chromosome
- The analysis question concerns TLD boundaries themselves rather than inter-TLD contact enrichment within a clique
- A clique contains 4 or fewer TLDs, since the pipeline retains only cliques with more than 4 TLDs

## Inputs

- Per-cell Hi-C contact matrix (intra-chromosome)
- Predefined single-cell TLD predictions (e.g., deDoc2 .TAD output partitions)
- Identified TLD-cliques (each containing more than 4 TLDs)

## Outputs

- Per-clique enrichment table (tld_clique_enrichment.csv) with observed contacts, mean of 250 permuted-clique contacts, and observed/expected enrichment ratio

## How to apply

For each candidate clique identified from predefined single-cell TLDs (retaining only cliques containing more than 4 TLDs), extract the observed inter-TLD contacts from the per-cell Hi-C contact matrix. Then construct 250 permuted control cliques per clique by randomly placing the clique start position on the same chromosome, while preserving the TLD sizes within the clique and the genomic distances between consecutive TLDs. Compute the expected inter-TLD contacts as the mean across the 250 permuted cliques, and calculate enrichment as the ratio of observed to expected contacts. Rationale: constraining the shuffle to preserve interval sizes and spacing isolates the effect of the specific domain arrangement from generic distance-dependent contact decay.

## Related tools

- **deDoc2** (Provides the predefined single-cell TLD predictions used as input to the TLD-clique identification pipeline adopted from Paulsen et al. (with TAD substituted by TLD)) — https://github.com/zengguangjie/deDoc2

## Evaluation signals

- For every clique of >4 TLDs, reported enrichment equals observed inter-TLD contacts divided by the mean of the 250 permuted-clique contacts
- All analyzed cliques have finite positive enrichment values
- Each clique has exactly 250 permuted control cliques with preserved TLD sizes and inter-TLD genomic distances
- Permuted clique start positions lie on the same chromosome as the original clique
- Per-clique results are written to tld_clique_enrichment.csv

## Limitations

- The null model preserves inter-TLD genomic distances, so distance-dependent contact decay is controlled for but other covariates (e.g., compartment identity) are not explicitly matched
- Validation requires a sufficient number of TLDs in the cell; cells with insufficient TLDs (e.g., '2n DNA' stage) must be excluded from analysis

## Evidence

- [other] construct 250 permuted control cliques by randomly placing the clique start position on the same chromosome while preserving the TLD sizes in the clique and the genomic distances between consecutive TLDs: [evidence span withheld — anchor status: not_found]
- [other] Calculate the enrichment of inter-TLD contacts as the ratio of observed to expected inter-TLD contacts and write per-clique results to tld_clique_enrichment.csv: [evidence span withheld — anchor status: not_found]
- [other] well-defined cliques should show a clean TLD pattern with few inter-TLD interactions despite enriched inter-TLD contacts within the clique: [evidence span withheld — anchor status: not_found]
- [readme] The output files contains partitions as TLD results. Each line represents a TLD, containg the node id of this TLD, space split, numbering from 1.: "The output files contains partitions as TLD results. Each line represents a TLD, containg the node id of this TLD, space split"
