---
name: eccdna-region-signal-aggregation
description: Use when you need to test whether chromatin is highly accessible at an
  eccDNA-containing region by extracting per-base ATAC-seq signal from BigWig tracks
  over a fixed coordinate interval and aggregating per-sample coverage.
when_to_use_negative:
- You need to detect eccDNAs de novo from WGS data — use AmpliconArchitect (the eccDNAdb
  pipeline) rather than region signal aggregation
- The region of interest lacks a matching ATAC-seq signal track (e.g. no TCGA ATAC-seq
  coverage for the cancer type), so no accessibility comparison is possible
- The goal is gene expression or clinical association rather than chromatin accessibility
  — use the eccDNAdb TCGA/CCLE expression integration instead
edam_operation: http://edamontology.org/operation_3211
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: pyBigWig
  role: Loads BigWig ATAC-seq signal tracks and extracts per-base coverage values
    over the eccDNA region interval
- name: pyGenomeTracks
  role: Renders the ATAC-seq coverage track over the interval with the EGFR gene locus
    annotated
- name: matplotlib
  role: Alternative to pyGenomeTracks for plotting the extracted coverage profile
- name: AmpliconArchitect
  role: Upstream tool that identified the eccDNA elements whose genomic regions are
    queried for accessibility; described as a robust and feasible tool for detecting
    eccDNA from WGS data
  repo: https://github.com/solo7773/eccDNAdb
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41388-022-02286-x
    title: 'eccDNAdb: a database of extrachromosomal circular DNA profiles in human
      cancers'
  source_content_hash: sha256:1a65e3e7cb5678cdd73d593dc7ec8e8a99e3d9c41738326dced111f494301571
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/solo7773/eccDNAdb
  tools:
  - AmpliconArchitect
  - matplotlib
  - pyBigWig
  - pyGenomeTracks
  edam_operation: http://edamontology.org/operation_3211
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# eccdna-region-signal-aggregation

## Summary

Aggregates chromatin accessibility signal (e.g. TCGA ATAC-seq BigWig tracks) over a defined eccDNA-containing genomic region, such as the EGFR locus chr7:54370001-55996000 in glioblastoma, to quantify and visually confirm that eccDNA-mapped loci are highly accessible, as in eccDNAdb's integration of TCGA ATAC-seq.

## When to use

Use when you need to test whether chromatin is highly accessible at an eccDNA-containing region by extracting per-base ATAC-seq signal from BigWig tracks over a fixed coordinate interval and aggregating per-sample coverage.

## When NOT to use

- You need to detect eccDNAs de novo from WGS data — use AmpliconArchitect (the eccDNAdb pipeline) rather than region signal aggregation
- The region of interest lacks a matching ATAC-seq signal track (e.g. no TCGA ATAC-seq coverage for the cancer type), so no accessibility comparison is possible
- The goal is gene expression or clinical association rather than chromatin accessibility — use the eccDNAdb TCGA/CCLE expression integration instead

## Inputs

- TCGA ATAC-seq BigWig signal track(s) (e.g. Corces et al. pan-cancer release, GBM samples)
- Genomic interval of the eccDNA-containing region (e.g. chr7:54370001-55996000, EGFR locus)
- eccDNA coordinates from eccDNAdb (1270 eccDNAs from 480 samples of 42 cancers) or AmpliconArchitect output

## Outputs

- Per-region ATAC-seq signal table (e.g. atacseq_coverage_chr7_egfr.tsv) with per-base or per-sample coverage values
- Rendered coverage track image with gene annotation (e.g. atacseq_coverage_chr7_egfr.png)
- Mean coverage statistic quantifying accessibility over the region

## How to apply

Obtain the pan-cancer ATAC-seq signal tracks (e.g. Corces et al. TCGA GBM ATAC-seq BigWig files from GDC or UCSC/ENCODE mirrors), then load them with pyBigWig and extract per-base values over the target interval — for the EGFR eccDNA locus this is chr7:54370001-55996000. Aggregate per-sample signal as mean coverage across GBM samples within the region to quantify accessibility. Render the coverage track over the interval with pyGenomeTracks or matplotlib, annotating the EGFR gene locus, and save the plot (atacseq_coverage_chr7_egfr.png) plus the extracted signal table (atacseq_coverage_chr7_egfr.tsv). The rationale, grounded in the paper, is that eccDNA exhibits higher chromatin accessibility than chromosomal DNA, so elevated ATAC-seq signal concentrated at the eccDNA-containing locus supports the eccDNA call.

## Related tools

- **pyBigWig** (Loads BigWig ATAC-seq signal tracks and extracts per-base coverage values over the eccDNA region interval)
- **pyGenomeTracks** (Renders the ATAC-seq coverage track over the interval with the EGFR gene locus annotated)
- **matplotlib** (Alternative to pyGenomeTracks for plotting the extracted coverage profile)
- **AmpliconArchitect** (Upstream tool that identified the eccDNA elements whose genomic regions are queried for accessibility; described as a robust and feasible tool for detecting eccDNA from WGS data) — https://github.com/solo7773/eccDNAdb

## Examples

```
import pyBigWig; bw = pyBigWig.open('GBM_ATAC.bw'); vals = bw.values('chr7', 54370000, 55996000); mean_cov = sum(v for v in vals if v is not None)/len(vals)
```

## Evaluation signals

- The extracted TSV contains per-base signal values spanning exactly the queried interval (chr7:54370001-55996000, ~1.63 Mb) with no gaps in coordinates
- Mean coverage across GBM samples within the region is markedly higher than flanking/background regions, consistent with high accessibility
- The rendered PNG shows ATAC-seq signal concentrated at the EGFR locus, qualitatively consistent with Fig. 4E of the paper
- Per-sample aggregation covers the expected number of TCGA GBM ATAC-seq samples without dropped tracks
- Output files match the expected names (atacseq_coverage_chr7_egfr.png, atacseq_coverage_chr7_egfr.tsv)

## Limitations

- Qualitative assessment: validation is that the profile 'shows markedly elevated ATAC-seq signal concentrated at the EGFR locus', not a formal statistical test
- All eccDNAs in eccDNAdb were identified by the AmpliconArchitect pipeline only, so the regions queried inherit that detection bias
- Findings are qualitative browser-track comparisons; no direct threshold-based accessibility cutoff is specified

## Evidence

- [results] Using the genome browser's TCGA ATAC-seq tracks for GBM, the authors found that chromatin is highly accessible at the EGFR gene locus: [evidence span withheld — anchor status: not_found]
- [discussion] ATAC-seq data from TCGA integrated for chromatin accessibility: "Meanwhile, ATAC-seq data from TCGA were integrated for the exploration of eccDNA’s chromatin accessibility."
- [discussion] EccDNA exhibits higher chromatin accessibility than chromosomal DNA: "EccDNA has been reported to exhibit higher chromatin accessibility and active chromatin state than chromosomal DNA"
- [discussion] AmpliconArchitect is a robust and feasible tool for detecting eccDNA elements from WGS data: "AmpliconArchitect is a robust and feasible tool for detecting eccDNA elements from WGS data [25]."
- [abstract] eccDNAdb integrates gene expression, clinical information and chromatin accessibility data: "With the integration of gene expression, clinical information and chromatin accessibility data, eccDNAdb enables users to easily determine the biological function and clinical relevance of eccDNAs in"
