---
name: dss-wald-test-workflow
description: Use when MethylC-seq methylation counts have already been clustered into
  genomic regions (combined.csv) and you need to test regions for differential methylation
  between two or more sample groups.
when_to_use_negative:
- Input is per-CpG-site methylation counts rather than clustered-region counts — cluster
  the CpGs first with combine_CpG_sites.py
- You need to model sets of regions or quantitative traits, or account for batch effects
  — DMRfinder explicitly does not support these
- The comparison goal is finding per-site differential methylation rather than DMRs
  over multiple CpG sites
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
tools:
- name: DMRfinder
  role: Pipeline package whose findDMRs.r script conducts pairwise group tests to
    find differentially methylated regions
  repo: https://github.com/jsh58/DMRfinder
- name: DSS
  role: Bioconductor package (v2.14.0) providing the beta-binomial hierarchical model
    and Wald tests used to test regions
  repo: http://www.bioconductor.org/packages/release/bioc/html/DSS.html
- name: R
  role: Runtime for findDMRs.r (tested with Rscript version 3.3.1)
- name: combine_CpG_sites.py
  role: Upstream DMRfinder script producing the clustered-region counts file (combined.csv)
    that serves as input to findDMRs.r
  repo: https://github.com/jsh58/DMRfinder
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-017-1909-0
    title: 'DMRfinder: efficiently identifying differentially methylated regions from
      MethylC-seq data'
  source_content_hash: sha256:8094019f2bf097ba6629c65cdd6f3c107339c3e187f25228e47913cb611e0c7c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jsh58/DMRfinder
  tools:
  - DMRfinder
  - DSS
  - R
  - combine_CpG_sites.py
  - findDMRs.r
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# dss-wald-test-workflow

## Summary

Tests clustered genomic regions for differential methylation between sample groups using the DSS beta-binomial hierarchical model and Wald tests, as implemented in DMRfinder's findDMRs.r, which tests whole regions (methylation counts conglomerated within clusters) rather than individual CpG sites to reduce hypothesis count and increase power.

## When to use

Use when MethylC-seq methylation counts have already been clustered into genomic regions (combined.csv) and you need to test regions for differential methylation between two or more sample groups.

## When NOT to use

- Input is per-CpG-site methylation counts rather than clustered-region counts — cluster the CpGs first with combine_CpG_sites.py
- You need to model sets of regions or quantitative traits, or account for batch effects — DMRfinder explicitly does not support these
- The comparison goal is finding per-site differential methylation rather than DMRs over multiple CpG sites

## Inputs

- combined.csv: clustered-region methylation counts with chr, start, end, CpG columns plus per-sample total counts (-N) and methylated counts (-X) columns from combine_CpG_sites.py
- Group definitions mapping sample count columns to named sample groups (e.g. Control: C1,C2; Exptl: E1,E2)

## Outputs

- results.csv: filtered DMR table with chr, start, end, CpG, and DSS columns mu (group methylation fraction), diff (group2 relative to group1), and pval for each group-group comparison

## How to apply

Load the clustered-region methylation counts file (combined.csv, produced by combine_CpG_sites.py) containing chr, start, end, CpG columns plus per-sample total counts (-N) and methylated counts (-X) columns. Invoke findDMRs.r via Rscript with paired sample columns per group (e.g. C1,C2 and E1,E2), relying on the Bioconductor package DSS (v2.14.0) to fit the Bayesian beta-binomial hierarchical model and run Wald tests comparing group methylation levels. Apply the default region filters: minimum 3 CpGs per region (-c 3), minimum methylation difference 0.10 (-d 0.10), maximum p-value 0.05 (-p 0.05), and maximum q-value 1 (-q 1). Report the DSS columns mu (group methylation fraction), diff (group2 relative to group1), and pval for each group-group comparison alongside chr, start, end, and CpG. Because counts are summed within each cluster before testing, the number of hypothesis tests is reduced and statistical power increases relative to per-CpG testing.

## Related tools

- **DMRfinder** (Pipeline package whose findDMRs.r script conducts pairwise group tests to find differentially methylated regions) — https://github.com/jsh58/DMRfinder
- **DSS** (Bioconductor package (v2.14.0) providing the beta-binomial hierarchical model and Wald tests used to test regions) — http://www.bioconductor.org/packages/release/bioc/html/DSS.html
- **R** (Runtime for findDMRs.r (tested with Rscript version 3.3.1))
- **combine_CpG_sites.py** (Upstream DMRfinder script producing the clustered-region counts file (combined.csv) that serves as input to findDMRs.r) — https://github.com/jsh58/DMRfinder

## Examples

```
Rscript findDMRs.r -i combined.csv -o results.csv -v -n Control,Exptl C1,C2 E1,E2
```

## Evaluation signals

- Every reported region has at least 3 CpGs (CpG column >= 3)
- Every reported region has an absolute methylation difference (diff) of at least 0.10 between groups
- Every reported region has pval <= 0.05
- Output results.csv matches the reference DMRfinder findDMRs.r output row-by-row (region coordinates, mu, diff, pval) on the same clustered-region input
- Output columns include chr, start, end, CpG, and the DSS mu/diff/pval columns for each group-group comparison

## Limitations

- DMRfinder identifies individual differentially methylated regions but does not consider sets of regions or quantitative traits
- DMRfinder cannot account for batch effects
- Because methylation counts are conglomerated within each cluster, results depend on the upstream clustering (max 100 bp spacing, max 500 bp region length by default)

## Evidence

- [abstract] Comparison of methylation levels using beta-binomial hierarchical modeling and Wald tests: "It then compares methylation levels using beta-binomial hierarchical modeling and Wald tests."
- [methods] findDMRs.r conducts pairwise tests: "The DMRfinder script `findDMRs.r` conducts pairwise tests of sample groups to find genomic regions that are differentially methylated"
- [methods] Filter defaults in findDMRs.r: "-d <float>    Min. methylation difference between sample groups
                    ([0-1]; def. 0.10)"
- [readme] Step 4 invocation from README: "$ Rscript findDMRs.r  -i combined.csv  -o results.csv  -v  -n Control,Exptl  C1,C2  E1,E2"
- [readme] DSS statistical framework: "It then performs pairwise comparisons to identify differentially methylated regions (DMRs) using the statistical framework of the Bioconductor package DSS"
