---
name: encode-pipeline-execution
description: Use when raw fastq files from RNA:DNA pulldown high-throughput experiments
  (ChIRP-seq, ChOP-seq, CHART-seq, RAP-seq) must be uniformly peak-called and replicate-harmonised
  to build a binding-site benchmark.
when_to_use_negative:
- The experiment lacks downloadable raw data or has unclear replicate annotation —
  these were explicitly excluded from the benchmark
- The goal is de novo triplex prediction rather than building an experimental binding-site
  benchmark (use 3plex directly on FASTA inputs instead)
- The dataset is Xist (X-chromosome-limited data), which was excluded from the collection
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: ENCODE ChIP-seq pipeline
  role: Uniform re-analysis of raw fastq files; peak calling with SPP and replicate
    handling via IDR/Overlap procedures
- name: GEO
  role: Source of raw fastq files for lncRNA:DNA interaction experiments identified
    via manual literature mining
- name: ENA
  role: Alternative source of raw fastq files for the collected RNA:DNA interaction
    experiments
- name: GENCODE version 32
  role: Uniform annotation of RNA molecules (human v32, mouse M25 gene symbols), longest
    transcript per gene
- name: 3plex
  role: 'Downstream consumer: the resulting Top1000 binding peaks serve as the positive
    benchmark set to evaluate 3plex performance'
  repo: https://github.com/molinerisLab/3plex
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2023.05.016
    title: 3plex enables deep computational investigation of triplex forming lncRNAs
  source_content_hash: sha256:cbaf9221566f0f1b381c4871d2e98361d410fac1e5bf2d6a9c7742816d0648d6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/molinerisLab/3plex
  repo_url: https://github.com/molinerisLab/3plex
  tools:
  - 3plex
  - ENA
  - ENCODE ChIP-seq pipeline
  - GENCODE version 32
  - GEO
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# encode-pipeline-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Uniformly re-analyse published experimentally determined lncRNA:DNA binding datasets (ChIRP-seq, ChOP-seq, CHART-seq, RAP-seq) by processing raw fastq files with the ENCODE ChIP-seq pipeline (v1.6.1) and deriving replicate-consistent peak sets via defined peak filtering methods, yielding a benchmark of validated TPX binding sites.

## When to use

Use when raw fastq files from RNA:DNA pulldown high-throughput experiments (ChIRP-seq, ChOP-seq, CHART-seq, RAP-seq) must be uniformly peak-called and replicate-harmonised to build a binding-site benchmark.

## When NOT to use

- The experiment lacks downloadable raw data or has unclear replicate annotation — these were explicitly excluded from the benchmark
- The goal is de novo triplex prediction rather than building an experimental binding-site benchmark (use 3plex directly on FASTA inputs instead)
- The dataset is Xist (X-chromosome-limited data), which was excluded from the collection

## Inputs

- Raw fastq files of lncRNA:DNA pulldown experiments (ChIRP-seq, ChOP-seq, CHART-seq, RAP-seq) downloaded from GEO or ENA
- Even/odd probe set annotations used to define replicate 1 and replicate 2
- GENCODE version 32 / M25 gene symbols annotation (longest transcript per gene)
- Reference genomes for human and mouse

## Outputs

- SPP low-stringency peak calls per dataset
- Replicate-consistent peak sets: IDR conservative, IDR optimal, Overlap conservative (SPP q-value < 0.05), Overlap optimal
- Top1000 peak set per lncRNA
- ENCODE reproducibility classification (ideal / acceptable / concerning) per dataset
- binding_site_peaks.bed with valid BED-formatted genomic intervals for the processed lncRNAs

## How to apply

Query GEO, PubMed and ENA for lncRNA:DNA interaction experiments (2011–2022), downloading raw fastq files while excluding experiments without raw data, with unclear replicate annotation, or limited to Xist. Process each dataset with the ENCODE ChIP-seq pipeline v1.6.1, treating even/odd probe sets as replicate 1 and replicate 2, or intersecting peaks across even/odd sets when multiple pairs exist. Call peaks with SPP at low stringency, then derive replicate-consistent sets using the peak filtering methods: IDR conservative, IDR optimal, Overlap conservative (filtered at SPP q-value < 0.05), and Overlap optimal. Build the Top1000 peak set per lncRNA by starting from IDR conservative peaks, filling up to 1000 with best non-overlapping IDR optimal peaks, then adding best non-overlapping Overlap conservative peaks. Classify each dataset's reproducibility with ENCODE criteria (Self-consistency Ratio max(N1,N2)/min(N1,N2) and Rescue Ratio max(Np,Nt)/min(Np,Nt); 'ideal' if both > 2, 'acceptable' if one < 2, 'concerning' if both < 2). Annotate all RNA molecules uniformly with GENCODE version 32 and M25 gene symbols, taking the longest transcript per gene (lncSmad7 taken from its original publication), and write final peak sets to binding_site_peaks.bed.

## Related tools

- **ENCODE ChIP-seq pipeline** (Uniform re-analysis of raw fastq files; peak calling with SPP and replicate handling via IDR/Overlap procedures)
- **GEO** (Source of raw fastq files for lncRNA:DNA interaction experiments identified via manual literature mining)
- **ENA** (Alternative source of raw fastq files for the collected RNA:DNA interaction experiments)
- **GENCODE version 32** (Uniform annotation of RNA molecules (human v32, mouse M25 gene symbols), longest transcript per gene)
- **3plex** (Downstream consumer: the resulting Top1000 binding peaks serve as the positive benchmark set to evaluate 3plex performance) — https://github.com/molinerisLab/3plex

## Evaluation signals

- Each lncRNA dataset yields peak sets under the applicable filtering methods with an ENCODE reproducibility classification (Self-consistency Ratio and Rescue Ratio) recorded
- binding_site_peaks.bed exists and contains valid BED-formatted genomic intervals for the processed lncRNAs
- Overlap conservative peak sets contain only peaks with SPP q-value < 0.05
- Top1000 sets are built starting from IDR conservative peaks and filled with best non-overlapping IDR optimal, then Overlap conservative peaks
- Peaks are non-overlapping when filling from successive filtering methods (no duplicated intervals within the Top1000 set)

## Limitations

- Only 2 out of 7 lncRNAs in the analysis had a defined set of IDR conservative peaks; the other 5 showed insufficient reproducibility among replicates to apply this method
- Experiments without raw data or with unclear replicate annotation could not be re-analysed and were excluded
- Even/odd probe sets are used as pseudo-replicates, which may not capture true biological replicate variability

## Evidence

- [methods] Uniform re-analysis of pulldown data: "We downloaded raw fastq files from GEO or ENA and analysed them using the ENCODE ChIP-seq pipeline"
- [methods] Peak filtering methods definition: "We defined all these replicate handling procedures as “peak filtering methods”"
- [methods] Overlap conservative SPP q-value threshold: "We additionally filtered the Overlap set by considering only those peaks having a SPP q-value below 0.05"
- [methods] Uniform RNA annotation: "RNA molecules were uniformly annotated using GENCODE version 32 and M25 gene symbols [36]"
- [results] IDR conservative applicability limit: "Only 2 out of 7 lncRNA considered in this analysis had a defined set of IDR conservative peaks, while the other 5 showed insufficient reproducibility"
