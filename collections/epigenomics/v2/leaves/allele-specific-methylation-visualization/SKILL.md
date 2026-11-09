---
name: allele-specific-methylation-visualization
description: Use when WGBS reads have been aligned and methylation-called with Msuite
  and you need per-read, genotype-preserved methylation views at imprinted or allele-specific
  loci, e.g. to phase reads at a heterozygous A/G SNP and observe reciprocal methylation
  bars.
when_to_use_negative:
- Your alignments and methylation calls were not produced by Msuite — Mviewer is specifically
  optimized for Msuite alignment results, so non-Msuite BAM/methylation inputs are
  out of scope
- 'Your data are bisulfite-free TAPS/5hmC-CATCH libraries: Msuite''s 4-letter mode
  is the intended path there, not the 3-letter BS workflow this visualization skill
  depends on'
- You only need genome-wide methylation summary statistics rather than single-locus,
  per-read ASM inspection — methylation-call and QC outputs suffice without Mviewer
  navigation
edam_operation: http://edamontology.org/operation_3208
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0622
tools:
- name: Mviewer
  role: Renders nucleotide-level, genotype-preserved per-read methylation tracks from
    Msuite BAM and methylation call files, enabling allele-specific methylation inspection
    at heterozygous SNPs
  repo: https://github.com/hellosunking/Msuite/
- name: Msuite
  role: Produces the BAM (Msuite.final.bam) and CpG methylation calls (Msuite.CpG.meth.call)
    that Mviewer consumes, run here in 3-letter bisulfite mode (-3 -m BS) against
    GRCh38
  repo: https://github.com/hellosunking/Msuite/
- name: samtools
  role: Converts Msuite SAM output to BAM format for the Mviewer track
- name: bowtie2
  role: Underlying aligner used by Msuite for the 3-letter bisulfite alignment feeding
    the visualization
  repo: https://github.com/BenLangmead/bowtie2
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.patter.2020.100127
    title: 'Msuite: A High-Performance and Versatile DNA Methylation Data-Analysis
      Toolkit'
  source_content_hash: sha256:27749fbb1880410352b150cf13baa35292895c298f5a3c23f020bb2eb966ff68
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
  edam_operation: http://edamontology.org/operation_3208
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# allele-specific-methylation-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Nucleotide-level, genotype-preserved visualization of DNA methylation sequencing data with Mviewer, the Msuite companion tool, to expose allele-specific methylation (ASM) at heterozygous SNPs such as rs2301335 at the MEST imprinting locus on chr7:130493085 in human placental WGBS data (sample GSM1186665).

## When to use

Use when WGBS reads have been aligned and methylation-called with Msuite and you need per-read, genotype-preserved methylation views at imprinted or allele-specific loci, e.g. to phase reads at a heterozygous A/G SNP and observe reciprocal methylation bars.

## When NOT to use

- Your alignments and methylation calls were not produced by Msuite — Mviewer is specifically optimized for Msuite alignment results, so non-Msuite BAM/methylation inputs are out of scope
- Your data are bisulfite-free TAPS/5hmC-CATCH libraries: Msuite's 4-letter mode is the intended path there, not the 3-letter BS workflow this visualization skill depends on
- You only need genome-wide methylation summary statistics rather than single-locus, per-read ASM inspection — methylation-call and QC outputs suffice without Mviewer navigation

## Inputs

- Msuite.final.bam — BAM alignment from Msuite 3-letter BS-Seq run (samtools-converted from SAM)
- Msuite.CpG.meth.call — Msuite CpG methylation call file
- WGBS FASTQ files for human placental sample GSM1186665 (downstream input to the Msuite step)
- GRCh38/hg38 genome index built with build.index.sh
- Target genomic region, e.g. MEST locus around chr7:130493085 with dbSNP SNP rs2301335 (A/G heterozygous)

## Outputs

- Nucleotide-level, genotype-preserved per-read methylation track for the queried locus (Mviewer display)
- Rendered/exported figure of the Mviewer track, e.g. MEST_mviewer_track.png, highlighting allele-specific methylation bars at the SNP

## How to apply

First produce Msuite-compatible inputs: build the GRCh38/hg38 index with build.index.sh and run Msuite in 3-letter bisulfite mode (-3 -m BS) on the placental WGBS FASTQ (GSM1186665), yielding Msuite.final.bam (SAM converted to BAM with samtools) and the Msuite.CpG.meth.call methylation calls. Launch Mviewer — it is specifically optimized for Msuite alignment results — and load the BAM and methylation call files, navigating to the target locus (chr7:130493085 at MEST). Display the nucleotide-level per-read methylation state and annotate the heterozygous A/G SNP rs2301335 recorded in dbSNP; reads phased at the SNP should show reciprocal allele-specific methylation bars consistent with imprinting. Export the rendered track (e.g. MEST_mviewer_track.png) and verify it reproduces the expected ASM pattern of Figure 3C.

## Related tools

- **Mviewer** (Renders nucleotide-level, genotype-preserved per-read methylation tracks from Msuite BAM and methylation call files, enabling allele-specific methylation inspection at heterozygous SNPs) — https://github.com/hellosunking/Msuite/
- **Msuite** (Produces the BAM (Msuite.final.bam) and CpG methylation calls (Msuite.CpG.meth.call) that Mviewer consumes, run here in 3-letter bisulfite mode (-3 -m BS) against GRCh38) — https://github.com/hellosunking/Msuite/
- **samtools** (Converts Msuite SAM output to BAM format for the Mviewer track)
- **bowtie2** (Underlying aligner used by Msuite for the 3-letter bisulfite alignment feeding the visualization) — https://github.com/BenLangmead/bowtie2

## Examples

```
msuite -3 -m BS -x hg38 -1 GSM1186665_R1.fq -2 GSM1186665_R2.fq -o placental_wgbs && samtools view -bS placental_wgbs/Msuite.final.sam > placental_wgbs/Msuite.final.bam
```

## Evaluation signals

- The exported track (MEST_mviewer_track.png) exists as a valid image file (file_exists check)
- The A/G heterozygous SNP chr7:130493085 (rs2301335, dbSNP) is visibly annotated on the rendered track
- Reads phased at the SNP show reciprocal allele-specific methylation bars, consistent with an imprinted locus (expert_review)
- The rendered pattern visually reproduces Figure 3C from the Msuite paper for human placental WGBS sample GSM1186665
- Upstream files are complete and well-formed: Msuite.final.bam is a valid BAM (after samtools conversion) and Msuite.CpG.meth.call contains CpG-level methylation calls

## Limitations

- Mviewer is designed for compatibility with Msuite alignment results; using alignments from other pipelines is not supported by the article's workflow
- If data were generated with a BS-Seq protocol, Msuite must be run in 3-letter mode with -m BS; the 4-letter mode only supports TAPS/5hmC-CATCH data where non-CpG methylation is very low
- Msuite filters reads that map to multiple locations with equal best scores and treats identical start/end/strand reads as PCR duplicates, so the per-read ASM evidence in the track reflects those filters

## Evidence

- [discussion] its accompanying tool, Mviewer, provides favorable characteristics that can be specifically meaningful in scenarios with allele-specific DNA methylation: "its accompanying tool, Mviewer, provides favorable characteristics that can be specifically meaningful in scenarios with allele-specific DNA methylation"
- [discussion] SAM to BAM conversion via samtools: "samtools35 for converting SAM format files into BAM format"
- [methods] build genome indices before running Msuite: "Before running `Msuite`, genome indices must be built. To this end, we have prepared a utility named
`build.index.sh`"
- [readme] BS-seq data must use 3-letter mode with -m BS: "If your data is generated using BS-seq protocol, you MUST use the 3-letter mode and set
`-m BS`."
- [discussion] Multi-mapping reads are filtered to unique best hits: "Msuite screens for multiple-mapping reads and only those with unique best hits are kept"
