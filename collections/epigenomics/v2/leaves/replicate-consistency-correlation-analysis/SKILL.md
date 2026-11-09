---
name: replicate-consistency-correlation-analysis
description: Use when two or more ChIP-seq treatment replicates (plus controls) are
  supplied as FASTQ and the QC goal is to quantify replicate agreement via wiggle
  correlation and peak overlap before interpreting the experiment.
when_to_use_negative:
- Only one replicate is available — pairwise wiggle correlation and peak overlap cannot
  be computed.
- The goal is annotation or motif analysis of a single merged peak set rather than
  replicate QC — use ChiLin steps 6 and 8 instead.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: ChiLin
  role: Pipeline wrapper that orchestrates step 5 (per-replicate MACS2 peak calling
    plus replicate overlap/correlation analysis) and compiles the QC report
  repo: https://github.com/cfce/chilin
- name: MACS2
  role: Default peak caller used to call peaks for each replicate sample before overlap/correlation
    computation
  repo: https://github.com/taoliu/MACS
- name: BWA
  role: Default mapper producing the BAM files that feed the replicate peak-calling
    step
  repo: https://github.com/lh3/bwa
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-016-1274-4
    title: 'ChiLin: a comprehensive ChIP-seq and DNase-seq quality control and analysis
      pipeline'
  source_content_hash: sha256:222ed1f55f2f75bc407d43dac3bc2bf5385a3ae246b0d474da853226004295b4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfce/chilin
  tools:
  - BWA
  - ChiLin
  - MACS2
  - bedtools
  - samtools
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# replicate-consistency-correlation-analysis

## Summary

Assesses ChIP-seq biological replicate consistency within the ChiLin QC pipeline by calling peaks per replicate with MACS2 and computing wiggle correlations and replicate peak-overlap counts (pipeline step 5), reported against historical reference metrics (e.g. TRRAP wiggle correlation 0.14 / 20-peak overlap).

## When to use

Use when two or more ChIP-seq treatment replicates (plus controls) are supplied as FASTQ and the QC goal is to quantify replicate agreement via wiggle correlation and peak overlap before interpreting the experiment.

## When NOT to use

- Only one replicate is available — pairwise wiggle correlation and peak overlap cannot be computed.
- The goal is annotation or motif analysis of a single merged peak set rather than replicate QC — use ChiLin steps 6 and 8 instead.

## Inputs

- Treatment replicate FASTQ files (comma-separated, e.g. treat1.fastq,treat2.fastq)
- Control replicate FASTQ files (comma-separated, e.g. control1.fastq,control2.fastq)
- BAM files produced by the bwa mapping step
- Species-specific dependent data archive in db/ (e.g. hg19.tgz) verified with md5sum -c

## Outputs

- Per-replicate peak sets from MACS2 (narrow or broad)
- Replicate wiggle correlation values
- Replicate peak-overlap counts
- Aggregated QC statistics in id_summary.txt and id_report.pdf

## How to apply

Run the mandatory ChiLin core first: FastQC evaluation (step 1), bwa mapping (step 2, cannot be skipped since it produces the BAMs), MACS2 fragment-size estimation on 4M-read subsampled BAMs (step 3), and PBC evaluation (step 4). Then execute step 5, which calls peaks with MACS2 separately for each replicate and performs replicate peak overlap/correlation analysis; step 6 (merged-BAM peak calling) is also mandatory because it feeds the annotation step. Compare the resulting wiggle correlations and peak-overlap counts against the paper's reference values (e.g. TRRAP wiggle correlation 0.14 with 20 overlapping peaks despite the same anti-GFP Abcam antibody; H3K4me3 wiggle correlation 0.02) to judge replicate consistency. Consistent with the paper's validation rule, accept the run when computed correlations and overlap counts match or fall within the published reference ranges recorded in id_summary.txt and id_report.pdf.

## Related tools

- **ChiLin** (Pipeline wrapper that orchestrates step 5 (per-replicate MACS2 peak calling plus replicate overlap/correlation analysis) and compiles the QC report) — https://github.com/cfce/chilin
- **MACS2** (Default peak caller used to call peaks for each replicate sample before overlap/correlation computation) — https://github.com/taoliu/MACS
- **BWA** (Default mapper producing the BAM files that feed the replicate peak-calling step) — https://github.com/lh3/bwa

## Examples

```
chilin simple -u user -s species --threads 8 -i id -o output -t treat1.fastq,treat2.fastq -c control1.fastq,control2.fastq -p narrow -r tf
```

## Evaluation signals

- Replicate wiggle correlation reproduces the published reference value (e.g. TRRAP 0.14, H3K4me3 0.02) within tolerance.
- Replicate peak-overlap count matches the reference (e.g. 20 overlapping peaks for the two TRRAP anti-GFP biological replicates).
- id_summary.txt and id_report.pdf are generated and contain the replicate-consistency metrics alongside FRiP scores (e.g. H3K27me3 FRiP 0.04–0.32%, STAT6 FRiP <1% with 45 peaks).
- Both mandatory steps completed: step 2 BAM generation and step 6 merged-BAM peak calling were not skipped.
- Peak calling used MACS2 with the declared peak type (-p narrow for TF datasets).

## Limitations

- Merged peak calling (step 6) cannot be skipped, so replicate analysis cannot be run as a fully standalone shortcut within ChiLin.
- No software version is stated for ChiLin in the article's discussion of reproducibility, which can complicate exact reproduction of correlation/overlap values.
- Reproducibility of the QC values requires concrete commands and environment specification, which the discussion section notes are not provided there.

## Evidence

- [methods] Replicate peak overlap/correlation via MACS2 in pipeline step 5: "step 5: call peak for replicates samples, and do replicates peak overlap/correlation analysis"
- [abstract] ChiLin automates QC and data analysis for ChIP-seq: "ChiLin is a computational pipeline that automates the quality control and data analyses of ChIP-seq and DNase-seq data."
- [readme] Environment pinning needed to reproduce wiggle correlation via ucsc-wigcorrelate and MACS2 2.1.0: "conda install bwa=0.7.13 samtools=0.1.19 bedtools=2.17.0 seqtk ucsc-bedclip ucsc-bedgraphtobigwig ucsc-wigcorrelate ucsc-wigtobigwig fastqc numpy macs2=2.1.0"
- [abstract] QC reports compare current metrics against historical data: "ChiLin generates comprehensive quality control reports that include comparisons with historical data"
