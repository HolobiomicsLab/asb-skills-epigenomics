---
name: bigwig-genomic-coverage-extraction
description: Use when you have TCGA or ENCODE ATAC-seq BigWig signal tracks and need
  to quantify or visualize chromatin accessibility over a specific locus (e.g., an
  eccDNA-containing region) in cancer samples.
when_to_use_negative:
- You need eccDNA amplicon reconstruction rather than chromatin accessibility — use
  the AmpliconArchitect WGS pipeline instead
- Raw FASTQ or BAM ATAC-seq alignments are the only available data (no BigWig signal
  track exists), requiring peak calling/coverage computation upstream
- The goal is circular layout visualization of eccDNA relationships rather than linear
  locus-level coverage rendering
edam_operation: http://edamontology.org/operation_3233
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3173
tools:
- name: pyBigWig
  role: Load BigWig files and extract per-base ATAC-seq signal values over the target
    interval
- name: pyGenomeTracks
  role: Render the extracted coverage track over the interval with gene locus annotation
- name: matplotlib
  role: Alternative plotting backend for saving the coverage figure as PNG
- name: AmpliconArchitect
  role: Upstream tool used in eccDNAdb to identify the eccDNA elements whose loci
    are then queried for accessibility
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41388-022-02286-x
    title: 'eccDNAdb: a database of extrachromosomal circular DNA profiles in human
      cancers'
  source_content_hash: sha256:ca7a577dc7023049bcaf2dd4435b2e714f1f3d4c78d9b66b02d8edc29e852aa6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AmpliconArchitect
  - matplotlib
  - pyBigWig
  - pyGenomeTracks
  edam_operation: http://edamontology.org/operation_3233
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# bigwig-genomic-coverage-extraction

## Summary

Extract per-base ATAC-seq signal from BigWig coverage tracks over a defined genomic interval (e.g., chr7:54370001-55996000 at the EGFR locus) using pyBigWig, aggregate per-sample signal across TCGA GBM samples, and render the coverage profile with pyGenomeTracks or matplotlib to assess chromatin accessibility at eccDNA-containing regions.

## When to use

Use when you have TCGA or ENCODE ATAC-seq BigWig signal tracks and need to quantify or visualize chromatin accessibility over a specific locus (e.g., an eccDNA-containing region) in cancer samples.

## When NOT to use

- You need eccDNA amplicon reconstruction rather than chromatin accessibility — use the AmpliconArchitect WGS pipeline instead
- Raw FASTQ or BAM ATAC-seq alignments are the only available data (no BigWig signal track exists), requiring peak calling/coverage computation upstream
- The goal is circular layout visualization of eccDNA relationships rather than linear locus-level coverage rendering

## Inputs

- TCGA GBM ATAC-seq BigWig signal track files (Corces et al. pan-cancer ATAC-seq release, via GDC or UCSC/ENCODE mirrors)
- Genomic interval of interest, e.g. chr7:54370001-55996000 (eccDNA-containing EGFR locus)
- Optional gene annotation for labeling the EGFR locus in the rendered track

## Outputs

- atacseq_coverage_chr7_egfr.tsv — extracted per-base/region ATAC-seq signal values over the queried interval
- atacseq_coverage_chr7_egfr.png — rendered coverage track over chr7:54370001-55996000 with the EGFR gene locus annotated
- Per-sample mean coverage (aggregated accessibility metric) within the region

## How to apply

Download the TCGA glioblastoma ATAC-seq BigWig signal track(s) from the Corces et al. pan-cancer ATAC-seq release via GDC or UCSC/ENCODE mirrors, consistent with eccDNAdb's integration of TCGA ATAC-seq data for exploring eccDNA chromatin accessibility. Load each BigWig with pyBigWig and extract per-base signal values over the region of interest, here chr7:54370001-55996000 spanning the EGFR locus. Aggregate per-sample signal as mean coverage across GBM samples within the region to quantify accessibility. Render the coverage track over the interval using pyGenomeTracks or matplotlib, annotating the EGFR gene locus, and export both the figure and the extracted signal table.

## Related tools

- **pyBigWig** (Load BigWig files and extract per-base ATAC-seq signal values over the target interval)
- **pyGenomeTracks** (Render the extracted coverage track over the interval with gene locus annotation)
- **matplotlib** (Alternative plotting backend for saving the coverage figure as PNG)
- **AmpliconArchitect** (Upstream tool used in eccDNAdb to identify the eccDNA elements whose loci are then queried for accessibility)

## Examples

```
import pyBigWig; bw = pyBigWig.open('TCGA_GBM_ATAC.bw'); vals = bw.values('chr7', 54370001, 55996000)
```

## Evaluation signals

- Coverage profile shows markedly elevated ATAC-seq signal concentrated at the EGFR locus within chr7:54370001-55996000, qualitatively consistent with the high chromatin accessibility reported in Fig. 4E
- Extracted TSV row count/base positions match the queried interval length (55996000 - 54370001 = 1,625,999 bases)
- Aggregated mean coverage across GBM samples is finite and non-zero within the region, and higher at the EGFR locus than in flanking regions
- Rendered PNG includes the EGFR gene annotation and covers exactly the requested coordinates without gaps or null signal blocks

## Limitations

- In eccDNAdb, all eccDNAs were identified by the AmpliconArchitect pipeline only, so locus definitions to query depend on a single detection method
- Qualitative validation only: accessibility is judged visually against the published figure rather than by a statistical test
- Signal quality depends on availability of TCGA ATAC-seq tracks; eccDNAdb notes only ATAC-seq data from TCGA were integrated for chromatin accessibility exploration

## Evidence

- [discussion] Meanwhile, ATAC-seq data from TCGA were integrated for the exploration of eccDNA’s chromatin accessibility.: "Meanwhile, ATAC-seq data from TCGA were integrated for the exploration of eccDNA’s chromatin accessibility."
- [discussion] EccDNA has been reported to exhibit higher chromatin accessibility and active chromatin state than chromosomal DNA.: "EccDNA has been reported to exhibit higher chromatin accessibility and active chromatin state than chromosomal DNA"
- [discussion] AmpliconArchitect is a robust and feasible tool for detecting eccDNA elements from WGS data.: "AmpliconArchitect is a robust and feasible tool for detecting eccDNA elements from WGS data [25]."
- [readme] The source code for the identification of eccDNAs from whole genome sequencing data of human cancers.: "The source code for the identification of eccDNAs from whole genome sequencing data of human cancers and for the annotation of eccDNA-derived genes."
