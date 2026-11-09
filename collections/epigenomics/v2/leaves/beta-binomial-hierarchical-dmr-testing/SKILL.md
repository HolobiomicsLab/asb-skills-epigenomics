---
name: beta-binomial-hierarchical-dmr-testing
description: Use when Bismark-aligned MethylC-seq reads must be tested for region-level
  differential methylation between two sample groups (e.g., IMR90/H1 replicates) with
  strict false-positive control, such that self-replicate contrasts yield minimal
  DMRs.
when_to_use_negative:
- You need to model sets of regions or quantitative traits — DMRfinder identifies
  individual differentially methylated regions only
- Batch effects must be adjusted for — DMRfinder cannot account for batch effects
- The goal is single-CpG-site differential testing rather than multi-CpG region DMRs
  — the pipeline deliberately tests regions of multiple CpG sites
edam_operation: http://edamontology.org/operation_3206
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0092
tools:
- name: DMRfinder
  role: 'Runs the full post-alignment pipeline: extract_CpG_data.py merges CpG counts,
    combine_CpG_sites.py clusters regions, findDMRs.r performs the beta-binomial Wald
    tests'
  repo: https://github.com/jsh58/DMRfinder
- name: DSS
  role: Bioconductor package whose statistical framework underlies findDMRs.r's beta-binomial
    hierarchical modeling and Wald tests (tested with DSS 2.14.0)
  repo: http://www.bioconductor.org/packages/release/bioc/html/DSS.html
- name: Bismark
  role: Aligns MethylC-seq reads and produces the SAM alignments (XM/XG tags) that
    extract_CpG_data.py converts to methylation count tables; v0.15.0 recommended
  repo: https://github.com/FelixKrueger/Bismark
- name: R
  role: Runtime for findDMRs.r; DMRfinder scripts tested with Rscript 3.3.1
  repo: https://www.r-project.org/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-017-1909-0
    title: 'DMRfinder: efficiently identifying differentially methylated regions from
      MethylC-seq data'
  source_content_hash: sha256:2090750c1d7b4fd85a85f725ae180a16bb243872489989cb0926b05b7302f394
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jsh58/DMRfinder
  tools:
  - Bismark
  - DMRfinder
  - DSS
  - Python
  - R
  edam_operation: http://edamontology.org/operation_3206
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 20
---

# beta-binomial-hierarchical-dmr-testing

## Summary

Region-level differential methylation testing for MethylC-seq data: cluster CpG sites with DMRfinder's combine_CpG_sites.py and test sample groups with findDMRs.r, which applies beta-binomial hierarchical modeling (the DSS statistical framework) with Wald tests. This design kept same-sample IMR90 replicate contrasts near zero DMRs where alternative software reported substantial false positives.

## When to use

Use when Bismark-aligned MethylC-seq reads must be tested for region-level differential methylation between two sample groups (e.g., IMR90/H1 replicates) with strict false-positive control, such that self-replicate contrasts yield minimal DMRs.

## When NOT to use

- You need to model sets of regions or quantitative traits — DMRfinder identifies individual differentially methylated regions only
- Batch effects must be adjusted for — DMRfinder cannot account for batch effects
- The goal is single-CpG-site differential testing rather than multi-CpG region DMRs — the pipeline deliberately tests regions of multiple CpG sites

## Inputs

- Bismark SAM/BAM alignment files (with header, 'XM' methylation strings, and 'XG' genome designations) per sample
- Combined region-level methylation count table (combined.csv) from combine_CpG_sites.py
- Sample group definitions naming samples per group (e.g., Control: C1,C2; Exptl: E1,E2) passed via -n

## Outputs

- DMR results table (results.csv) listing genomic regions with methylation difference, p-value, and q-value
- Region-level combined methylation counts (combined.csv) summing methylated/unmethylated CpG counts within clusters
- False-positive comparison table (false_positives.csv) compiling DMR counts from DMRfinder, DSS, and the Bismark route on the same replicate pair

## How to apply

Align MethylC-seq reads with Bismark (v0.15.0 recommended, since 0.16.0 introduced an ambiguous-alignment bug), then pipe the SAM files with 'XM' methylation strings and 'XG' strand tags to extract_CpG_data.py to merge cytosine and guanine counts per CpG. Cluster CpGs with combine_CpG_sites.py using defaults -r 3 (min counts/site), -s 1, -d 100 (max inter-CpG distance), -c 3 (min CpGs/region), -x 500 (max region length), -m 20 (min total counts); clustering ignores methylation levels and thus is not biased toward finding DMRs, and regions exceeding 500 bp are split to limit single-linkage chaining. Run Rscript findDMRs.r on the combined counts, declaring groups with -n (e.g., -n Control,Exptl C1,C2 E1,E2), which applies DSS's beta-binomial hierarchical modeling and Wald tests per region. Retain regions passing the default significance filters: methylation difference >= 0.10 (-d) and p-value <= 0.05 (-p); note that -q defaults to 1, so no q-value/FDR cutoff is imposed unless set explicitly. Sanity-check on a self-replicate contrast (two replicates of the same IMR90 sample): DMRfinder should report minimal regions while direct DSS and the Bismark methylation-extractor/coverage2cytosine route yield substantial false positives under comparable criteria.

## Related tools

- **DMRfinder** (Runs the full post-alignment pipeline: extract_CpG_data.py merges CpG counts, combine_CpG_sites.py clusters regions, findDMRs.r performs the beta-binomial Wald tests) — https://github.com/jsh58/DMRfinder
- **DSS** (Bioconductor package whose statistical framework underlies findDMRs.r's beta-binomial hierarchical modeling and Wald tests (tested with DSS 2.14.0)) — http://www.bioconductor.org/packages/release/bioc/html/DSS.html
- **Bismark** (Aligns MethylC-seq reads and produces the SAM alignments (XM/XG tags) that extract_CpG_data.py converts to methylation count tables; v0.15.0 recommended) — https://github.com/FelixKrueger/Bismark
- **R** (Runtime for findDMRs.r; DMRfinder scripts tested with Rscript 3.3.1) — https://www.r-project.org/

## Examples

```
Rscript findDMRs.r -i combined.csv -o results.csv -v -n Control,Exptl C1,C2 E1,E2
```

## Evaluation signals

- Self-replicate contrast of two replicates of the same IMR90 sample reports minimal DMRs, while direct DSS and Bismark's coverage2cytosine route report substantial false positives on the same input
- Every reported region satisfies the significance filters: methylation difference >= 0.10 and p-value <= 0.05 (q-value <= 1 by default)
- Regions conform to clustering constraints: >= 3 CpGs (-c 3), max length 500 bp (-x 500), min total counts 20 (-m 20), max inter-CpG distance 100 bp (-d 100)
- Output schema matches DMRfinder's results.csv: region coordinates plus per-sample methylated/unmethylated counts and test statistics
- Efficiency check: extract_CpG_data.py processed IMR90 replicate 1a in 9,378 sec versus Bismark's 22,514 sec on the same data

## Limitations

- Does not consider sets of regions or quantitative traits; it identifies individual regions only
- Cannot account for batch effects
- Bismark 0.16.0 introduced a bug with ambiguous read alignments; version 0.15.0 is recommended until fixed
- The default q-value threshold is 1, so reported DMRs are not FDR-filtered unless -q is lowered

## Evidence

- [abstract] It then compares methylation levels using beta-binomial hierarchical modeling and Wald tests.: "It then compares methylation levels using beta-binomial hierarchical modeling and Wald tests."
- [abstract] Contrasting two replicates of the same sample yielded minimal genomic regions with DMRfinder, whereas two alternative software packages reported a substantial number of false positives.: "Contrasting two replicates of the same sample yielded minimal genomic regions with DMRfinder, whereas two alternative software packages reported a substantial number of false positives."
- [methods] The DMRfinder script `findDMRs.r` conducts pairwise tests of sample groups to find genomic regions that are differentially methylated: "The DMRfinder script `findDMRs.r` conducts pairwise tests of sample groups to find genomic regions that are differentially methylated"
- [methods] -p <float>    Max. p-value ([0-1]; def. 0.05]: "-p <float>    Max. p-value ([0-1]; def. 0.05]"
- [methods] The DMRfinder script limits this effect by splitting any region whose length exceeds a specified threshold (by default, 500 bp): "The DMRfinder script limits this effect by splitting any region whose length exceeds a specified threshold (by default, 500 bp)"
- [readme] performs pairwise comparisons to identify differentially methylated regions (DMRs) using the statistical framework of the Bioconductor package DSS: "performs pairwise comparisons to identify differentially methylated regions (DMRs) using the statistical framework of the Bioconductor package DSS"
- [readme] DMRfinder scripts have been tested with Rscript version 3.3.1 and DSS version 2.14.0.: "DMRfinder scripts have been tested with Rscript version 3.3.1 and DSS version 2.14.0."
- [discussion] It identifies individual regions that are differentially methylated but does not consider sets of regions or quantitative traits: "It identifies individual regions that are differentially methylated but does not consider sets of regions or quantitative traits"
