---
name: geodata-fastq-retrieval
description: Use when a methylation-analysis reproduction task requires the raw sequencing
  reads of a published dataset identified by a GEO accession (e.g., GSE129673 for
  TAPS murine ESCs) before alignment or methylation calling can begin.
when_to_use_negative:
- The dataset is already available locally as FASTQ/BAM and only re-analysis (alignment,
  methylation call) is needed — go straight to Msuite.
- The target data are in silico simulated reads (e.g., generated with the SHERMAN
  script for benchmarking) rather than deposited GEO samples.
- The analysis target is a different tissue/sample type not covered by the cited accessions
  (e.g., GSM2191922 murine liver or GSM1545829 murine placenta) without confirming
  those accessions separately.
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3674
tools:
- name: Msuite
  role: Downstream consumer of the retrieved FASTQ files; performs QC, 3-letter/4-letter
    alignment, and methylation call on the murine ESC TAPS/WGBS reads
  repo: https://github.com/hellosunking/Msuite/
- name: bowtie2
  role: Underlying aligner invoked by Msuite on the retrieved FASTQ reads; must be
    installed and in PATH before running the pipeline
  repo: https://github.com/BenLangmead/bowtie2
- name: samtools
  role: Converts the SAM alignments produced from the retrieved reads into BAM format
    (Msuite.final.bam)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2020.100127
    title: 'Msuite: A High-Performance and Versatile DNA Methylation Data-Analysis
      Toolkit'
  source_content_hash: sha256:1e576b11db40cbd315807b2c149e3b391b0a46952b711bd5089e3707b706c1e0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/hellosunking/Msuite
  repo_url: https://github.com/hellosunking/Msuite
  tools:
  - Msuite
  - bowtie2
  - samtools
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3674
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# geodata-fastq-retrieval

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retrieves raw paired-end FASTQ sequencing files for DNA methylation assays (e.g., WGBS and TAPS datasets from murine ESCs, GEO accessions GSE112520 and GSE129673) from GEO so they can serve as real-data inputs to downstream methylation analysis pipelines such as Msuite.

## When to use

Use when a methylation-analysis reproduction task requires the raw sequencing reads of a published dataset identified by a GEO accession (e.g., GSE129673 for TAPS murine ESCs) before alignment or methylation calling can begin.

## When NOT to use

- The dataset is already available locally as FASTQ/BAM and only re-analysis (alignment, methylation call) is needed — go straight to Msuite.
- The target data are in silico simulated reads (e.g., generated with the SHERMAN script for benchmarking) rather than deposited GEO samples.
- The analysis target is a different tissue/sample type not covered by the cited accessions (e.g., GSM2191922 murine liver or GSM1545829 murine placenta) without confirming those accessions separately.

## Inputs

- GEO accession numbers for published methylation datasets (e.g., GSE112520 murine ESC WGBS, GSE129673 murine ESC TAPS)
- Sample-level metadata indicating organism (murine ESCs), library protocol (WGBS/BS-seq vs. TAPS), and read layout (paired-end)

## Outputs

- Paired-end FASTQ files (read 1 and read 2) for each selected sample, ready for alignment with Msuite against GRCm38/mm10 indices

## How to apply

Identify the GEO accession(s) cited in the source article (here, GEO: GSE112520 for murine ESC WGBS and GSE129673 for murine ESC TAPS, both from murine ESCs) and download the paired-end FASTQ files for the relevant samples from GEO. Verify that the downloaded reads correspond to the intended library protocol (BS-seq vs. TAPS) because downstream tooling depends on it — e.g., Msuite requires '-m BS' with 3-letter mode for bisulfite data and supports '-m TAPS' in either 3- or 4-letter mode. Confirm read pairing (read 1 / read 2 files) and organism/reference compatibility (murine ESC data aligned to GRCm38/mm10 indices built with Msuite's build.index.sh). The retrieved FASTQ files then feed directly into the aligner via its '-1 Read1.fq -2 Read2.fq' inputs, with sequencing cycle auto-detected. Correct retrieval is judged by successful index-based alignment producing Msuite.final.bam outputs and report files from which runtime and mapped-read counts can be extracted.

## Related tools

- **Msuite** (Downstream consumer of the retrieved FASTQ files; performs QC, 3-letter/4-letter alignment, and methylation call on the murine ESC TAPS/WGBS reads) — https://github.com/hellosunking/Msuite/
- **bowtie2** (Underlying aligner invoked by Msuite on the retrieved FASTQ reads; must be installed and in PATH before running the pipeline) — https://github.com/BenLangmead/bowtie2
- **samtools** (Converts the SAM alignments produced from the retrieved reads into BAM format (Msuite.final.bam))

## Examples

```
build.index/build.index.sh GRCm38.fa mm10_RefSeq.txt mm10 && msuite -x mm10 -1 TAPS_R1.fq -2 TAPS_R2.fq -4 -m TAPS -p 32 -o taps_4letter_out
```

## Evaluation signals

- Downloaded files are valid paired-end FASTQ (read 1/read 2 pairs present and consistent with the GSE accession's sample layout).
- Protocol attribution is correct: BS-seq samples mapped to WGBS accessions (GSE112520) and TAPS samples to GSE129673, matching the '-m BS' vs '-m TAPS' requirement.
- Msuite runs on the retrieved reads to completion, producing Msuite.final.bam and report files from which runtime and mapped-read counts can be extracted.
- Quality control around TSS (using the mm10 RefSeq annotation shipped with Msuite) shows expected methylation profiles, indicating the correct organism/reference pairing.

## Limitations

- GEO deposition does not guarantee perfect library chemistry: the real TAPS dataset had imperfect chemical treatment, which can affect downstream methylation calls.
- Retrieved reads must match the assay supported by the downstream tool — 4-letter mode in Msuite ONLY supports TAPS/5hmC-CATCH data where non-CpG methylation is very low, so mis-retrieved or mislabeled BS-seq data cannot be processed in 4-letter mode.
- Accession-level metadata may not fully specify sequencing cycle or kit; Msuite auto-detects cycle by default but this should be verified against the FASTQ files.

## Evidence
