---
name: chip-seq-peak-calling-replicate-handling
description: Use when raw sequencing data of lncRNA:DNA capture experiments (ChIRP-seq,
  ChOP-seq, CHART-seq, RAP-seq) from GEO/ENA must be uniformly re-processed and replicate-inconsistent
  peaks reconciled into benchmark BED peak sets.
when_to_use_negative:
- The experiment lacks raw fastq data or has unclear replicate annotation — such datasets
  were excluded from the collection
- The data are protein-mediated DNA:RNA interactions or TF ChIP-seq unrelated to benchmarking
  lncRNA:DNA triplex predictions (e.g. the JASPAR TF PWM comparison task)
- Only one replicate or probe set exists, making the IDR/Overlap replicate-handling
  procedures inapplicable
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0091
tools:
- name: ENCODE ChIP-seq pipeline
  role: Uniform re-analysis pipeline (v1.6.1) applied to raw fastq files, with even/odd
    probe sets treated as replicates
- name: SPP
  role: Peak caller used at low stringency to generate peaks subsequently filtered
    by the replicate-handling (peak filtering) procedures
- name: GEO
  role: Source database queried for high-throughput RNA:DNA interaction raw fastq
    files (2011–2022)
- name: ENA
  role: Alternative source database for downloading raw fastq files
- name: 3plex
  role: Downstream consumer of the resulting Top1000/binding_site_peaks.bed benchmark
    peak sets for performance evaluation
  repo: https://github.com/molinerisLab/3plex
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2023.05.016
    title: 3plex enables deep computational investigation of triplex forming lncRNAs
  source_content_hash: sha256:899f08bd19f438bcdeba06ed5b77ca6ed323c9d6250ac4f5eef6ea6e6cdb4861
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
  - GEO
  - SPP
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# chip-seq-peak-calling-replicate-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A method for uniformly re-analysing published experimental lncRNA:DNA binding data (ChIRP-seq, ChOP-seq, CHART-seq, RAP-seq) with the ENCODE ChIP-seq pipeline and reconciling probe-set replicates into benchmark peak sets (IDR, Overlap, Top1000) for tool evaluation.

## When to use

Use when raw sequencing data of lncRNA:DNA capture experiments (ChIRP-seq, ChOP-seq, CHART-seq, RAP-seq) from GEO/ENA must be uniformly re-processed and replicate-inconsistent peaks reconciled into benchmark BED peak sets.

## When NOT to use

- The experiment lacks raw fastq data or has unclear replicate annotation — such datasets were excluded from the collection
- The data are protein-mediated DNA:RNA interactions or TF ChIP-seq unrelated to benchmarking lncRNA:DNA triplex predictions (e.g. the JASPAR TF PWM comparison task)
- Only one replicate or probe set exists, making the IDR/Overlap replicate-handling procedures inapplicable

## Inputs

- Raw FASTQ files of lncRNA:DNA interaction experiments (ChIRP-seq, ChOP-seq, CHART-seq, RAP-seq) downloaded from GEO or ENA
- Even/odd probe set annotations defining replicate 1 and replicate 2 per experiment
- GENCODE version 32 and M25 gene symbol annotations (longest transcript per gene; lncSmad7 sequence from its original publication)

## Outputs

- binding_site_peaks.bed containing replicate-consistent, BED-formatted genomic peak intervals for each processed lncRNA
- Per-dataset peak sets under IDR conservative, IDR optimal, Overlap conservative (SPP q < 0.05) and Overlap optimal filtering
- ENCODE reproducibility classification (ideal / acceptable / concerning) per dataset based on Self-consistency Ratio and Rescue Ratio
- Top1000 peak set per lncRNA (positive benchmark dataset for 7 TPX-validated lncRNAs)

## How to apply

Query GEO, PubMed and ENA for high-throughput lncRNA:DNA interaction experiments (2011–2022), downloading raw fastq files and excluding experiments without raw data, unclear replicate annotation, or Xist (X-chromosome-limited data). Process each dataset with the ENCODE ChIP-seq pipeline (v1.6.1), treating even/odd probe sets as replicate 1 and replicate 2 (or intersecting peaks across sets when multiple pairs exist), and call peaks with SPP at low stringency. Derive replicate-consistent peak sets using the 'peak filtering methods': IDR conservative, IDR optimal, Overlap conservative filtered at SPP q-value < 0.05, and Overlap optimal. Build the Top1000 peak set per lncRNA by starting from IDR conservative peaks, filling up to 1000 with best non-overlapping IDR optimal peaks, then adding best non-overlapping Overlap conservative peaks. Classify each dataset's reproducibility with the ENCODE criteria (Self-consistency Ratio max(N1,N2)/min(N1,N2) and Rescue Ratio max(Np,Nt)/min(Np,Nt): 'ideal' if both > 2, 'acceptable' if one < 2, 'concerning' if both < 2). Annotate all RNA molecules uniformly with GENCODE version 32 and M25 gene symbols, taking the longest transcript per gene, and write the final peak sets to binding_site_peaks.bed.

## Related tools

- **ENCODE ChIP-seq pipeline** (Uniform re-analysis pipeline (v1.6.1) applied to raw fastq files, with even/odd probe sets treated as replicates)
- **SPP** (Peak caller used at low stringency to generate peaks subsequently filtered by the replicate-handling (peak filtering) procedures)
- **GEO** (Source database queried for high-throughput RNA:DNA interaction raw fastq files (2011–2022))
- **ENA** (Alternative source database for downloading raw fastq files)
- **3plex** (Downstream consumer of the resulting Top1000/binding_site_peaks.bed benchmark peak sets for performance evaluation) — https://github.com/molinerisLab/3plex

## Evaluation signals

- Each lncRNA dataset yields peak sets under the applicable filtering methods with its ENCODE reproducibility classification (Self-consistency Ratio and Rescue Ratio) recorded
- binding_site_peaks.bed exists and contains valid BED-formatted genomic intervals for all processed lncRNAs
- Overlap conservative peak sets contain only peaks with SPP q-value below 0.05
- Top1000 sets are constructed in the prescribed priority order (IDR conservative first, then IDR optimal, then Overlap conservative) with no overlapping peaks
- Benchmark quality: IDR conservative peaks produced an average AUC increase of 0.09 versus published peak sets when used as positives for 3plex evaluation

## Limitations

- Only 2 out of 7 lncRNAs had a defined set of IDR conservative peaks; the other 5 showed insufficient reproducibility among replicates to apply this method
- Datasets without raw data, with unclear replicate annotation, or limited to Xist (X-chromosome) were excluded
- Reproducibility is often 'acceptable' or 'concerning' rather than 'ideal' (both ratios > 2), so fallback filtering methods must be used per dataset

## Evidence

- [methods] We downloaded raw fastq files from GEO or ENA and analysed them using the ENCODE ChIP-seq pipeline: "We downloaded raw fastq files from GEO or ENA and analysed them using the ENCODE ChIP-seq pipeline"
- [methods] peak filtering methods definition: "We defined all these replicate handling procedures as “peak filtering methods”"
- [methods] SPP q-value filter: "We additionally filtered the Overlap set by considering only those peaks having a SPP q-value below 0.05"
- [methods] GENCODE annotation: "RNA molecules were uniformly annotated using GENCODE version 32 and M25 gene symbols [36]"
- [results] IDR conservative best method: "The best method was the IDR conservative one, producing an increase in average AUC of 0.09 compared with published peaks sets."
- [results] limited IDR conservative applicability: "Only 2 out of 7 lncRNA considered in this analysis had a defined set of IDR conservative peaks, while the other 5 showed insufficient reproducibility"
