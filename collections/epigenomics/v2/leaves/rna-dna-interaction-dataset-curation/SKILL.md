---
name: rna-dna-interaction-dataset-curation
description: Use when building a gold-standard benchmark of experimentally validated
  lncRNA:DNA binding sites from heterogeneous published ChIRP/ChOP/CHART/RAP-seq studies,
  requiring uniform peak-calling and reproducibility classification.
when_to_use_negative:
- The experiment lacks raw fastq data, has unclear replicate annotation, or is Xist
  (X-chromosome-limited data) — these were explicitly excluded during collection
- The goal is protein-mediated RNA:DNA contacts or TF binding site prediction (the
  authors instead used ENCODE ChIP-seq IDR conservative thresholded peaks and JASPAR
  PWMs for such comparisons)
- 'Only published peak coordinates are needed without uniform re-analysis — note that
  published peak sets underperformed: IDR conservative re-analysis improved average
  AUC by 0.09'
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3524
tools:
- name: 3plex
  role: 'Downstream consumer of the curated benchmark: the Top1000 RNA:DNA interaction
    peaks for the 7 TPX-validated lncRNAs serve as the positive dataset to evaluate
    3plex prediction performance'
  repo: https://github.com/molinerisLab/3plex
- name: GEO
  role: Source database queried for high-throughput RNA:DNA interaction raw fastq
    files
- name: ENA
  role: Source database queried for high-throughput RNA:DNA interaction raw fastq
    files
- name: GENCODE version 32
  role: Uniform RNA annotation reference (version 32 and M25 gene symbols) for the
    collected lncRNAs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2023.05.016
    title: 3plex enables deep computational investigation of triplex forming lncRNAs
  source_content_hash: sha256:6cd067f6aa62a2b6e3797bb939370dc2a36422e69b7da1da4eb2a42e043ba2f3
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
  - GENCODE version 32
  - GEO
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# rna-dna-interaction-dataset-curation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmark-construction skill: systematically collect published experimental lncRNA:DNA binding datasets (ChIRP-seq, ChOP-seq, CHART-seq, RAP-seq; human and mouse, ~800 GB), re-analyse them uniformly with the ENCODE ChIP-seq pipeline, and derive replicate-consistent peak sets to benchmark lncRNA:DNA interaction predictors such as 3plex.

## When to use

Use when building a gold-standard benchmark of experimentally validated lncRNA:DNA binding sites from heterogeneous published ChIRP/ChOP/CHART/RAP-seq studies, requiring uniform peak-calling and reproducibility classification.

## When NOT to use

- The experiment lacks raw fastq data, has unclear replicate annotation, or is Xist (X-chromosome-limited data) — these were explicitly excluded during collection
- The goal is protein-mediated RNA:DNA contacts or TF binding site prediction (the authors instead used ENCODE ChIP-seq IDR conservative thresholded peaks and JASPAR PWMs for such comparisons)
- Only published peak coordinates are needed without uniform re-analysis — note that published peak sets underperformed: IDR conservative re-analysis improved average AUC by 0.09

## Inputs

- Raw fastq files of lncRNA:DNA interaction experiments (ChIRP-seq, ChOP-seq, CHART-seq, RAP-seq) from GEO/ENA/PubMed (2011–2022)
- Probe-set replicate structure (even/odd probe pairs) per experiment
- GENCODE version 32 (human) and M25 (mouse) transcript annotations
- Reference genome assemblies for human and mouse

## Outputs

- binding_site_peaks.bed — uniformly re-analysed peak sets per lncRNA in valid BED format
- Per-dataset peak sets under IDR conservative, IDR optimal, Overlap conservative (SPP q-value < 0.05) and Overlap optimal filtering
- Top1000 peak set per lncRNA (positive dataset for benchmarking)
- ENCODE reproducibility classification per dataset (ideal / acceptable / concerning)

## How to apply

Query GEO, PubMed and ENA for high-throughput lncRNA:DNA interaction experiments from 2011–2022, downloading raw fastq files and excluding datasets lacking raw data, with unclear replicate annotation, or Xist (X-chromosome-limited data). Process each dataset with the ENCODE ChIP-seq pipeline (v1.6.1), treating even/odd probe sets as replicate 1 and replicate 2, calling peaks with SPP at low stringency. Derive replicate-consistent peak sets using four peak filtering methods — IDR conservative, IDR optimal, Overlap conservative (filtered at SPP q-value < 0.05), and Overlap optimal — and build a Top1000 set per lncRNA starting from IDR conservative peaks, filling with best non-overlapping IDR optimal then Overlap conservative peaks. Classify reproducibility with ENCODE criteria using the Self-consistency Ratio max(N1,N2)/min(N1,N2) and Rescue Ratio max(Np,Nt)/min(Np,Nt): 'ideal' if both > 2, 'acceptable' if one < 2, 'concerning' if both < 2. Annotate all RNA molecules uniformly with GENCODE version 32 and M25 gene symbols, taking the longest transcript per gene (lncSmad7 taken from its original publication), and write the final peak sets to binding_site_peaks.bed.

## Related tools

- **3plex** (Downstream consumer of the curated benchmark: the Top1000 RNA:DNA interaction peaks for the 7 TPX-validated lncRNAs serve as the positive dataset to evaluate 3plex prediction performance) — https://github.com/molinerisLab/3plex
- **GEO** (Source database queried for high-throughput RNA:DNA interaction raw fastq files)
- **ENA** (Source database queried for high-throughput RNA:DNA interaction raw fastq files)
- **GENCODE version 32** (Uniform RNA annotation reference (version 32 and M25 gene symbols) for the collected lncRNAs)

## Examples

```
docker run -u `id -u`:`id -g` -it --rm -v $PWD:$PWD imolineris/3plex:v1.0 $PWD/ssRNA.fa $PWD/dsDNA.fa $PWD/results_3plex
```

## Evaluation signals

- Each lncRNA dataset yields peak sets under the applicable filtering methods, with the ENCODE reproducibility classification (Self-consistency and Rescue Ratios) recorded
- binding_site_peaks.bed exists and contains valid BED-formatted genomic intervals for the processed lncRNAs
- 7 lncRNAs with experimentally validated TPX binding mechanisms are identified for benchmarking; only 2 of 7 have defined IDR conservative peaks (the other 5 show insufficient replicate reproducibility)
- Top1000 peak sets contain exactly up to 1000 peaks per lncRNA, built from IDR conservative peaks first, then best non-overlapping IDR optimal, then Overlap conservative peaks
- All RNA sequences are annotated with GENCODE v32/M25 symbols using the longest transcript per gene (lncSmad7 exception from its original publication)

## Limitations

- Only 2 out of 7 lncRNAs had a defined set of IDR conservative peaks; the other 5 showed insufficient reproducibility among replicates to apply this method
- Experiments without raw data or with unclear replicate annotation cannot be re-analysed; Xist data are excluded as X-chromosome-limited
- The IDR conservative filtering, although best performing (+0.09 average AUC), is applicable only to datasets meeting ENCODE reproducibility thresholds

## Evidence
