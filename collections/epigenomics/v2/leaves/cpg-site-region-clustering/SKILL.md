---
name: cpg-site-region-clustering
description: Use when Bismark alignment outputs (SAM/BAM or *.cov count tables) from
  one or more MethylC-seq samples must be aggregated from individual CpG sites into
  candidate regions prior to DMR testing with findDMRs.r or equivalent beta-binomial
  approaches.
when_to_use_negative:
- The sequencing reads have not yet been aligned and methylation counts extracted
  (run Bismark/Bowtie2 and extract_CpG_data.py first; clustering needs per-CpG count
  tables)
- The analysis requires modeling sets of regions, quantitative traits, or batch effects,
  which DMRfinder does not support
- Deduplicated or strand-aware site-level calls are needed without region aggregation
  (use Bismark's methylation extractor directly instead)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3168
tools:
- name: DMRfinder
  role: Provides combine_CpG_sites.py, the script that clusters adjacent CpG sites
    into genomic regions and sums methylation counts per sample
  repo: https://github.com/jsh58/DMRfinder
- name: extract_CpG_data.py
  role: Upstream DMRfinder script converting Bismark alignment output into sorted,
    cytosine/guanine-merged per-CpG methylated/unmethylated count tables
  repo: https://github.com/jsh58/DMRfinder
- name: Bismark
  role: Aligns bisulfite-treated reads with Bowtie2 and produces the SAM/BAM alignments
    from which CpG counts are extracted; version 0.15.0 recommended due to a v0.16.0
    bug
  repo: https://github.com/FelixKrueger/Bismark
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1186/s12859-017-1909-0
    title: 'DMRfinder: efficiently identifying differentially methylated regions from
      MethylC-seq data'
  source_content_hash: sha256:4256e0594b8d4c4b14720b7350856b5ebfc22520d0e219191323f85ce6c2b591
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jsh58/DMRfinder
  tools:
  - Bismark
  - Bowtie2
  - DMRfinder
  - DSS
  - Python
  - R
  - bsseq
  - combine_CpG_sites.py
  - extract_CpG_data.py
  - findDMRs.r
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.76
n_claims: 25
---

# cpg-site-region-clustering

## Summary

Clusters per-CpG methylation counts from Bismark-aligned MethylC-seq data into genomic regions using DMRfinder's combine_CpG_sites.py, a modified single-linkage clustering that is methylation-blind and therefore unbiased toward finding DMRs before differential testing.

## When to use

Use when Bismark alignment outputs (SAM/BAM or *.cov count tables) from one or more MethylC-seq samples must be aggregated from individual CpG sites into candidate regions prior to DMR testing with findDMRs.r or equivalent beta-binomial approaches.

## When NOT to use

- The sequencing reads have not yet been aligned and methylation counts extracted (run Bismark/Bowtie2 and extract_CpG_data.py first; clustering needs per-CpG count tables)
- The analysis requires modeling sets of regions, quantitative traits, or batch effects, which DMRfinder does not support
- Deduplicated or strand-aware site-level calls are needed without region aggregation (use Bismark's methylation extractor directly instead)

## Inputs

- Bismark SAM/BAM alignment files (with XM methylation strings and XG genome designations) from MethylC-seq samples
- Sorted, merged per-CpG methylated/unmethylated count tables (*.cov) produced by extract_CpG_data.py
- Reference genome index prepared with bismark_genome_preparation

## Outputs

- combined.csv — region-level methylation count table with clustered CpG regions summed per sample
- Regions respecting -c 3 minimum CpGs and -x 500 bp maximum length

## How to apply

Run DMRfinder's extract_CpG_data.py on Bismark SAM/BAM files to obtain sorted, merged cytosine+guanine methylation count tables (*.cov), then apply combine_CpG_sites.py with the article's default parameters: -r 3 (minimum counts per CpG), -s 1 (minimum samples with sufficient counts), -d 100 (maximum CpG-to-CpG distance), -c 3 (minimum CpGs per region), -x 500 (maximum region length), -m 20 (minimum total region counts) to produce combined.csv. The clustering is a modified single-linkage procedure that deliberately ignores methylation levels, so it is not biased for or against finding DMRs; the -x 500 bp cap specifically limits the chaining effect of single-linkage clustering. The resulting region-level count table is then fed to findDMRs.r (beta-binomial hierarchical modeling with Wald tests, -d 0.10 methylation difference, -p 0.05 p-value) for differential testing.

## Related tools

- **DMRfinder** (Provides combine_CpG_sites.py, the script that clusters adjacent CpG sites into genomic regions and sums methylation counts per sample) — https://github.com/jsh58/DMRfinder
- **extract_CpG_data.py** (Upstream DMRfinder script converting Bismark alignment output into sorted, cytosine/guanine-merged per-CpG methylated/unmethylated count tables) — https://github.com/jsh58/DMRfinder
- **Bismark** (Aligns bisulfite-treated reads with Bowtie2 and produces the SAM/BAM alignments from which CpG counts are extracted; version 0.15.0 recommended due to a v0.16.0 bug) — https://github.com/FelixKrueger/Bismark

## Examples

```
python combine_CpG_sites.py -v -o combined.csv C1.cov C2.cov E1.cov E2.cov
```

## Evaluation signals

- combined.csv regions all contain at least 3 CpGs (-c 3) and none exceed 500 bp in length (-x 500)
- Every region in combined.csv spans a maximum CpG-to-CpG distance of 100 bp (-d 100) and has at least 20 total counts (-m 20)
- Counts within a region equal the sum of per-CpG methylated/unmethylated counts across all input *.cov samples
- Downstream findDMRs.r on the replicate self-contrast (IMR90 replicate 1a vs 1b) reports minimal DMRs, confirming the methylation-blind clustering did not inflate false positives
- Output is sorted by chromosome order of the reference sequence names and by 1-based CpG coordinate

## Limitations

- Single-linkage clustering can chain distant CpGs; this is only mitigated by splitting regions longer than 500 bp, so very long methylation domains are fragmented
- Because clustering ignores methylation levels, it cannot be tuned to preferentially find candidate DMRs — a strength for unbiasedness but potentially lower sensitivity than methylation-aware clustering
- DMRfinder's pipeline (including this clustering) does not consider sets of regions or quantitative traits and cannot account for batch effects

## Evidence

- [methods] It clusters adjacent CpG sites into genomic regions, following a three-step procedure as described below, and sums the methylation data within the regions for each sample: "It clusters adjacent CpG sites into genomic regions, following a three-step procedure as described below, and sums the methylation data within the regions for each sample"
- [methods] The DMRfinder script limits this effect by splitting any region whose length exceeds a specified threshold (by default, 500 bp): "The DMRfinder script limits this effect by splitting any region whose length exceeds a specified threshold (by default, 500 bp)"
- [methods] it does not consider methylation levels, and thus is not biased in favor of (or opposed to) finding DMRs: "it does not consider methylation levels, and thus is not biased in favor of (or opposed to) finding DMRs"
- [readme] $ python combine_CpG_sites.py  -v  -o combined.csv  C1.cov  C2.cov  E1.cov  E2.cov: "$ python combine_CpG_sites.py  -v  -o combined.csv  C1.cov  C2.cov  E1.cov  E2.cov"
