---
name: pairwise-group-methylation-comparison
description: Use when you have a clustered-region methylation counts table (combined.csv
  with chr, start, end, CpG and per-sample -N/-X count columns) and need to identify
  DMRs between two or more sample groups.
when_to_use_negative:
- You need to model sets of regions or quantitative traits — DMRfinder identifies
  individual differentially methylated regions only
- Your experiment has batch effects that must be accounted for — DMRfinder cannot
  account for batch effects
- You want to test individual CpG sites rather than clustered genomic regions
edam_operation: http://edamontology.org/operation_3206
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0203
tools:
- name: findDMRs.r
  role: DMRfinder R script that conducts pairwise tests of sample groups to find differentially
    methylated regions and writes results.csv
  repo: https://github.com/jsh58/DMRfinder
- name: DSS
  role: Bioconductor package (v2.14.0) providing the Bayesian beta-binomial hierarchical
    model and Wald tests used for regional differential methylation testing
- name: R
  role: Runtime environment for findDMRs.r (tested with Rscript version 3.3.1)
- name: combine_CpG_sites.py
  role: Upstream DMRfinder script that clusters CpG sites and produces the combined
    counts file consumed by findDMRs.r
  repo: https://github.com/jsh58/DMRfinder
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-017-1909-0
    title: 'DMRfinder: efficiently identifying differentially methylated regions from
      MethylC-seq data'
  source_content_hash: sha256:4dddd43c301e3eec86284948487be14954e725384211187927419c34eb7896cd
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
  edam_operation: http://edamontology.org/operation_3206
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# pairwise-group-methylation-comparison

## Summary

Tests clustered genomic regions for differential methylation between sample groups using beta-binomial hierarchical modeling and Wald tests via the Bioconductor package DSS, as implemented in DMRfinder's findDMRs.r script.

## When to use

Use when you have a clustered-region methylation counts table (combined.csv with chr, start, end, CpG and per-sample -N/-X count columns) and need to identify DMRs between two or more sample groups.

## When NOT to use

- You need to model sets of regions or quantitative traits — DMRfinder identifies individual differentially methylated regions only
- Your experiment has batch effects that must be accounted for — DMRfinder cannot account for batch effects
- You want to test individual CpG sites rather than clustered genomic regions

## Inputs

- combined.csv clustered-region methylation counts file (chr, start, end, CpG columns plus per-sample total counts -N and methylated counts -X columns from combine_CpG_sites.py)
- Sample-to-group assignments (e.g. C1,C2 for Control; E1,E2 for Exptl)

## Outputs

- results.csv table of differentially methylated regions with chr, start, end, CpG, and per-comparison DSS columns mu, diff, and pval

## How to apply

Run `Rscript findDMRs.r -i combined.csv -o results.csv C1,C2 E1,E2 -n Ctrl,Exp` on the clustered-region counts file produced by combine_CpG_sites.py; findDMRs.r fits the DSS beta-binomial hierarchical model and applies Wald tests to whole regions rather than individual CpG sites. Because methylation counts are conglomerated within each region, the number of hypothesis tests is reduced and statistical power increases. Apply the default region filters: minimum 3 CpGs per region (-c 3), minimum methylation difference of 0.10 between groups (-d 0.10), maximum p-value 0.05 (-p 0.05), and maximum q-value 1 (-q 1). The output reports per-comparison DSS columns mu (group methylation fraction), diff (group2 relative to group1), and pval alongside chr, start, end, and CpG. Optionally use -up to report only regions hypermethylated in the later group.

## Related tools

- **findDMRs.r** (DMRfinder R script that conducts pairwise tests of sample groups to find differentially methylated regions and writes results.csv) — https://github.com/jsh58/DMRfinder
- **DSS** (Bioconductor package (v2.14.0) providing the Bayesian beta-binomial hierarchical model and Wald tests used for regional differential methylation testing)
- **R** (Runtime environment for findDMRs.r (tested with Rscript version 3.3.1))
- **combine_CpG_sites.py** (Upstream DMRfinder script that clusters CpG sites and produces the combined counts file consumed by findDMRs.r) — https://github.com/jsh58/DMRfinder

## Examples

```
Rscript findDMRs.r -i combined.csv -o results.csv -v -n Control,Exptl C1,C2 E1,E2
```

## Evaluation signals

- Every reported region satisfies the minimum 10% methylation difference and a p-value at or below 0.05
- Each reported region contains at least 3 CpGs (default -c 3)
- The produced results.csv matches the reference DMRfinder output row-by-row (region coordinates, mu, diff, pval values) on the identical clustered-region input
- Output contains the expected DSS columns mu, diff, and pval for each group-group comparison plus chr, start, end, CpG
- Rscript and DSS versions match the tested configuration (Rscript 3.3.1, DSS 2.14.0)

## Limitations

- DMRfinder does not consider sets of regions or quantitative traits, only individual differentially methylated regions
- DMRfinder cannot account for batch effects
- Contrasting replicates of the same sample should yield minimal DMRs; other packages reported substantial false positives in this setting, so spurious calls may indicate a modeling issue

## Evidence

- [abstract] It then compares methylation levels using beta-binomial hierarchical modeling and Wald tests.: "It then compares methylation levels using beta-binomial hierarchical modeling and Wald tests."
- [methods] The DMRfinder script findDMRs.r conducts pairwise tests of sample groups: "The DMRfinder script `findDMRs.r` conducts pairwise tests of sample groups to find genomic regions that are differentially methylated"
- [methods] Minimum methylation difference filter default: "-d <float>    Min. methylation difference between sample groups
                    ([0-1]; def. 0.10)"
- [readme] README quick start step 4 invocation: "$ Rscript findDMRs.r  -i combined.csv  -o results.csv  -v  -n Control,Exptl  C1,C2  E1,E2"
- [readme] README on DSS dependency: "It then performs pairwise comparisons to identify differentially methylated regions (DMRs) using the statistical framework of the Bioconductor package DSS"
