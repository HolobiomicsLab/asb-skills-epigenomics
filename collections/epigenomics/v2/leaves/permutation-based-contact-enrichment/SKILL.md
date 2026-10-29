---
name: permutation-based-contact-enrichment
description: Use when validating that single-cell chromatin structures (e.g., TLD-cliques
  from deDoc2 TLD predictions) show genuine inter-TLD contact enrichment beyond what
  is expected from genomic layout alone.
when_to_use_negative:
- Input TLD structures contain 4 or fewer TLDs — the pipeline retains only cliques
  with more than 4 TLDs
- The analysis goal is de novo TLD detection from raw Hi-C data rather than enrichment
  testing of already-predicted structures
- Per-cell contact matrices are unavailable or lack sufficient contacts to support
  a chromosome-matched permutation control
tools:
- name: deDoc2
  role: Provides the predefined single-cell TLD predictions (via deDoc2.w and deDoc2.s
    structural-entropy minimization) that serve as input for clique identification
    and the contact-enrichment permutation test
  repo: https://github.com/zengguangjie/deDoc2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/advs.202300366
    title: DeDoc2 Identifies and Characterizes the Hierarchy and Dynamics of Chromatin
      TAD‐Like Domains in the Single Cells
  source_content_hash: sha256:2026d14370920b13395d628d69f9a0621e4cdea813086d430513257cc2a8a56b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zengguangjie/deDoc2
  tools:
  - deDoc2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# permutation-based-contact-enrichment

## Summary

Quantifies whether predicted single-cell TLD-cliques have enriched inter-TLD Hi-C contacts by comparing observed per-cell inter-TLD contacts to expected contacts from 250 permuted control cliques, computing an observed/expected ratio per clique.

## When to use

Use when validating that single-cell chromatin structures (e.g., TLD-cliques from deDoc2 TLD predictions) show genuine inter-TLD contact enrichment beyond what is expected from genomic layout alone.

## When NOT to use

- Input TLD structures contain 4 or fewer TLDs — the pipeline retains only cliques with more than 4 TLDs
- The analysis goal is de novo TLD detection from raw Hi-C data rather than enrichment testing of already-predicted structures
- Per-cell contact matrices are unavailable or lack sufficient contacts to support a chromosome-matched permutation control

## Inputs

- Predefined single-cell TLD predictions (e.g., from deDoc2, output .TAD / .window.TAD partitions)
- Per-cell Hi-C contact matrix (n x n intra-chromosome contact matrix or sparse format, with a known binsize such as 40 kb)
- Chromosome start positions and TLD boundary coordinates on each chromosome

## Outputs

- tld_clique_enrichment.csv with per-clique rows: observed inter-TLD contacts, expected contacts (mean over 250 permuted cliques), and observed/expected enrichment ratio

## How to apply

Run the TLD clique identification pipeline adopted from Paulsen et al. with TAD substituted by TLD, retaining only cliques containing more than 4 TLDs. For each identified clique, extract the observed inter-TLD contacts directly from the per-cell Hi-C contact matrix. Build 250 permuted control cliques per clique by randomly placing the clique start position on the same chromosome while preserving the TLD sizes in the clique and the genomic distances between consecutive TLDs. Compute expected contacts as the mean across the 250 permuted cliques, then calculate enrichment as observed/expected and write per-clique results to tld_clique_enrichment.csv. The rationale is that a well-defined clique should show a clean TLD pattern with few inter-TLD interactions outside the clique, yet enriched inter-TLD contacts within it.

## Related tools

- **deDoc2** (Provides the predefined single-cell TLD predictions (via deDoc2.w and deDoc2.s structural-entropy minimization) that serve as input for clique identification and the contact-enrichment permutation test) — https://github.com/zengguangjie/deDoc2

## Examples

```
java -jar deDoc2.jar -inputfile ./example/14.clean.con.chr18.matrix -binsize 40
```

## Evaluation signals

- For every clique of >4 TLDs, confirm the reported enrichment equals observed inter-TLD contacts divided by the mean of the 250 permuted-clique contacts
- All analyzed cliques must have finite, positive enrichment values
- Verify each permuted control clique preserves the clique's TLD sizes and the genomic distances between consecutive TLDs, and lies on the same chromosome
- Check tld_clique_enrichment.csv contains exactly one row per identified clique with the expected schema (observed, expected, enrichment)
- Enriched cliques should simultaneously show a clean TLD pattern with few inter-TLD interactions outside the clique

## Limitations

- Only cliques with more than 4 TLDs are analyzed, so smaller structures are silently excluded
- Expected contacts are estimated from 250 position-shuffled cliques, so the null model preserves genomic-distance structure but not other regional features (e.g., local contact density heterogeneity)
- deDoc2 predictions depend on binsize; downstream clique enrichment inherits any sensitivity to binsize choice

## Evidence

- [other] For each clique, construct 250 permuted control cliques by randomly placing the clique start position on the same chromosome while preserving the TLD sizes: [evidence span withheld — anchor status: not_found]
- [other] Enrichment as observed/expected ratio written to tld_clique_enrichment.csv: [evidence span withheld — anchor status: not_found]
- [other] Well-defined clique signature: clean TLD pattern with few inter-TLD interactions despite enriched inter-TLD contacts: [evidence span withheld — anchor status: not_found]
- [readme] deDoc2 TLD predictors supply the clique input structures: "The deDoc2 package consists of two predictors, deDoc2.w and deDoc2.s, to predict higher level, larger and lower level, smaller TLDs, respectively."
- [methods] TLD prediction underlying the analysis uses structural entropy minimization: "deDoc2 employed a dynamic programming algorithm to minimize 2D structural entropy."
