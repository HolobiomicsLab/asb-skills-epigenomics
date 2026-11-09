---
name: chromatin-accessibility-visualization-at-egfr-locus
description: Use when you need to test whether a mapped eccDNA/amplified region (e.g.
  chr7:54370001-55996000 at the EGFR locus in GBM) overlaps highly accessible chromatin
  using TCGA ATAC-seq signal tracks.
when_to_use_negative:
- When the input is raw ATAC-seq FASTQ or BAM files requiring peak calling/alignment
  rather than pre-computed BigWig signal tracks
- When the eccDNA of interest lies outside any cancer type covered by TCGA ATAC-seq
  data (eccDNAdb integrates TCGA ATAC-seq only)
- When the goal is eccDNA detection itself from WGS data (use the AmpliconArchitect
  pipeline instead)
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: pyBigWig
  role: Opens TCGA ATAC-seq BigWig signal tracks and extracts per-base coverage values
    over the eccDNA region chr7:54370001-55996000
- name: pyGenomeTracks
  role: Renders the extracted ATAC-seq coverage track over the EGFR locus with gene
    annotation
- name: matplotlib
  role: Alternative renderer for the per-sample aggregated coverage track and locus
    annotation
- name: AmpliconArchitect
  role: Upstream pipeline whose WGS-based eccDNA calls (curated in eccDNAdb) define
    the genomic coordinates queried for accessibility
  repo: github.com/solo7773/eccDNAdb
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41388-022-02286-x
    title: 'eccDNAdb: a database of extrachromosomal circular DNA profiles in human
      cancers'
  source_content_hash: sha256:2925854da719c941cc85be18eb8045ca832bf0634de738e7d3114cdb49fe48ce
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AmpliconArchitect
  - matplotlib
  - pyBigWig
  - pyGenomeTracks
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# chromatin-accessibility-visualization-at-egfr-locus

## Summary

Quantifies and renders chromatin accessibility at an eccDNA-containing locus (e.g. EGFR at chr7:54370001-55996000) by extracting per-base TCGA ATAC-seq BigWig signal with pyBigWig and plotting an annotated coverage track with pyGenomeTracks/matplotlib, as done in eccDNAdb to link eccDNAs to open chromatin in GBM.

## When to use

Use when you need to test whether a mapped eccDNA/amplified region (e.g. chr7:54370001-55996000 at the EGFR locus in GBM) overlaps highly accessible chromatin using TCGA ATAC-seq signal tracks.

## When NOT to use

- When the input is raw ATAC-seq FASTQ or BAM files requiring peak calling/alignment rather than pre-computed BigWig signal tracks
- When the eccDNA of interest lies outside any cancer type covered by TCGA ATAC-seq data (eccDNAdb integrates TCGA ATAC-seq only)
- When the goal is eccDNA detection itself from WGS data (use the AmpliconArchitect pipeline instead)

## Inputs

- TCGA pan-cancer ATAC-seq per-sample BigWig signal tracks (Corces et al. release, obtained via GDC or UCSC/ENCODE mirrors)
- eccDNA/amplified region coordinates from eccDNAdb (e.g. chr7:54370001-55996000 at the EGFR locus in GBM)
- Gene locus annotation for labeling (EGFR gene model)

## Outputs

- Per-region ATAC-seq signal table (e.g. atacseq_coverage_chr7_egfr.tsv) with per-base or per-sample coverage over chr7:54370001-55996000
- Rendered, gene-annotated coverage track image (e.g. atacseq_coverage_chr7_egfr.png)
- Aggregate accessibility metric (mean per-sample coverage over the eccDNA region)

## How to apply

Download the TCGA pan-cancer ATAC-seq BigWig signal track(s) for the cancer type of interest (e.g. GBM) from the Corces et al. release via GDC or UCSC/ENCODE mirrors. Open each BigWig with pyBigWig and extract per-base signal values over the exact eccDNA coordinates (here chr7:54370001-55996000). Aggregate per-sample signal as mean coverage across GBM samples within the region to quantify accessibility at the locus. Render the coverage track over the region with pyGenomeTracks or matplotlib, annotating the gene locus (EGFR), and export both the figure and the extracted signal as TSV. Validation is qualitative: elevated, concentrated ATAC-seq signal at the gene locus within the region, consistent with the paper's Fig. 4E.

## Related tools

- **pyBigWig** (Opens TCGA ATAC-seq BigWig signal tracks and extracts per-base coverage values over the eccDNA region chr7:54370001-55996000)
- **pyGenomeTracks** (Renders the extracted ATAC-seq coverage track over the EGFR locus with gene annotation)
- **matplotlib** (Alternative renderer for the per-sample aggregated coverage track and locus annotation)
- **AmpliconArchitect** (Upstream pipeline whose WGS-based eccDNA calls (curated in eccDNAdb) define the genomic coordinates queried for accessibility) — github.com/solo7773/eccDNAdb

## Examples

```
import pyBigWig; bw = pyBigWig.open('TCGA-GBM_ATAC.bigWig'); vals = bw.values('chr7', 54370001, 55996000); mean_cov = sum(v for v in vals if v is not None)/len(vals)
```

## Evaluation signals

- Extracted TSV spans exactly chr7:54370001-55996000 with one signal value per base (1,626,000 positions per sample)
- Aggregated mean coverage across GBM samples is markedly elevated at the EGFR locus relative to flanking sequence within the region
- Rendered PNG shows the coverage track with a labeled EGFR gene annotation and the correct coordinates on the axis
- Profile is qualitatively consistent with the high chromatin accessibility reported at the EGFR locus in Fig. 4E of the eccDNAdb paper
- BigWig query returns non-missing values (NaN-free) for the TCGA GBM sample tracks

## Limitations

- eccDNAdb's chromatin accessibility layer is restricted to TCGA ATAC-seq data; non-TCGA tumors or tissue types are not covered
- All eccDNAs in the database were identified by the AmpliconArchitect pipeline only, so queried coordinates inherit that pipeline's biases
- Validation of accessibility at the locus is qualitative (visual comparison to Fig. 4E), not a formal statistical test

## Evidence

- [discussion] Meanwhile, ATAC-seq data from TCGA were integrated for the exploration of eccDNA's chromatin accessibility.: "Meanwhile, ATAC-seq data from TCGA were integrated for the exploration of eccDNA’s chromatin accessibility."
- [discussion] EccDNA higher accessibility quote: "EccDNA has been reported to exhibit higher chromatin accessibility and active chromatin state than chromosomal DNA"
- [other] Task workflow quote on BigWig extraction: [evidence span withheld — anchor status: not_found]
- [other] Task workflow quote on Corces release: [evidence span withheld — anchor status: not_found]
- [readme] README repo description: "The source code for the identification of eccDNAs from whole genome sequencing data of human cancers and for the annotation of eccDNA-derived genes."
