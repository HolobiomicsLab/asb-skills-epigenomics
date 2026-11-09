---
name: idr-reproducibility-filtering
description: Use when building a gold-standard set of experimentally determined lncRNA:DNA
  binding sites from raw fastq files with replicate structure (even/odd probe sets),
  where reproducible peaks are needed as positives for prediction benchmarks.
when_to_use_negative:
- The experiment lacks raw fastq data or has unclear replicate annotation — such datasets
  were explicitly excluded from the benchmark collection
- The data are Xist ChIRP-seq (X-chromosome-limited), which the authors excluded from
  the curated collection
- A published peak set is already available and accepted as-is — though the authors
  found IDR conservative filtering gave a +0.09 average AUC gain over published peaks,
  so re-filtering is preferred when raw data exist
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0160
tools:
- name: ENCODE ChIP-seq pipeline
  role: Uniform re-analysis of raw fastq files (version v1.6.1) with even/odd probe
    sets treated as replicates; provides IDR/Overlap peak filtering procedures
- name: SPP
  role: Peak caller used at low stringency; supplies the q-values (< 0.05) used to
    filter Overlap conservative peak sets
- name: GENCODE version 32
  role: Uniform RNA annotation source (M25 gene symbols, longest transcript per gene)
    for the lncRNAs whose peaks are filtered
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2023.05.016
    title: 3plex enables deep computational investigation of triplex forming lncRNAs
  source_content_hash: sha256:40aeb818cb934d7caebdc1f1972b1515ee17f20375854376728ffb8b5f9af3d4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/molinerisLab/3plex
  tools:
  - 3plex
  - ENA
  - ENCODE ChIP-seq pipeline
  - GENCODE version 32
  - GEO
  - SPP
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# idr-reproducibility-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A replicate-handling procedure that converts raw ChIP-seq-style lncRNA:DNA binding data (ChIRP-seq, ChOP-seq, CHART-seq, RAP-seq) into reproducibility-filtered peak sets using ENCODE IDR/Overlap peak filtering methods, providing the experimental benchmark used to evaluate 3plex TPX predictions.

## When to use

Use when building a gold-standard set of experimentally determined lncRNA:DNA binding sites from raw fastq files with replicate structure (even/odd probe sets), where reproducible peaks are needed as positives for prediction benchmarks.

## When NOT to use

- The experiment lacks raw fastq data or has unclear replicate annotation — such datasets were explicitly excluded from the benchmark collection
- The data are Xist ChIRP-seq (X-chromosome-limited), which the authors excluded from the curated collection
- A published peak set is already available and accepted as-is — though the authors found IDR conservative filtering gave a +0.09 average AUC gain over published peaks, so re-filtering is preferred when raw data exist

## Inputs

- Raw fastq files of lncRNA:DNA interaction experiments (ChIRP-seq, ChOP-seq, CHART-seq, RAP-seq) from GEO and ENA (human and mouse, 2011–2022)
- Even/odd probe set replicate annotations per experiment
- GENCODE version 32 / M25 transcript annotations
- lncSmad7 RNA sequence from its original publication

## Outputs

- Replicate-consistent peak sets per lncRNA under IDR conservative, IDR optimal, Overlap conservative (SPP q < 0.05) and Overlap optimal filtering
- Top1000 peak set per lncRNA with non-overlapping fill hierarchy (IDR conservative → IDR optimal → Overlap conservative)
- ENCODE reproducibility classification per dataset (ideal / acceptable / concerning) with Self-consistency and Rescue Ratios
- binding_site_peaks.bed with valid BED-formatted genomic intervals for the processed lncRNAs

## How to apply

Download raw fastq files from GEO or ENA for RNA:DNA interaction experiments published 2011–2022 (excluding datasets without raw data, unclear replicate annotation, or Xist), and analyse them with the ENCODE ChIP-seq pipeline (v1.6.1), treating even/odd probe sets as replicate 1 and 2 (intersecting peaks across each even/odd set when multiple pairs exist). Call peaks with SPP at low stringency, then derive replicate-consistent peak sets via the peak filtering methods: IDR conservative, IDR optimal, and Overlap conservative filtered at SPP q-value < 0.05 (Overlap optimal). Build a Top1000 peak set per lncRNA by starting from IDR conservative peaks, filling up to 1000 with best non-overlapping IDR optimal peaks, then adding best non-overlapping Overlap conservative peaks. Classify each dataset's reproducibility with ENCODE criteria (Self-consistency Ratio max(N1,N2)/min(N1,N2) and Rescue Ratio max(Np,Nt)/min(Np,Nt); 'ideal' if both > 2, 'acceptable' if one < 2, 'concerning' if both < 2), since insufficient reproducibility means IDR filtering cannot be applied. Annotate RNAs uniformly with GENCODE version 32 / M25 gene symbols (longest transcript per gene; lncSmad7 sequence from its original publication) and write final peaks to binding_site_peaks.bed.

## Related tools

- **ENCODE ChIP-seq pipeline** (Uniform re-analysis of raw fastq files (version v1.6.1) with even/odd probe sets treated as replicates; provides IDR/Overlap peak filtering procedures)
- **SPP** (Peak caller used at low stringency; supplies the q-values (< 0.05) used to filter Overlap conservative peak sets)
- **GENCODE version 32** (Uniform RNA annotation source (M25 gene symbols, longest transcript per gene) for the lncRNAs whose peaks are filtered)

## Evaluation signals

- Each lncRNA dataset yields peak sets under the applicable filtering methods, with its ENCODE reproducibility classification (ideal / acceptable / concerning) recorded
- binding_site_peaks.bed exists and contains valid BED-formatted genomic intervals for the processed lncRNAs
- Self-consistency Ratio max(N1,N2)/min(N1,N2) and Rescue Ratio max(Np,Nt)/min(Np,Nt) are computed for every dataset; only datasets with sufficient reproducibility receive IDR conservative peaks (the authors found only 2 of 7 lncRNAs qualified)
- Top1000 sets contain at most 1000 peaks assembled by the non-overlapping fill order: IDR conservative first, then best non-overlapping IDR optimal, then best non-overlapping Overlap conservative
- Overlap peak sets are restricted to peaks with SPP q-value < 0.05

## Limitations

- IDR filtering cannot be applied to datasets with insufficient replicate reproducibility — only 2 out of 7 benchmarked lncRNAs had a defined set of IDR conservative peaks
- Datasets without raw data or with unclear replicate annotation cannot be re-analysed under this scheme
- IDR conservative peak sets may miss true binding sites for poorly reproducible probes; the Top1000 composite set was built as a fallback that mixes filtering methods

## Evidence
