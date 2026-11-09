---
name: genomic-bin-count-aggregation
description: Use when raw single-cell ChIP-seq/ATAC-seq reads (FASTQ→BAM/BED) or per-cell
  alignment files must be converted into a cell-by-region count matrix binned across
  the genome for downstream clustering.
when_to_use_negative:
- Input is already a normalized feature table or reduced-dimension representation
  (e.g. PCA/t-SNE output) — no bin aggregation is needed.
- The goal is peak-level analysis requiring a user-supplied peak set but no BED peaks
  or BAM files are available (the peak-calling module requires one BAM per sample).
- Data are bulk ChIP-seq reads without single-cell barcodes — the 1600-read-per-cell
  and top-5%-coverage cell filters do not apply.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3173
tools:
- name: ChromSCape
  role: Shiny/R Bioconductor application that creates the count matrix by aggregating
    signal into genomic bins, peaks, or TSS regions and applies QC filtering and normalization
  repo: https://github.com/vallotlab/ChromSCape_benchmarking
- name: R
  role: implementation language for the count-matrix generation and downstream QC/normalization
    steps
- name: Bioconductor
  role: distribution channel for ChromSCape as a Bioconductor package
- name: scChIPseq_DataEngineering pipeline
  role: upstream data-engineering pipeline converting raw paired-end FASTQ to an exploitable
    count matrix (barcode mapping, trimming, genomic mapping, RT/PCR/window duplicate
    removal, counting)
  repo: https://github.com/vallotlab/scChIPseq_DataEngineering
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-19542-x
    title: Interactive analysis of single-cell epigenomic landscapes with ChromSCape
  source_content_hash: sha256:199f24e3efb4f3085e7f564f44edd1f694cc77ac52bc29b857eb88649cb9f230
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/vallotlab/ChromSCape_benchmarking
  tools:
  - Bioconductor
  - ChromSCape
  - R
  - scChIPseq_DataEngineering pipeline
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# genomic-bin-count-aggregation

## Summary

Reconstructs ChromSCape's count-matrix generation from single-cell epigenomic inputs (gzipped single-cell BED/BAM files), aggregating signal into mark-dependent genomic bins (50 kbp for broad marks like H3K27me3, 5 kbp or gene TSS ±2500 bp using Gencode_hg38_v26 for sharp marks), then applying QC cell and region filters and normalizing by total read count and region size.

## When to use

Use when raw single-cell ChIP-seq/ATAC-seq reads (FASTQ→BAM/BED) or per-cell alignment files must be converted into a cell-by-region count matrix binned across the genome for downstream clustering.

## When NOT to use

- Input is already a normalized feature table or reduced-dimension representation (e.g. PCA/t-SNE output) — no bin aggregation is needed.
- The goal is peak-level analysis requiring a user-supplied peak set but no BED peaks or BAM files are available (the peak-calling module requires one BAM per sample).
- Data are bulk ChIP-seq reads without single-cell barcodes — the 1600-read-per-cell and top-5%-coverage cell filters do not apply.

## Inputs

- gzipped single-cell BED files (e.g. from GEO accession GSE105012)
- BAM files with reads labeled by cell barcode ID
- pre-computed count matrix of reads in peaks (e.g. GSE99172)
- reference transcriptome annotation (Gencode_hg38_v26) for TSS-based regions
- user-supplied BED file of peaks (optional, for peak-based counting)

## Outputs

- cell-by-region count matrix (rows = retained regions, columns = retained cells)
- normalized count matrix (count_matrix_normalized.tsv) scaled by total read count and region size

## How to apply

Aggregate per-cell signal into successive genomic bins whose size depends on the histone mark: 50,000 bp bins for broad marks such as H3K27me3, and 5,000 bp bins or regions around gene Transcription Start Sites (Gencode_hg38_v26 coordinates) for sharp marks such as H3K4me3. After counting, apply the QC cell filter — keep cells with a minimum of 1600 unique reads per cell and remove cells in the top 5% coverage percentile to eliminate coverage outliers. Then filter regions: keep regions detected in at least 1% of cells and drop regions not supported by a user-defined percentage of cells with coverage greater than 1000 reads. Finally, normalize the filtered matrix by total read count and region size (e.g. writing count_matrix_normalized.tsv). The rationale is that bulk ChIP-seq tools are unfit for the low coverage and sparsity of single-cell histone modification datasets, so bin aggregation pools sparse per-cell signal while the filters remove low-quality and outlier cells.

## Related tools

- **ChromSCape** (Shiny/R Bioconductor application that creates the count matrix by aggregating signal into genomic bins, peaks, or TSS regions and applies QC filtering and normalization) — https://github.com/vallotlab/ChromSCape_benchmarking
- **R** (implementation language for the count-matrix generation and downstream QC/normalization steps)
- **Bioconductor** (distribution channel for ChromSCape as a Bioconductor package)
- **scChIPseq_DataEngineering pipeline** (upstream data-engineering pipeline converting raw paired-end FASTQ to an exploitable count matrix (barcode mapping, trimming, genomic mapping, RT/PCR/window duplicate removal, counting)) — https://github.com/vallotlab/scChIPseq_DataEngineering

## Examples

```
schip_processing All -f FORWARD.fastq.gz -r REVERSE.fastq.gz -o OUTPUT -c CONFIG
```

## Evaluation signals

- Output matrix rows correspond to regions passing the ≥1%-of-cells detection filter and the user-defined percentage-of-cells-with->1000-reads filter.
- Output matrix columns correspond to cells retained after the 1600-unique-read minimum and top-5%-coverage-removal filters.
- Normalized values scale with each cell's total read count and each region's size (verify by comparing raw vs. normalized entries).
- Bin size matches the mark's biology: 50 kbp bins for H3K27me3, 5 kbp or TSS ±2500 bp (Gencode_hg38_v26) for sharp marks such as H3K4me3.
- Cell barcodes map 1:1 between the raw BED/BAM inputs and the count-matrix columns (no barcode loss after counting).

## Limitations

- Benchmarking comparisons were restricted to a subsampled set of 1287 cells passing QC (and random sub-sampling of 500 MDA-MB-468 cells), so binning/filter parameter behavior on larger datasets is less characterized.
- The benchmark required prior removal of known CNV segments of MDA-MB-468 (77 Mbp) — a manual curation step before count aggregation for some cell lines.
- Most R dependency versions are unreported, limiting environment reproducibility; no versioned archive/Zenodo DOI for the code is provided.
- For peak-based counting, the user must supply a BED peak file and BAM files, which are not deposited for all benchmarking datasets.

## Evidence

- [discussion] Bin-size choice depends on mark breadth: "ChromSCape creates the count matrix by aggregating the signal into successive genomic bins, peaks (BED file must be provided by user) or into regions around genes Transcription Start Sites"
- [discussion] Cell QC threshold and outlier removal: "a minimum coverage of 1600 unique reads per cell, filtering out the cells with the top 5% coverage"
- [discussion] Region filtering rule: "Regions not supported by a minimum user-defined percentage of cells that have a coverage greater than 1000 reads are filtered out"
- [discussion] Normalization after QC: "Post quality control filtering, the matrices are normalized by total read count and region size"
- [discussion] Reference transcriptome for TSS regions: "ChromSCape uses the reference human transcriptome Gencode_hg38_v26"
- [readme] Count matrices in 50kbp bins input to ChromSCape: "Count matrices of each sample in 50kbp bins can be inputted directly in ChromSCape with the option ‘subsample - 500 cells’ to reproduce the analysis."
