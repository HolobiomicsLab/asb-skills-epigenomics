---
name: methylc-seq-region-dmr-identification
description: Use when you have per-sample clustered-region methylation counts (combined.csv
  with chr, start, end, CpG and per-sample total/methylated count columns) from two
  or more sample groups and need to call DMRs between groups.
when_to_use_negative:
- You need to test individual CpG sites rather than multi-CpG regions — DMRfinder's
  final step conglomerates counts within clusters and tests whole regions
- Your design includes batch effects or quantitative traits / sets of regions — the
  article states DMRfinder 'cannot account for batch effects' and 'does not consider
  sets of regions or quantitative traits'
- You have raw Bismark SAM output instead of clustered-region counts — extract_CpG_data.py
  and combine_CpG_sites.py must run first before findDMRs.r
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0091
tools:
- name: DMRfinder
  role: Pipeline package whose findDMRs.r script conducts pairwise tests of sample
    groups to find differentially methylated regions
  repo: https://github.com/jsh58/DMRfinder
- name: DSS
  role: Bioconductor package providing the beta-binomial hierarchical model and Wald
    tests invoked by findDMRs.r (tested with version 2.14.0)
  repo: http://www.bioconductor.org/packages/release/bioc/html/DSS.html
- name: R
  role: Runtime environment for findDMRs.r (tested with Rscript version 3.3.1)
  repo: https://www.r-project.org/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-017-1909-0
    title: 'DMRfinder: efficiently identifying differentially methylated regions from
      MethylC-seq data'
  source_content_hash: sha256:2baaae6bd0715829f29ca16c7b16e5502e8d5ddbd936cf170f45c070a62032b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jsh58/DMRfinder
  tools:
  - DMRfinder
  - DSS
  - R
  - findDMRs.r
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# methylc-seq-region-dmr-identification

## Summary

Identify differentially methylated regions (DMRs) from MethylC-seq data by testing clustered genomic regions with DSS beta-binomial hierarchical modeling and Wald tests via DMRfinder's findDMRs.r script, rather than testing individual CpG sites.

## When to use

Use when you have per-sample clustered-region methylation counts (combined.csv with chr, start, end, CpG and per-sample total/methylated count columns) from two or more sample groups and need to call DMRs between groups.

## When NOT to use

- You need to test individual CpG sites rather than multi-CpG regions — DMRfinder's final step conglomerates counts within clusters and tests whole regions
- Your design includes batch effects or quantitative traits / sets of regions — the article states DMRfinder 'cannot account for batch effects' and 'does not consider sets of regions or quantitative traits'
- You have raw Bismark SAM output instead of clustered-region counts — extract_CpG_data.py and combine_CpG_sites.py must run first before findDMRs.r

## Inputs

- combined.csv: clustered-region methylation counts table from combine_CpG_sites.py containing chr, start, end, CpG columns plus per-sample total counts (-N) and methylated counts (-X) columns
- Sample-to-group assignment defining sample groups (e.g. Control: C1,C2; Exptl: E1,E2)

## Outputs

- results.csv: filtered table of differentially methylated regions with chr, start, end, CpG, mu, diff, and pval columns per group-group comparison
- diff: methylation difference of group2 relative to group1 for each region

## How to apply

First cluster CpG sites into regions using combine_CpG_sites.py (modified single-linkage clustering with default max CpG distance 100 bp, then split regions longer than 500 bp), producing a combined.csv with -N total and -X methylated counts per sample. Then run findDMRs.r via 'Rscript findDMRs.r -i combined.csv -o results.csv C1,C2 E1,E2 -n Ctrl,Exp', which fits the DSS (v2.14.0) Bayesian beta-binomial hierarchical model and Wald tests on whole regions. Apply the default region filters: minimum 3 CpGs per region (-c 3), minimum 0.10 methylation difference between groups (-d 0.10), maximum p-value 0.05 (-p 0.05), and maximum q-value 1 (-q 1). Report chr, start, end, CpG, mu (group methylation fraction), diff (group2 relative to group1), and pval per group-group comparison. Testing whole regions instead of individual CpGs reduces the number of hypothesis tests and increases statistical power because methylation counts are conglomerated within each cluster.

## Related tools

- **DMRfinder** (Pipeline package whose findDMRs.r script conducts pairwise tests of sample groups to find differentially methylated regions) — https://github.com/jsh58/DMRfinder
- **DSS** (Bioconductor package providing the beta-binomial hierarchical model and Wald tests invoked by findDMRs.r (tested with version 2.14.0)) — http://www.bioconductor.org/packages/release/bioc/html/DSS.html
- **R** (Runtime environment for findDMRs.r (tested with Rscript version 3.3.1)) — https://www.r-project.org/

## Examples

```
Rscript findDMRs.r -i combined.csv -o results.csv -v -n Control,Exptl C1,C2 E1,E2
```

## Evaluation signals

- Each reported region passes the default filters: ≥3 CpGs, ≥0.10 methylation difference between groups, p-value ≤ 0.05
- results.csv contains the expected columns chr, start, end, CpG, mu, diff, and pval for each group-group comparison
- Row-by-row comparison of results.csv (region coordinates, mu, diff, pval) against the reference DMRfinder output on the identical clustered-region input shows an exact match
- The input combined.csv schema includes chr, start, end, CpG plus per-sample -N (total) and -X (methylated) count columns as produced by combine_CpG_sites.py
- DSS version 2.14.0 and Rscript version 3.3.1 (the tested versions) are used to reproduce reference behavior

## Limitations

- DMRfinder cannot account for batch effects
- DMRfinder does not allow modeling of sets of regions or quantitative traits
- A region whose length exceeds the specified threshold (default 500 bp) is split to limit the chaining effect of single-linkage clustering, so very long co-methylated blocks are not tested as single regions

## Evidence

- [methods] The DMRfinder script `findDMRs.r` conducts pairwise tests of sample groups to find genomic regions that are differentially methylated: "The DMRfinder script `findDMRs.r` conducts pairwise tests of sample groups to find genomic regions that are differentially methylated"
- [readme] It then performs pairwise comparisons to identify differentially methylated regions (DMRs) using the statistical framework of the Bioconductor package DSS: "It then performs pairwise comparisons to identify differentially methylated regions (DMRs) using the statistical framework of the Bioconductor package DSS"
- [abstract] It compares methylation levels using beta-binomial hierarchical modeling and Wald tests.: "It then compares methylation levels using beta-binomial hierarchical modeling and Wald tests."
- [methods] -d <float>    Min. methylation difference between sample groups: "-d <float>    Min. methylation difference between sample groups
                    ([0-1]; def. 0.10)"
- [readme] DMRfinder scripts have been tested with Rscript version 3.3.1 and DSS version 2.14.0.: "DMRfinder scripts have been tested with Rscript version 3.3.1 and DSS version 2.14.0."
