---
name: snp-phased-methylation-track-inspection
description: Use when a WGBS sample may harbor allele-specific methylation at an imprinting
  locus or known heterozygous SNP, and per-read, genotype-preserving visualization
  is needed (e.g., A/G SNP at MEST on chr7).
when_to_use_negative:
- The library was generated with a bisulfite-free protocol (TAPS/5hmC-CATCH) — Msuite's
  README states 4-letter mode is required there, while BS-seq data MUST use 3-letter
  mode; the visualization assumptions differ
- The analysis target is genome-wide methylation summarization rather than single-locus,
  per-read inspection — methylation-call files and QC summaries are the appropriate
  output
- No heterozygous SNP or phased genotype information is available in the region, so
  allele-specific methylation cannot be resolved from reads
edam_operation: http://edamontology.org/operation_3207
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0622
tools:
- name: Msuite
  role: Aligns WGBS reads in 3-letter bisulfite mode and produces the BAM and CpG
    methylation-call files that Mviewer consumes
  repo: https://github.com/hellosunking/Msuite/
- name: Mviewer
  role: Renders the nucleotide-level, genotype-preserved per-read methylation track
    at the SNP for allele-specific methylation inspection
  repo: https://github.com/hellosunking/Msuite/
- name: samtools
  role: Converts Msuite's SAM alignment output into BAM format for Mviewer input
  repo: http://samtools.sourceforge.net/
- name: bowtie2
  role: Underlying aligner used by Msuite for 3-letter bisulfite read mapping
  repo: https://github.com/BenLangmead/bowtie2
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.patter.2020.100127
    title: 'Msuite: A High-Performance and Versatile DNA Methylation Data-Analysis
      Toolkit'
  source_content_hash: sha256:795dd1c74efd00140e74351c310f307f75a835c18c36872b7c6c03ff07229c30
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
  - Mviewer
  - bowtie2
  - samtools
  edam_operation: http://edamontology.org/operation_3207
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# snp-phased-methylation-track-inspection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Inspect nucleotide-level, genotype-preserved per-read methylation at a heterozygous SNP to detect allele-specific methylation, using Mviewer on Msuite-derived BAM and methylation-call files from WGBS data.

## When to use

Use when a WGBS sample may harbor allele-specific methylation at an imprinting locus or known heterozygous SNP, and per-read, genotype-preserving visualization is needed (e.g., A/G SNP at MEST on chr7).

## When NOT to use

- The library was generated with a bisulfite-free protocol (TAPS/5hmC-CATCH) — Msuite's README states 4-letter mode is required there, while BS-seq data MUST use 3-letter mode; the visualization assumptions differ
- The analysis target is genome-wide methylation summarization rather than single-locus, per-read inspection — methylation-call files and QC summaries are the appropriate output
- No heterozygous SNP or phased genotype information is available in the region, so allele-specific methylation cannot be resolved from reads

## Inputs

- WGBS FASTQ files (e.g., GEO sample GSM1186665, human placental tissue)
- GRCh38/hg38 genome index built with Msuite's build.index.sh plus RefSeq annotation
- Msuite output BAM file (Msuite.final.bam, converted from SAM via samtools)
- Msuite methylation-call file (e.g., Msuite.CpG.meth.call)
- Optional: dbSNP identifier or coordinates of a candidate heterozygous SNP

## Outputs

- Nucleotide-level, genotype-preserved per-read methylation track in Mviewer for the queried region
- Exported track figure highlighting allele-specific methylation bars at the SNP (e.g., MEST_mviewer_track.png)
- Annotated heterozygous SNP (e.g., rs2301335 at chr7:130493085) with allele-phased methylation calls

## How to apply

Process the WGBS FASTQ (e.g., sample GSM1186665 from human placenta) with Msuite in 3-letter bisulfite mode ('-3 -m BS') against a GRCh38/hg38 index built with build.index.sh, producing Msuite.final.bam (converted from SAM with samtools) and the CpG methylation-call file. Launch Mviewer, which is specifically optimized for compatibility with Msuite alignment results, and load both the BAM and methylation-call files for the target region. Navigate to the locus of interest (e.g., MEST around chr7:130493085) and read the nucleotide-level, genotype-preserved per-read methylation states: reads are phased by the A/G SNP (rs2301335 in dbSNP) so reciprocal allele-specific methylation bars become visible. Judge success by whether the rendered track shows the heterozygous SNP annotated and reads partitioned into two alleles with distinctly different methylation levels, consistent with an imprinted locus; export the track (e.g., MEST_mviewer_track.png).

## Related tools

- **Msuite** (Aligns WGBS reads in 3-letter bisulfite mode and produces the BAM and CpG methylation-call files that Mviewer consumes) — https://github.com/hellosunking/Msuite/
- **Mviewer** (Renders the nucleotide-level, genotype-preserved per-read methylation track at the SNP for allele-specific methylation inspection) — https://github.com/hellosunking/Msuite/
- **samtools** (Converts Msuite's SAM alignment output into BAM format for Mviewer input) — http://samtools.sourceforge.net/
- **bowtie2** (Underlying aligner used by Msuite for 3-letter bisulfite read mapping) — https://github.com/BenLangmead/bowtie2

## Examples

```
msuite -3 -m BS -x hg38 -1 GSM1186665_R1.fq -2 GSM1186665_R2.fq -o out.dir  # then load Msuite.final.bam + Msuite.CpG.meth.call in Mviewer at chr7:130493085
```

## Evaluation signals

- The Mviewer track exports as a valid image file (e.g., MEST_mviewer_track.png passes a file_exists check)
- The rendered track visually reproduces the paper's Figure 3C pattern: the A/G heterozygous SNP at chr7:130493085 (rs2301335) is annotated
- Reads phased at the SNP show reciprocal allele-specific methylation bars, i.e., one allele hypermethylated and the other hypomethylated, consistent with an imprinted locus
- BAM and methylation-call inputs are standard Msuite outputs (Msuite.final.bam, Msuite.CpG.meth.call) and are accepted by Mviewer without reprocessing
- Expert review confirms the per-read methylation states are genotype-preserved (read sequences retain the original A/G base rather than a bisulfite-converted surrogate)

## Limitations

- For BS-seq data Msuite MUST be run in 3-letter mode with '-m BS'; 4-letter mode only supports TAPS/5hmC-CATCH data with very low non-CpG methylation, so the same visualization pipeline is not interchangeable across protocols
- Mviewer's allele-specific value is specifically meaningful because it is compatible with Msuite alignment results; paired methylation-call files from other pipelines may not be supported
- Mviewer provides favorable characteristics specifically in scenarios with allele-specific DNA methylation — it is not a general-purpose genome browser replacement

## Evidence
