---
name: single-cell-epigenomic-data-ingestion
description: Use when raw single-cell epigenomic data (paired-end FASTQ or per-cell
  BED files from GEO accessions like GSE124680, GSE124690, GSE105012) must be converted
  into count matrices for ChromSCape-based subpopulation analysis.
when_to_use_negative:
- Input is already a count matrix or feature table — skip ingestion and load directly
  into ChromSCape
- Data are single-cell RNA-seq or other non-chromatin modalities; existing bulk ChIP-seq
  tools are also not fit for the low coverage and sparsity of single-cell histone
  modification datasets
- Peak calling is required but only BED files (no BAM) are available — the ChromSCape
  peak calling module requires user-supplied BAM files with barcode-labeled reads
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_3169
tools:
- name: ChromSCape
  role: Shiny/R Bioconductor app that builds count matrices from BED files by aggregating
    signal into bins, peaks, or TSS regions, then applies QC filtering
  repo: https://github.com/vallotlab/ChromSCape_benchmarking
- name: scChIPseq_DataEngineering pipeline
  role: Data engineering pipeline converting raw paired-end FASTQ (barcode mapping,
    trimming, genomic mapping, RT/PCR/window duplicate removal) into exploitable count
    matrices
  repo: https://github.com/vallotlab/scChIPseq_DataEngineering
- name: R / Bioconductor / Shiny
  role: Runtime environment in which ChromSCape count-matrix generation and filtering
    are executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-19542-x
    title: Interactive analysis of single-cell epigenomic landscapes with ChromSCape
  source_content_hash: sha256:be5a7651295ce249683f7f9952f2f4a514da0f424939286e0e287af3833c8aef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/vallotlab/ChromSCape_benchmarking
  tools:
  - Bioconductor
  - ChromSCape
  - R
  - R / Bioconductor / Shiny
  - Shiny
  - scChIPseq_DataEngineering pipeline
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# single-cell-epigenomic-data-ingestion

## Summary

Ingest public single-cell histone modification datasets (scCUT&Tag, scChIC-seq, scChIP-seq) as gzipped single-cell BED files from GEO into ChromSCape, building count matrices by aggregating signal around gene TSS (±2500bp) or into successive 50kbp genomic bins before downstream clustering.

## When to use

Use when raw single-cell epigenomic data (paired-end FASTQ or per-cell BED files from GEO accessions like GSE124680, GSE124690, GSE105012) must be converted into count matrices for ChromSCape-based subpopulation analysis.

## When NOT to use

- Input is already a count matrix or feature table — skip ingestion and load directly into ChromSCape
- Data are single-cell RNA-seq or other non-chromatin modalities; existing bulk ChIP-seq tools are also not fit for the low coverage and sparsity of single-cell histone modification datasets
- Peak calling is required but only BED files (no BAM) are available — the ChromSCape peak calling module requires user-supplied BAM files with barcode-labeled reads

## Inputs

- Gzipped single-cell BED files from GEO (e.g. GSE124680 n=908 K562 cells, GSE124690 n=479 K562 + n=486 H1 cells, GSE105012 H3K4me3 scChIC-seq)
- Paired-end FASTQ files for the scChIPseq_DataEngineering raw-read pipeline
- Reference transcriptome Gencode_hg38_v26 for TSS-based aggregation
- Optional user-provided BED regions to exclude (e.g. CNV regions like MM468_identified_CNA.bed)

## Outputs

- Cell x region count matrix (TSS ±2500bp features or 50kbp bins)
- QC-filtered, normalized count matrix (normalized by total read count and region size)
- Coverage files (bigwig) and pipeline reports when using the FASTQ pipeline

## How to apply

For raw reads, run the scChIPseq_DataEngineering pipeline (barcode mapping, trimming, genomic mapping, RT/PCR/window duplicate removal, counting) to go from FASTQ to count matrix. For published gzipped single-cell BED files, load them directly into ChromSCape (R/Bioconductor Shiny app) and choose the aggregation feature: regions around gene TSS ±2500bp using Gencode_hg38_v26 for H3K27me3 scCUT&Tag data, or successive 50kbp genomic bins for H3K4me3 scChIC-seq data. Apply QC filtering inside ChromSCape: minimum coverage of 1600 unique reads per cell, removal of cells in the top 5% coverage, and keep regions detected in at least 1% of cells (regions also require a user-defined percentage of cells with coverage >1000 reads). The rationale is that single-cell histone modification data is sparse and low-coverage, so aggressive per-cell and per-region thresholds are needed before normalization, PCA (first 50 PCs with centering), and optional FastMNN batch correction.

## Related tools

- **ChromSCape** (Shiny/R Bioconductor app that builds count matrices from BED files by aggregating signal into bins, peaks, or TSS regions, then applies QC filtering) — https://github.com/vallotlab/ChromSCape_benchmarking
- **scChIPseq_DataEngineering pipeline** (Data engineering pipeline converting raw paired-end FASTQ (barcode mapping, trimming, genomic mapping, RT/PCR/window duplicate removal) into exploitable count matrices) — https://github.com/vallotlab/scChIPseq_DataEngineering
- **R / Bioconductor / Shiny** (Runtime environment in which ChromSCape count-matrix generation and filtering are executed)

## Examples

```
schip_processing All -f sample_R1.fastq.gz -r sample_R2.fastq.gz -c config.yml -o output/ -n sample
```

## Evaluation signals

- Every retained cell passes the minimum coverage threshold of 1600 unique reads and no cell in the top 5% coverage survives filtering
- Retained regions are detected in at least 1% of cells and supported by the user-defined percentage of cells with coverage >1000 reads
- Count matrix features match the chosen aggregation: genes (TSS ±2500bp, Gencode_hg38_v26) for scCUT&Tag H3K27me3 or successive 50kbp bins for scChIC-seq H3K4me3
- Downstream reproducibility check: K562 vs H1 scCUT&Tag clustering gives ARI = 0.976 and the scChIC-seq UMAP shows 7 clusters
- FASTQ pipeline outputs all expected artifacts: count matrix, bigwig coverage file, and reporting step completed without dropped barcodes

## Limitations

- Benchmark comparison restricted to a subsampled set of 1287 cells and random sub-sampling of 500 MDA-MB-468 cells, limiting scope
- Peak calling module requires user-supplied BAM files, which are not deposited for all benchmarking datasets
- MACS2 version not specified despite being an external requirement for the peak calling module; most R dependency versions unreported, limiting environment reproducibility
- No versioned release/archive (e.g. Zenodo DOI) for ChromSCape code; the demonstration app is hosted on third-party shinyapps.io with no long-term availability guarantee

## Evidence

- [discussion] ChromSCape creates the count matrix by aggregating the signal into successive genomic bins, peaks (BED file must be provided by user) or into regions around genes Transcription Start Sites: "ChromSCape creates the count matrix by aggregating the signal into successive genomic bins, peaks (BED file must be provided by user) or into regions around genes Transcription Start Sites"
- [discussion] a minimum coverage of 1600 unique reads per cell, filtering out the cells with the top 5% coverage: "a minimum coverage of 1600 unique reads per cell, filtering out the cells with the top 5% coverage"
- [discussion] Regions not supported by a minimum user-defined percentage of cells that have a coverage greater than 1000 reads are filtered out: "Regions not supported by a minimum user-defined percentage of cells that have a coverage greater than 1000 reads are filtered out"
- [readme] The multiple steps involved in the pipeline are :: [evidence span withheld — anchor status: external_doc]
