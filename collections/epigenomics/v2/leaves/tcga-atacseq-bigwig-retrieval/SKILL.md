---
name: tcga-atacseq-bigwig-retrieval
description: Use when a candidate eccDNA region from whole-genome sequencing (e.g.
  an AmpliconArchitect-called amplicon) needs chromatin accessibility validation using
  the TCGA pan-cancer ATAC-seq BigWig signal tracks.
when_to_use_negative:
- No TCGA ATAC-seq data are available for the cancer type of the eccDNA call (ATAC-seq
  coverage in TCGA is cancer-type dependent)
- The goal is eccDNA detection/amplicon reconstruction from WGS data itself — use
  the AmpliconArchitect pipeline instead; this skill only validates accessibility
  of already-called regions
- The candidate region is tiny (e.g. short amplicon) where per-base ATAC-seq signal
  is too sparse/noisy to interpret as accessibility
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0654
tools:
- name: pyBigWig
  role: Opens TCGA ATAC-seq BigWig tracks and extracts per-base signal values over
    the eccDNA region
- name: pyGenomeTracks
  role: Renders the extracted coverage track over the region with locus annotation
- name: matplotlib
  role: Alternative plotting backend for rendering the ATAC-seq coverage track image
- name: AmpliconArchitect
  role: Upstream tool whose eccDNA/amplicon calls define the genomic regions queried
    against the ATAC-seq tracks
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41388-022-02286-x
    title: 'eccDNAdb: a database of extrachromosomal circular DNA profiles in human
      cancers'
  source_content_hash: sha256:dc815b004a8835ba9f3ce50e74f33ee8c38fd05f16f1230aa9c59552cb94763f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AmpliconArchitect
  - matplotlib
  - pyBigWig
  - pyGenomeTracks
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# tcga-atacseq-bigwig-retrieval

## Summary

Retrieves and renders TCGA ATAC-seq BigWig coverage tracks over candidate eccDNA genomic regions to test whether eccDNA-containing loci (e.g. EGFR at chr7:54370001-55996000 in GBM) show elevated chromatin accessibility, as done in eccDNAdb's chromatin accessibility integration.

## When to use

Use when a candidate eccDNA region from whole-genome sequencing (e.g. an AmpliconArchitect-called amplicon) needs chromatin accessibility validation using the TCGA pan-cancer ATAC-seq BigWig signal tracks.

## When NOT to use

- No TCGA ATAC-seq data are available for the cancer type of the eccDNA call (ATAC-seq coverage in TCGA is cancer-type dependent)
- The goal is eccDNA detection/amplicon reconstruction from WGS data itself — use the AmpliconArchitect pipeline instead; this skill only validates accessibility of already-called regions
- The candidate region is tiny (e.g. short amplicon) where per-base ATAC-seq signal is too sparse/noisy to interpret as accessibility

## Inputs

- TCGA pan-cancer ATAC-seq BigWig signal track(s) for the cancer of interest (e.g. GBM, from the Corces et al. release via GDC or UCSC/ENCODE mirrors)
- Candidate eccDNA region coordinates (chromosome:start-end) from WGS/AmpliconArchitect analysis, e.g. chr7:54370001-55996000
- Gene locus annotation for labeling (e.g. EGFR)

## Outputs

- Rendered coverage track image over the eccDNA region (e.g. atacseq_coverage_chr7_egfr.png)
- Extracted per-base region signal table (e.g. atacseq_coverage_chr7_egfr.tsv)
- Aggregate accessibility metric (mean ATAC-seq coverage) for the eccDNA-containing region

## How to apply

Download the TCGA glioblastoma (or relevant cancer type) ATAC-seq BigWig signal tracks from the Corces et al. pan-cancer ATAC-seq release (via GDC or UCSC/ENCODE mirrors), matching samples to the cancer type of the eccDNA call. Open each BigWig with pyBigWig and extract per-base signal values over the eccDNA-containing coordinates (e.g. chr7:54370001-55996000 covering the EGFR locus). Aggregate per-sample mean coverage across GBM samples to quantify accessibility within the region, then render the aggregated coverage track over the region with pyGenomeTracks or matplotlib, annotating the eccDNA gene locus. Validate by confirming the coverage profile shows markedly elevated ATAC-seq signal concentrated at the eccDNA locus, qualitatively consistent with the high chromatin accessibility reported for eccDNA loci (e.g. Fig. 4E of the eccDNAdb paper).

## Related tools

- **pyBigWig** (Opens TCGA ATAC-seq BigWig tracks and extracts per-base signal values over the eccDNA region)
- **pyGenomeTracks** (Renders the extracted coverage track over the region with locus annotation)
- **matplotlib** (Alternative plotting backend for rendering the ATAC-seq coverage track image)
- **AmpliconArchitect** (Upstream tool whose eccDNA/amplicon calls define the genomic regions queried against the ATAC-seq tracks)

## Examples

```
import pyBigWig; bw = pyBigWig.open('TCGA_GBM_ATACseq.bigWig'); vals = bw.values('chr7', 54370001, 55996000)
```

## Evaluation signals

- Coverage profile shows markedly elevated ATAC-seq signal concentrated at the eccDNA-containing gene locus (e.g. EGFR) within the queried region, consistent with Fig. 4E
- Extracted TSV contains one signal value per queried base/interval over chr7:54370001-55996000 with matching coordinates to the input region
- Per-sample aggregation yields finite mean coverage values (no NaNs) across the GBM ATAC-seq samples queried
- Rendered PNG displays the correct genomic interval and the annotated gene locus label

## Limitations

- ATAC-seq integration in eccDNAdb is limited to the cancer types covered by the TCGA ATAC-seq release (e.g. GBM); other cancers may lack signal tracks
- Accessibility assessment is qualitative — signal must be markedly elevated rather than compared against formal statistical thresholds
- eccDNAs in the database were all identified via the AmpliconArchitect pipeline only, so regions queried against ATAC-seq inherit that detection bias

## Evidence

- [discussion] Meanwhile, ATAC-seq data from TCGA were integrated for the exploration of eccDNA’s chromatin accessibility.: "Meanwhile, ATAC-seq data from TCGA were integrated for the exploration of eccDNA’s chromatin accessibility."
- [discussion] EccDNA has been reported to exhibit higher chromatin accessibility and active chromatin state than chromosomal DNA: "EccDNA has been reported to exhibit higher chromatin accessibility and active chromatin state than chromosomal DNA"
- [readme] The source code for the identification of eccDNAs from whole genome sequencing data of human cancers and for the annotation of eccDNA-derived genes.: "The source code for the identification of eccDNAs from whole genome sequencing data of human cancers and for the annotation of eccDNA-derived genes."
- [abstract] gene expression, clinical information and chromatin accessibility data: "With the integration of gene expression, clinical information and chromatin accessibility data, eccDNAdb enables users to easily determine the biological function and clinical relevance of eccDNAs in"
