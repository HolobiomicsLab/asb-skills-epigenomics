---
name: imprinted-locus-methylation-interpretation
description: Use when a WGBS/bisulfite-seq dataset must be interpreted at a known
  imprinted or allele-specifically methylated locus, and per-read methylation phased
  by a heterozygous SNP genotype is needed.
when_to_use_negative:
- Data generated with TAPS or 5hmC-CATCH protocols — the README restricts 4-letter
  mode to those and mandates 3-letter mode with `-m BS` for BS-seq; mixing these invalidates
  the calls
- Analyses requiring only aggregated CpG methylation levels with no allele/resolution
  read-out — Mviewer's per-read, genotype-preserved view adds nothing
- Loci without a heterozygous SNP marker to phase reads by — allele-specific methylation
  cannot be interpreted at single-allele level from the track
edam_operation: http://edamontology.org/operation_3206
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0160
tools:
- name: Msuite
  role: Runs 3-letter BS-mode alignment and methylation calling producing Msuite.final.bam
    and Msuite.CpG.meth.call consumed by Mviewer
  repo: https://github.com/hellosunking/Msuite/
- name: Mviewer
  role: Renders the nucleotide-level, genotype-preserved per-read methylation track
    at the MEST locus with SNP annotation
- name: bowtie2
  role: Underlying aligner used by Msuite for bisulfite read alignment
  repo: https://github.com/BenLangmead/bowtie2
- name: samtools
  role: Converts Msuite SAM output into the BAM format loaded into Mviewer
  repo: http://samtools.sourceforge.net/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.patter.2020.100127
    title: 'Msuite: A High-Performance and Versatile DNA Methylation Data-Analysis
      Toolkit'
  source_content_hash: sha256:1cf125850f748c1b011fb92fd5311fd3692927d5157e7c52ffc3f31b19bd6d2d
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
  edam_operation: http://edamontology.org/operation_3206
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# imprinted-locus-methylation-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Interprets allele-specific DNA methylation at imprinted loci (e.g., MEST on chr7) by visualizing nucleotide-level, genotype-preserved per-read methylation states with Mviewer on Msuite bisulfite-mode outputs, annotating heterozygous SNPs such as rs2301335.

## When to use

Use when a WGBS/bisulfite-seq dataset must be interpreted at a known imprinted or allele-specifically methylated locus, and per-read methylation phased by a heterozygous SNP genotype is needed.

## When NOT to use

- Data generated with TAPS or 5hmC-CATCH protocols — the README restricts 4-letter mode to those and mandates 3-letter mode with `-m BS` for BS-seq; mixing these invalidates the calls
- Analyses requiring only aggregated CpG methylation levels with no allele/resolution read-out — Mviewer's per-read, genotype-preserved view adds nothing
- Loci without a heterozygous SNP marker to phase reads by — allele-specific methylation cannot be interpreted at single-allele level from the track

## Inputs

- Human placental WGBS FASTQ files (e.g., GEO sample GSM1186665)
- GRCh38/hg38 genome index built with Msuite build.index.sh
- Msuite.final.bam alignment file (SAM converted to BAM with samtools)
- Msuite.CpG.meth.call methylation call file
- dbSNP annotation for the heterozygous SNP (e.g., rs2301335 at chr7:130493085)

## Outputs

- Mviewer nucleotide-level, genotype-preserved per-read methylation track at the MEST locus
- Annotated heterozygous A/G SNP marker (rs2301335) on the track
- Exported track image (e.g., MEST_mviewer_track.png) reproducing the article's Figure 3C pattern

## How to apply

Process the BS-seq FASTQ data with Msuite in 3-letter bisulfite mode (`-3 -m BS`) against a GRCh38/hg38 index built with build.index.sh, since the README mandates 3-letter mode for BS-seq data. Record alignments as Msuite.final.bam (via samtools SAM-to-BAM conversion) and generate methylation calls in Msuite.CpG.meth.call. Launch Mviewer, which is optimized for compatibility with Msuite alignment results, loading both the BAM and methylation call files for the locus region. Navigate to the target locus (e.g., MEST around chr7:130493085), display nucleotide-level genotype-preserved per-read methylation, and annotate the heterozygous A/G SNP (rs2301335 in dbSNP); reads phased at the SNP should show reciprocal methylation bars consistent with imprinting. Export the track (e.g., MEST_mviewer_track.png) and validate that the annotated SNP and reciprocal allele-specific methylation pattern are visible.

## Related tools

- **Msuite** (Runs 3-letter BS-mode alignment and methylation calling producing Msuite.final.bam and Msuite.CpG.meth.call consumed by Mviewer) — https://github.com/hellosunking/Msuite/
- **Mviewer** (Renders the nucleotide-level, genotype-preserved per-read methylation track at the MEST locus with SNP annotation)
- **bowtie2** (Underlying aligner used by Msuite for bisulfite read alignment) — https://github.com/BenLangmead/bowtie2
- **samtools** (Converts Msuite SAM output into the BAM format loaded into Mviewer) — http://samtools.sourceforge.net/

## Examples

```
msuite -3 -m BS -x hg38 -1 GSM1186665_R1.fq.gz -2 GSM1186665_R2.fq.gz -o placental_wgbs.dir
```

## Evaluation signals

- The exported track file exists and is a valid image (file_exists check on MEST_mviewer_track.png)
- The rendered track visually reproduces the Figure 3C pattern: the A/G heterozygous SNP rs2301335 at chr7:130493085 is annotated
- Reads phased at the SNP show reciprocal allele-specific methylation bars consistent with an imprinted locus (expert review)
- Alignment and methylation artifacts Msuite.final.bam and Msuite.CpG.meth.call were produced from the GSM1186665 data with 3-letter BS mode
- Mviewer was loaded with both the BAM and methylation call files for the chr7 MEST region, per its Msuite-compatibility design

## Limitations

- Mviewer's strengths are specifically meaningful for allele-specific DNA methylation scenarios; for bulk-level questions other visualizations may suffice
- BS-seq data must use 3-letter mode with `-m BS`; 4-letter mode only supports TAPS/5hmC-CATCH data where non-CpG methylation is very low
- Genome indices must be pre-built with build.index.sh before running Msuite (hg38/mm10 files included in the package)

## Evidence
