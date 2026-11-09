---
name: dmr-threshold-filtering
description: Use when beta-binomial-tested clustered regions from MethylC-seq data
  must be filtered into a final DMR table using CpG-count, methylation-difference,
  and p/q-value cutoffs.
when_to_use_negative:
- Input is raw per-CpG methylation counts that have not yet been clustered into regions
  by combine_CpG_sites.py
- The analysis requires modeling sets of regions, quantitative traits, or batch effects,
  which DMRfinder explicitly does not support
- The goal is single-CpG-site differential testing rather than whole-region testing
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
tools:
- name: findDMRs.r
  role: R script that fits the DSS beta-binomial hierarchical model and Wald tests
    on clustered regions and applies the CpG-count, methylation-difference, and p/q-value
    reporting filters
  repo: https://github.com/jsh58/DMRfinder
- name: DSS
  role: Bioconductor package (v2.14.0) supplying the beta-binomial hierarchical modeling
    and Wald test statistics underlying the differential methylation tests
  repo: http://www.bioconductor.org/packages/release/bioc/html/DSS.html
- name: DMRfinder
  role: Pipeline package whose final statistical step performs the region-level differential
    testing and threshold filtering
  repo: https://github.com/jsh58/DMRfinder
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-017-1909-0
    title: 'DMRfinder: efficiently identifying differentially methylated regions from
      MethylC-seq data'
  source_content_hash: sha256:4ff62c2af58d8654f106b7742c96ed43ba323a73453c3fa42e43d8ad2c1133ca
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
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# dmr-threshold-filtering

## Summary

Filters clustered genomic regions for differential methylation by applying DMRfinder's reporting thresholds — minimum CpG count, minimum methylation difference, and maximum p-/q-value cutoffs — after beta-binomial testing with DSS, so that only biologically meaningful DMRs are reported.

## When to use

Use when beta-binomial-tested clustered regions from MethylC-seq data must be filtered into a final DMR table using CpG-count, methylation-difference, and p/q-value cutoffs.

## When NOT to use

- Input is raw per-CpG methylation counts that have not yet been clustered into regions by combine_CpG_sites.py
- The analysis requires modeling sets of regions, quantitative traits, or batch effects, which DMRfinder explicitly does not support
- The goal is single-CpG-site differential testing rather than whole-region testing

## Inputs

- combined.csv clustered-region methylation counts file with chr, start, end, CpG columns plus per-sample total counts (-N) and methylated counts (-X) columns
- Sample-to-group assignment (e.g., C1,C2 as Control; E1,E2 as Exptl)

## Outputs

- results.csv filtered DMR table with chr, start, end, CpG, and per-comparison mu, diff, pval columns

## How to apply

Run findDMRs.r on the combined.csv clustered-region count table produced by combine_CpG_sites.py, which fits the DSS beta-binomial hierarchical model and Wald tests on whole regions rather than individual CpG sites (reducing the number of hypothesis tests and increasing power). Apply the default region filters via command-line options: minimum 3 CpGs per region (-c 3), minimum methylation difference of 0.10 between groups (-d 0.10), maximum p-value 0.05 (-p 0.05), and maximum q-value 1 (-q 1). A region is reported only if it passes all thresholds for a given group-group comparison. Report the DSS columns mu (group methylation fraction), diff (group2 relative to group1), and pval alongside chr, start, end, and CpG in results.csv.

## Related tools

- **findDMRs.r** (R script that fits the DSS beta-binomial hierarchical model and Wald tests on clustered regions and applies the CpG-count, methylation-difference, and p/q-value reporting filters) — https://github.com/jsh58/DMRfinder
- **DSS** (Bioconductor package (v2.14.0) supplying the beta-binomial hierarchical modeling and Wald test statistics underlying the differential methylation tests) — http://www.bioconductor.org/packages/release/bioc/html/DSS.html
- **DMRfinder** (Pipeline package whose final statistical step performs the region-level differential testing and threshold filtering) — https://github.com/jsh58/DMRfinder

## Examples

```
Rscript findDMRs.r -i combined.csv -o results.csv C1,C2 E1,E2 -n Ctrl,Exp
```

## Evaluation signals

- Every reported region contains at least 3 CpGs (the -c 3 default)
- Each reported region shows an absolute methylation difference of at least 0.10 between sample groups (the -d 0.10 default)
- Each reported region has a p-value at or below 0.05 (the -p 0.05 default)
- Output results.csv contains the columns chr, start, end, CpG, and per-comparison mu, diff, pval
- results.csv matches the reference DMRfinder output table row-by-row (region coordinates, mu, diff, pval) on identical clustered-region input

## Limitations

- DMRfinder identifies individual differentially methylated regions but does not consider sets of regions or quantitative traits
- DMRfinder cannot account for batch effects
- Because methylation counts are conglomerated within each cluster, the test evaluates whole regions rather than individual CpG sites

## Evidence

- [methods] Region filters: CpG count, methylation difference, p/q-value: "-c <int>    Min. number of CpGs in a region (def. 3)"
- [methods] Methylation difference threshold: "-d <float>    Min. methylation difference between sample groups
                    ([0-1]; def. 0.10)"
- [abstract] Statistical framework: "It then compares methylation levels using beta-binomial hierarchical modeling and Wald tests."
- [results] Region-level testing rationale: [evidence span withheld — anchor status: not_found]
- [methods] findDMRs.r role: "The DMRfinder script `findDMRs.r` conducts pairwise tests of sample groups to find genomic regions that are differentially methylated"
- [discussion] Scope limitation: "It identifies individual regions that are differentially methylated but does not consider sets of regions or quantitative traits"
- [readme] Example invocation: "$ Rscript findDMRs.r  -i combined.csv  -o results.csv  -v  -n Control,Exptl  C1,C2  E1,E2"
