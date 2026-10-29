---
name: tld-clique-identification
description: Use when you have single-cell Hi-C contact matrices plus per-cell TLD
  predictions (e.g. from deDoc2.w/deDoc2.s) and want to detect higher-order multi-TLD
  structures and test whether inter-TLD contacts within candidate cliques exceed genomic-distance-expectation.
when_to_use_negative:
- The goal is de novo TLD (domain boundary) prediction from raw single-cell Hi-C —
  deDoc2.w/deDoc2.s should be run first; this skill consumes existing TLD predictions,
  not raw contact matrices alone.
- Bulk (population-average) Hi-C data is the target where TADs are the unit of analysis
  — the pipeline is designed for per-cell TLDs and per-cell contact matrices.
- Cliques with 4 or fewer TLDs are the only candidates available — the pipeline explicitly
  retains only cliques containing more than 4 TLDs.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3940
tools:
- name: deDoc2
  role: Produces the predefined single-cell TLD partitions (via deDoc2.w/deDoc2.s
    structural-entropy minimization) that serve as input to the clique identification
    and permutation-testing pipeline.
  repo: https://github.com/zengguangjie/deDoc2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/advs.202300366
    title: DeDoc2 Identifies and Characterizes the Hierarchy and Dynamics of Chromatin
      TAD‐Like Domains in the Single Cells
  source_content_hash: sha256:9fd6700f12e5ae5935c9e50b167241d4ec8b153ab9da7557da44781209fad7b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zengguangjie/deDoc2
  tools:
  - deDoc2
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3940
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# tld-clique-identification

## Summary

Identifies single-cell TLD-cliques — higher-level chromatin structures beyond nested TLDs — by adapting the Paulsen et al. TAD-clique pipeline to predefined single-cell TLDs, and quantifies whether cliques are genuine higher-order structures by measuring inter-TLD contact enrichment as observed/expected over 250 position-permuted control cliques.

## When to use

Use when you have single-cell Hi-C contact matrices plus per-cell TLD predictions (e.g. from deDoc2.w/deDoc2.s) and want to detect higher-order multi-TLD structures and test whether inter-TLD contacts within candidate cliques exceed genomic-distance-expectation.

## When NOT to use

- The goal is de novo TLD (domain boundary) prediction from raw single-cell Hi-C — deDoc2.w/deDoc2.s should be run first; this skill consumes existing TLD predictions, not raw contact matrices alone.
- Bulk (population-average) Hi-C data is the target where TADs are the unit of analysis — the pipeline is designed for per-cell TLDs and per-cell contact matrices.
- Cliques with 4 or fewer TLDs are the only candidates available — the pipeline explicitly retains only cliques containing more than 4 TLDs.

## Inputs

- Per-cell single-cell Hi-C contact matrix (intra-chromosome, deDoc2 contact-matrix or sparse format)
- Predefined single-cell TLD predictions (deDoc2.w / deDoc2.s .TAD partition files, one TLD per line, bin IDs from 1)
- Chromosome bin size (e.g. deDoc2 default 40 kb) used to map clique positions to contact-matrix indices

## Outputs

- tld_clique_enrichment.csv with per-clique rows: clique coordinates/TLD membership, observed inter-TLD contacts, mean expected contacts over 250 permuted cliques, and O/E enrichment ratio
- Set of identified TLD-cliques (each containing >4 TLDs)

## How to apply

Run the TLD clique identification pipeline adopted from Paulsen et al., substituting TAD by TLD throughout, using predefined single-cell TLDs as input and retaining only cliques containing more than 4 TLDs. For each identified clique, extract observed inter-TLD contacts from the per-cell Hi-C contact matrix. Build 250 permuted control cliques per clique by randomly placing the clique start position on the same chromosome while preserving the clique's TLD sizes and the genomic distances between consecutive TLDs; compute expected contacts as the mean over the 250 permutations. Report enrichment as the ratio observed/expected per clique, writing results to tld_clique_enrichment.csv. A well-defined clique should show a clean TLD pattern with few inter-TLD interactions outside the clique but enriched inter-TLD contacts within it.

## Related tools

- **deDoc2** (Produces the predefined single-cell TLD partitions (via deDoc2.w/deDoc2.s structural-entropy minimization) that serve as input to the clique identification and permutation-testing pipeline.) — https://github.com/zengguangjie/deDoc2

## Examples

```
java -jar deDoc2.jar -inputfile ./example/14.clean.con.chr18.matrix -binsize 40
```

## Evaluation signals

- For every clique of >4 TLDs, the reported enrichment equals observed inter-TLD contacts divided by the mean of the 250 permuted-clique contacts.
- All analyzed cliques have finite, positive enrichment values (no divide-by-zero or negative enrichment).
- Each clique contains strictly more than 4 TLDs, and permuted controls preserve both the clique's TLD sizes and the genomic distances between consecutive TLDs, with all 250 controls placed on the same chromosome.
- Output file tld_clique_enrichment.csv exists with one row per identified clique and columns for observed, expected, and O/E enrichment.
- Well-defined cliques should show enriched inter-TLD contacts within the clique despite few inter-TLD interactions elsewhere in the per-cell contact matrix.

## Limitations

- The permutation scheme fixes clique geometry (TLD sizes and inter-TLD distances), so enrichment reflects contacts beyond distance expectation only; cliques whose enrichment is not positive may not represent genuine higher-order structure.
- Cells with too few identifiable TLDs (e.g. '2n DNA' stage cells in the paper) cannot support clique analysis and should be excluded.
- deDoc2 treats single bins appearing alone in one line of .TAD output as non-TAD regions and these should be removed before defining clique constituents.

## Evidence

- [other] The paper describes a pipeline that identifies TLD-cliques at the single-cell level using predefined single-cell TLDs as input: [evidence span withheld — anchor status: not_found]
- [other] For each identified clique, construct 250 permuted control cliques: [evidence span withheld — anchor status: not_found]
- [other] retaining only cliques containing more than 4 TLDs: [evidence span withheld — anchor status: not_found]
- [readme] The output files contains partitions as TLD results: "The output files contains partitions as TLD results. Each line represents a TLD, containg the node id of this TLD, space split, numbering from 1."
- [discussion] deDoc2 decodes TLD from single‐cell Hi‐C data with no need of data imputation: "deDoc2 decodes TLD from single‐cell Hi‐C data with no need of data imputation, while most state‐of‐the‐art TLD predictors do"
