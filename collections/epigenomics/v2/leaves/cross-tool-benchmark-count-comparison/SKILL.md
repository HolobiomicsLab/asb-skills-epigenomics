---
name: cross-tool-benchmark-count-comparison
description: Use when evaluating or benchmarking DMR-calling software with a same-sample
  replicate pair where no differential methylation should be detected, and you need
  comparable region counts across tools.
when_to_use_negative:
- Input consists of genuinely different biological groups (e.g. IMR90 vs H1), where
  DMRs are expected and the zero-DMR null does not apply
- You need region-set or quantitative-trait modeling, or batch-effect correction —
  DMRfinder explicitly does not support these
- You only want per-CpG-site differential testing rather than region-level clustering
  and testing
edam_operation: http://edamontology.org/operation_3208
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_3168
tools:
- name: DMRfinder
  role: 'Primary tool under test: extract_CpG_data.py, combine_CpG_sites.py, and findDMRs.r
    extract counts, cluster CpGs, and test the same-sample replicates'
  repo: https://github.com/jsh58/DMRfinder
- name: DSS
  role: Alternative comparison method using the beta-binomial/Wald-test statistical
    framework applied directly on the same replicate pair
  repo: http://www.bioconductor.org/packages/release/bioc/html/DSS.html
- name: Bismark
  role: Provides the alignments analyzed by DMRfinder and the alternative bismark_methylation_extractor/coverage2cytosine
    route benchmarked against it
  repo: https://github.com/FelixKrueger/Bismark
- name: Python
  role: Runs the DMRfinder scripts extract_CpG_data.py and combine_CpG_sites.py
  repo: https://github.com/jsh58/DMRfinder
- name: R
  role: Runs findDMRs.r via Rscript and hosts the DSS package for the alternative
    approach
  repo: https://github.com/jsh58/DMRfinder
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-017-1909-0
    title: 'DMRfinder: efficiently identifying differentially methylated regions from
      MethylC-seq data'
  source_content_hash: sha256:0f63a278e220b60b548a663d13f79cb057ad8bee2ed0559432fb35c1af5a4a4b
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
  edam_operation: http://edamontology.org/operation_3208
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cross-tool-benchmark-count-comparison

## Summary

A negative-control benchmarking skill: contrast two replicates of the same sample (IMR90), where zero DMRs is the expected outcome, and compare the number of reported regions from DMRfinder against alternative software (DSS, Bismark's methylation-extraction route) to expose false positives.

## When to use

Use when evaluating or benchmarking DMR-calling software with a same-sample replicate pair where no differential methylation should be detected, and you need comparable region counts across tools.

## When NOT to use

- Input consists of genuinely different biological groups (e.g. IMR90 vs H1), where DMRs are expected and the zero-DMR null does not apply
- You need region-set or quantitative-trait modeling, or batch-effect correction — DMRfinder explicitly does not support these
- You only want per-CpG-site differential testing rather than region-level clustering and testing

## Inputs

- Bismark SAM/BAM alignment files for two replicates of the same sample (IMR90), with XM methylation strings and XG genome designations
- Reference genome used for Bismark alignment
- Methylation count tables (.cov) per replicate produced by extract_CpG_data.py

## Outputs

- Combined region table (combined.csv) from combine_CpG_sites.py clustering
- Per-tool DMR result tables (e.g. results.csv from findDMRs.r) listing differentially methylated regions
- Comparison table (false_positives.csv) of region counts reported by each tool for the self-replicate contrast

## How to apply

Extract methylation counts from Bismark alignment files (SAM/BAM with XM methylation strings and XG genome designations) for the two same-sample IMR90 replicates using DMRfinder's extract_CpG_data.py, which merges cytosine and guanine counts per CpG. Cluster CpG sites into regions with combine_CpG_sites.py using default parameters (-r 3 minimum counts, -s 1 minimum samples, -d 100 max CpG distance, -c 3 minimum CpGs, -x 500 bp max length, -m 20 minimum total counts). Test the two replicates as separate groups with findDMRs.r at default thresholds -d 0.10 (minimum methylation difference) and -p 0.05 (maximum p-value). Run the alternatives — DSS's beta-binomial/Wald-test framework and Bismark's bismark_methylation_extractor/coverage2cytosine route — on the same replicate pair under comparable criteria. Count the genomic regions each tool reports as differentially methylated and compile them into a single table (false_positives.csv); correct application means DMRfinder yields minimal regions while the alternatives yield a substantial number of false positives.

## Related tools

- **DMRfinder** (Primary tool under test: extract_CpG_data.py, combine_CpG_sites.py, and findDMRs.r extract counts, cluster CpGs, and test the same-sample replicates) — https://github.com/jsh58/DMRfinder
- **DSS** (Alternative comparison method using the beta-binomial/Wald-test statistical framework applied directly on the same replicate pair) — http://www.bioconductor.org/packages/release/bioc/html/DSS.html
- **Bismark** (Provides the alignments analyzed by DMRfinder and the alternative bismark_methylation_extractor/coverage2cytosine route benchmarked against it) — https://github.com/FelixKrueger/Bismark
- **Python** (Runs the DMRfinder scripts extract_CpG_data.py and combine_CpG_sites.py) — https://github.com/jsh58/DMRfinder
- **R** (Runs findDMRs.r via Rscript and hosts the DSS package for the alternative approach) — https://github.com/jsh58/DMRfinder

## Examples

```
samtools view -h C1_bismark_bt2.bam | python extract_CpG_data.py -i - -o C1.cov -v
```

## Evaluation signals

- DMRfinder reports minimal (near-zero) genomic regions in the self-replicate contrast at -d 0.10 / -p 0.05
- The two alternative approaches (DSS and the Bismark methylation-extractor route) report a substantial number of false-positive regions on the same replicate pair
- false_positives.csv contains a comparable region count for every tool run on the identical replicate pair and clustering parameters
- Region clustering output respects the combine_CpG_sites.py defaults (regions of ≥3 CpGs, ≤500 bp, ≥20 total counts, max 100 bp between CpGs)
- Coverage-filtered count tables contain merged cytosine-plus-guanine counts per CpG, sorted by chromosome and position

## Limitations

- Bismark version 0.16.0 contains a bug related to ambiguous read alignments; version 0.15.0 is recommended until fixed
- Bismark's deduplication script is not recommended because it keeps the first read at a position regardless of methylation information
- DMRfinder does not consider sets of regions or quantitative traits and cannot account for batch effects

## Evidence

- [abstract] Self-replicate null expectation: "Contrasting two replicates of the same sample yielded minimal genomic regions with DMRfinder, whereas two alternative software packages reported a substantial number of false positives."
- [readme] Count extraction method: "The DMRfinder script `extract_CpG_data.py` converts the output from Bismark's aligner into a table of methylated/unmethylated counts at each CpG site."
- [methods] Region-level testing: "The DMRfinder script `findDMRs.r` conducts pairwise tests of sample groups to find genomic regions that are differentially methylated"
- [methods] Significance threshold: "-p <float>    Max. p-value ([0-1]; def. 0.05)"
- [readme] Statistical framework: "It then performs pairwise comparisons to identify differentially methylated regions (DMRs) using the statistical framework of the Bioconductor package DSS"
