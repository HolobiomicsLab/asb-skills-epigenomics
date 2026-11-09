---
name: genomic-region-coordinate-aggregation
description: Use when individual-CpG methylation count tables (e.g. Bismark-derived
  .cov files with methylated/unmethylated counts per site) must be collapsed into
  genomic regions before differential methylation testing.
when_to_use_negative:
- Input is already a region-level methylation table rather than per-CpG-site counts
- The analysis goal requires modeling sets of regions or quantitative traits, or accounting
  for batch effects — DMRfinder explicitly does not support these
- Methylation-level-guided region definition is desired; this clustering deliberately
  ignores methylation levels
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
tools:
- name: DMRfinder
  role: Provides combine_CpG_sites.py, the script implementing the modified single-linkage
    clustering of CpG sites into regions
  repo: https://github.com/jsh58/DMRfinder
- name: Python
  role: Runtime for combine_CpG_sites.py and for reimplementing/validating the clustering
    step
  repo: https://www.python.org/downloads/
- name: Bismark
  role: Upstream aligner whose output is converted to the per-CpG count tables that
    feed the clustering step
  repo: FelixKrueger/Bismark
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-017-1909-0
    title: 'DMRfinder: efficiently identifying differentially methylated regions from
      MethylC-seq data'
  source_content_hash: sha256:9667a1650e3a13d822835516b2a22050c210bcd230472bcf87ef32dbd7c929cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jsh58/DMRfinder
  tools:
  - Bismark
  - DMRfinder
  - Python
  - combine_CpG_sites.py
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# genomic-region-coordinate-aggregation

## Summary

Aggregates per-CpG-site methylation counts into genomic regions using a modified single-linkage clustering that chains sites within a maximum inter-site distance, then splits oversized regions, so downstream DMR testing operates on multi-CpG regions rather than individual sites.

## When to use

Use when individual-CpG methylation count tables (e.g. Bismark-derived .cov files with methylated/unmethylated counts per site) must be collapsed into genomic regions before differential methylation testing.

## When NOT to use

- Input is already a region-level methylation table rather than per-CpG-site counts
- The analysis goal requires modeling sets of regions or quantitative traits, or accounting for batch effects — DMRfinder explicitly does not support these
- Methylation-level-guided region definition is desired; this clustering deliberately ignores methylation levels

## Inputs

- Per-sample methylation-count tables from extract_CpG_data.py (tab-delimited: chromosome, position, methylated counts, unmethylated counts)
- Clustering parameters: -d 100 (max inter-site distance), -c 3 (min CpGs per region), -x 500 (max region length), -r 3/-s 1 (coverage validity), -m 20 (min regional total counts)

## Outputs

- A single combined region table (combined.csv) in chr/start/end/CpG format with per-sample methylated (-N) and total (-X) count columns, NA where regional totals fall below the -m threshold

## How to apply

Load the per-sample methylation-count tables produced by extract_CpG_data.py (chromosome, 1-based position, methylated counts, unmethylated counts per line), and determine valid CpG sites per sample using coverage thresholds (-r 3 minimum counts per position, -s 1 minimum number of samples meeting that threshold). Cluster adjacent valid CpG sites by single linkage with a maximum inter-site distance of 100 bp (-d 100), then split any region longer than 500 bp (-x 500) into subregions to limit the chaining effect, requiring each subcluster to retain at least 3 CpGs (-c 3). Sum methylated and total counts per region for each sample, reporting NA for samples whose regional total falls below -m 20, and write the chr/start/end/CpG plus per-sample -N/-X column format. Clustering is driven only by site coordinates, never methylation levels, which keeps region construction unbiased toward (or against) finding DMRs.

## Related tools

- **DMRfinder** (Provides combine_CpG_sites.py, the script implementing the modified single-linkage clustering of CpG sites into regions) — https://github.com/jsh58/DMRfinder
- **Python** (Runtime for combine_CpG_sites.py and for reimplementing/validating the clustering step) — https://www.python.org/downloads/
- **Bismark** (Upstream aligner whose output is converted to the per-CpG count tables that feed the clustering step) — FelixKrueger/Bismark

## Examples

```
python combine_CpG_sites.py -v -o combined.csv C1.cov C2.cov E1.cov E2.cov
```

## Evaluation signals

- Clustered region table matches the output of the reference combine_CpG_sites.py run on the same input (output_matches_reference check on combined.csv)
- Every reported region contains at least the minimum CpG count (-c 3), including split subregions
- No region exceeds the maximum length threshold (-x 500 bp) after splitting
- Samples with regional total counts below -m 20 are reported as NA, not as partial counts
- Regions and summed counts are invariant to permutations of methylation values across samples, since clustering uses only site coordinates

## Limitations

- Single-linkage clustering chains together sites in dense CpG-rich loci; the -x 500 bp split mitigates but does not eliminate this effect
- The clustering considers only CpG site locations and cannot model sets of regions, quantitative traits, or batch effects
- Split subclusters that lose the minimum CpG count (-c 3) are dropped, which can discard sites in split regions

## Evidence

- [methods] The DMRfinder script limits this effect by splitting any region whose length exceeds a specified threshold (by default, 500 bp): "The DMRfinder script limits this effect by splitting any region whose length exceeds a specified threshold (by default, 500 bp)"
- [methods] it does not consider methylation levels, and thus is not biased in favor of (or opposed to) finding DMRs: "it does not consider methylation levels, and thus is not biased in favor of (or opposed to) finding DMRs"
- [methods] It clusters adjacent CpG sites into genomic regions, following a three-step procedure as described below, and sums the methylation data within the regions for each sample: "It clusters adjacent CpG sites into genomic regions, following a three-step procedure as described below, and sums the methylation data within the regions for each sample"
- [readme] $ python combine_CpG_sites.py  -v  -o combined.csv  C1.cov  C2.cov  E1.cov  E2.cov: "$ python combine_CpG_sites.py  -v  -o combined.csv  C1.cov  C2.cov  E1.cov  E2.cov"
- [methods] -d <int>    Max. distance between CpG sites (def. 100): "-d <int>    Max. distance between CpG sites (def. 100)"
