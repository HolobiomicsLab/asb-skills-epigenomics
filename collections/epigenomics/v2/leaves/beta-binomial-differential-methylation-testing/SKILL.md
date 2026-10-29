---
name: beta-binomial-differential-methylation-testing
description: Use when you have a clustered-region methylation counts table (combined.csv
  with chr, start, end, CpG, per-sample -N and -X columns) and need to identify DMRs
  between two or more sample groups from MethylC-seq data.
when_to_use_negative:
- The goal is to model sets of regions or quantitative traits — DMRfinder 'identifies
  individual regions that are differentially methylated but does not consider sets
  of regions or quantitative traits'.
- The data contain batch effects that must be accounted for — DMRfinder 'cannot account
  for batch effects'.
- The question concerns differential methylation at individual CpG sites rather than
  regions of multiple CpG sites; the paper argues a more biologically appropriate
  approach is to test regions of multiple CpG sites.
edam_operation: http://edamontology.org/operation_3206
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
tools:
- name: findDMRs.r
  role: DMRfinder script that conducts pairwise tests of sample groups to find genomic
    regions that are differentially methylated, fitting the DSS beta-binomial model
    and Wald tests
  repo: https://github.com/jsh58/DMRfinder
- name: DSS
  role: Bioconductor package (v2.14.0) providing the Bayesian beta-binomial hierarchical
    model and Wald tests used to test regions for differential methylation
  repo: http://www.bioconductor.org/packages/release/bioc/html/DSS.html
- name: R
  role: Runtime environment; findDMRs.r is invoked via Rscript (tested with Rscript
    version 3.3.1)
  repo: https://www.r-project.org/
- name: DMRfinder
  role: Pipeline package whose final statistical step this skill reconstructs; provides
    the filtering defaults and output schema
  repo: https://github.com/jsh58/DMRfinder
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-017-1909-0
    title: 'DMRfinder: efficiently identifying differentially methylated regions from
      MethylC-seq data'
  source_content_hash: sha256:722da46ae7c7586dbcdbea8891a76adb2eff3b2ca77ed174af13937dc6fd4b75
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jsh58/DMRfinder
  tools:
  - DMRfinder
  - DSS
  - R
  - findDMRs.r
  edam_operation: http://edamontology.org/operation_3206
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# beta-binomial-differential-methylation-testing

## Summary

Tests clustered genomic regions for differential methylation between sample groups using DSS's beta-binomial hierarchical modeling and Wald tests, as implemented in DMRfinder's findDMRs.r. Because methylation counts are conglomerated within each cluster, the number of hypothesis tests is reduced and statistical power increases compared with testing individual CpG sites.

## When to use

Use when you have a clustered-region methylation counts table (combined.csv with chr, start, end, CpG, per-sample -N and -X columns) and need to identify DMRs between two or more sample groups from MethylC-seq data.

## When NOT to use

- The goal is to model sets of regions or quantitative traits — DMRfinder 'identifies individual regions that are differentially methylated but does not consider sets of regions or quantitative traits'.
- The data contain batch effects that must be accounted for — DMRfinder 'cannot account for batch effects'.
- The question concerns differential methylation at individual CpG sites rather than regions of multiple CpG sites; the paper argues a more biologically appropriate approach is to test regions of multiple CpG sites.

## Inputs

- combined.csv: clustered-region methylation counts table from combine_CpG_sites.py with chr, start, end, CpG columns and per-sample total counts (-N) and methylated counts (-X) columns
- Sample-to-group assignment (e.g., C1,C2 for Control and E1,E2 for Exptl) and group names (e.g., -n Control,Exptl)

## Outputs

- results.csv: filtered differentially methylated region table with chr, start, end, CpG and DSS columns mu (group methylation fraction), diff (group2 relative to group1), and pval for each group-group comparison

## How to apply

Load the clustered-region methylation counts file produced by combine_CpG_sites.py (columns chr, start, end, CpG plus per-sample total counts (-N) and methylated counts (-X)) and invoke 'Rscript findDMRs.r -i combined.csv -o results.csv C1,C2 E1,E2 -n Ctrl,Exp', which fits the Bayesian beta-binomial hierarchical model and Wald tests via the Bioconductor package DSS (v2.14.0). Apply the default region filters: minimum 3 CpGs per region (-c 3), minimum methylation difference of 0.10 between groups (-d 0.10), maximum p-value 0.05 (-p 0.05), and maximum q-value 1 (-q 1). Report the DSS columns mu (group methylation fraction), diff (group2 relative to group1), and pval for each group-group comparison, plus chr, start, end, CpG. The rationale for testing whole regions rather than individual CpG sites is that clustering reduces the number of hypothesis tests and increases statistical power.

## Related tools

- **findDMRs.r** (DMRfinder script that conducts pairwise tests of sample groups to find genomic regions that are differentially methylated, fitting the DSS beta-binomial model and Wald tests) — https://github.com/jsh58/DMRfinder
- **DSS** (Bioconductor package (v2.14.0) providing the Bayesian beta-binomial hierarchical model and Wald tests used to test regions for differential methylation) — http://www.bioconductor.org/packages/release/bioc/html/DSS.html
- **R** (Runtime environment; findDMRs.r is invoked via Rscript (tested with Rscript version 3.3.1)) — https://www.r-project.org/
- **DMRfinder** (Pipeline package whose final statistical step this skill reconstructs; provides the filtering defaults and output schema) — https://github.com/jsh58/DMRfinder

## Examples

```
Rscript findDMRs.r -i combined.csv -o results.csv -v -n Control,Exptl C1,C2 E1,E2
```

## Evaluation signals

- A region is reported only if it meets the minimum methylation difference of 0.10 and a p-value at or below 0.05.
- The produced results.csv matches the reference DMRfinder output table row-by-row (region coordinates, mu, diff, pval values) on the identical clustered-region input.
- Every reported region contains at least 3 CpGs (-c 3 default).
- Output columns include chr, start, end, CpG, and for each comparison mu, diff, and pval as returned by DSS.
- Diff values are interpretable as group2 methylation relative to group1 and mu values lie in [0-1].

## Limitations

- DMRfinder does not allow modeling of sets of regions or quantitative traits; it identifies individual differentially methylated regions only.
- DMRfinder cannot account for batch effects.
- Because methylation counts are conglomerated within each cluster, differences at individual CpG sites within a region are not tested separately.

## Evidence

- [methods] Pairwise testing via findDMRs.r fits DSS model on clustered regions: "The DMRfinder script `findDMRs.r` conducts pairwise tests of sample groups to find genomic regions that are differentially methylated"
- [abstract] Statistical framework of DSS with beta-binomial modeling and Wald tests: "It then compares methylation levels using beta-binomial hierarchical modeling and Wald tests."
- [results] Region-level testing reduces number of tests and increases power: [evidence span withheld — anchor status: not_found]
- [intro] Region-level testing is more biologically appropriate: "Therefore, a more biologically appropriate approach is to determine differentially methylated regions (DMRs) of multiple CpG sites."
- [readme] Quick start step 4 invocation of findDMRs.r: "Rscript findDMRs.r  -i combined.csv  -o results.csv  -v  -n Control,Exptl  C1,C2  E1,E2"
- [methods] findDMRs.r filter defaults: -c 3, -d 0.10, -p 0.05, -q 1: "-c <int>    Min. number of CpGs in a region (def. 3)"
- [readme] README documents DSS as the statistical framework: "It then performs pairwise comparisons to identify differentially methylated regions (DMRs) using the statistical framework of the Bioconductor package DSS"
- [readme] DMRfinder tested with DSS version 2.14.0: "DMRfinder scripts have been tested with Rscript version 3.3.1 and DSS version 2.14.0."
