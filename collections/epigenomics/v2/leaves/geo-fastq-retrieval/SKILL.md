---
name: geo-fastq-retrieval
description: Use when an analysis starts from a published GEO accession (e.g. GSE136050)
  and requires the raw FASTQ files of ChIP-seq IP samples to reproduce published peak
  counts such as 506 LuxR or 237/302 H-NS peaks.
when_to_use_negative:
- Processed data (BAM, bigWig, or peak calls) are already deposited and sufficient
  for the analysis goal
- The target dataset is RNA-seq deposited under a different accession (GSE136191)
  and the goal is expression quantification rather than ChIP peak calling
- No network access to NCBI GEO/SRA is available
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0769
tools:
- name: SRA toolkit
  role: Fetches raw ChIP-seq FASTQ files from the GSE136050 accession for trimming,
    alignment, and peak calling
- name: Trimmomatic
  role: Receives the downloaded FASTQ files and trims adapters and low-quality bases
    as the next workflow step
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkz1089
    title: Global H-NS counter-silencing by LuxR activates quorum sensing gene expression
  source_content_hash: sha256:9f3ede906e69b6ab26f7f758f341c65d4a6a2eccb6f574e90939fbd273918935
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/Juliacvk/LuxR_H-NS
  tools:
  - JBrowse
  - SRA toolkit
  - Trimmomatic
  - bwa
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# geo-fastq-retrieval

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Download raw ChIP-seq (and RNA-seq) FASTQ files from NCBI GEO/SRA accessions — here GSE136050 for LuxR and H-NS immunoprecipitation samples in V. harveyi — as the entry point for a downstream alignment and peak-calling workflow.

## When to use

Use when an analysis starts from a published GEO accession (e.g. GSE136050) and requires the raw FASTQ files of ChIP-seq IP samples to reproduce published peak counts such as 506 LuxR or 237/302 H-NS peaks.

## When NOT to use

- Processed data (BAM, bigWig, or peak calls) are already deposited and sufficient for the analysis goal
- The target dataset is RNA-seq deposited under a different accession (GSE136191) and the goal is expression quantification rather than ChIP peak calling
- No network access to NCBI GEO/SRA is available

## Inputs

- NCBI GEO accession (GSE136050 for ChIP-seq; GSE136191 for RNA-seq)
- SRA run metadata for LuxR and H-NS immunoprecipitation samples in wild-type and ΔluxR backgrounds
- SRA toolkit (prefetch/fasterq-dump) installation

## Outputs

- Per-sample FASTQ files for LuxR and H-NS ChIP-seq IP samples
- Sample-to-condition mapping (wild-type vs ΔluxR) for downstream peak calling

## How to apply

Resolve the GEO accession (GSE136050 for ChIP-seq data in this study) to its linked SRA runs and fetch FASTQ files with the SRA toolkit, keeping separate samples for LuxR IP, H-NS IP in wild-type, and H-NS IP in the ΔluxR background so downstream conditions remain distinguishable. Validate each download by checking read counts and FASTQ integrity before handing off to Trimmomatic adapter/quality trimming. The retrieved files map to the V. harveyi ATCC BAA-1116 reference genome with bwa and feed macs2 peak calling, whose per-condition peak counts (506 LuxR, 237 H-NS wild-type, 302 H-NS ΔluxR) serve as acceptance criteria for the whole pipeline. Note the paper also deposits RNA-seq data under a separate accession (GSE136191), so pick the correct accession for the data type at hand.

## Related tools

- **SRA toolkit** (Fetches raw ChIP-seq FASTQ files from the GSE136050 accession for trimming, alignment, and peak calling)
- **Trimmomatic** (Receives the downloaded FASTQ files and trims adapters and low-quality bases as the next workflow step)

## Examples

```
prefetch GSE136050; fasterq-dump <SRR_run> --split-files -O fastq/
```

## Evaluation signals

- Each expected sample from GSE136050 yields a non-empty FASTQ file with plausible read counts
- Sample names/conditions (LuxR IP, H-NS IP wild-type, H-NS IP ΔluxR) map unambiguously to downloaded files
- Downstream bwa+macs2 pipeline on the retrieved files reproduces published counts: 506 LuxR peaks, 237 H-NS peaks in wild-type, and 302 H-NS peaks in ΔluxR
- FASTQ files pass integrity checks (no truncation) before Trimmomatic trimming

## Limitations

- Only the accessions named in the paper (GSE136050 ChIP-seq, GSE136191 RNA-seq) are supported by the published metadata; other experiments require their own accessions
- Raw FASTQ retrieval does not itself validate sample quality — errors only surface downstream during alignment and peak calling

## Evidence

- [other] Download the ChIP-seq FASTQ files for the LuxR and H-NS immunoprecipitation samples from NCBI GEO accession GSE136050 using the SRA toolkit.: [evidence span withheld — anchor status: not_found]
