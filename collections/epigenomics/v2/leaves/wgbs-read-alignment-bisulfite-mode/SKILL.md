---
name: wgbs-read-alignment-bisulfite-mode
description: Use when raw WGBS/BS-seq FASTQ reads must be aligned to a reference genome
  while preserving cytosine methylation information, and the protocol is bisulfite
  (not TAPS/5hmC-CATCH).
when_to_use_negative:
- Data from TAPS/5hmC-CATCH or other bisulfite-free protocols with low non-CpG methylation
  — use Msuite's 4-letter mode (-4) instead, since 3-letter mode is required for BS-seq
- Input is already an aligned BAM/methylation call file rather than raw FASTQ — skip
  directly to methylation calling or Mviewer visualization
- A different aligner's output must be reused unchanged — this skill is specific to
  Msuite's 3-letter alignment and its built-in multi-mapping/duplicate filters
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_3308
tools:
- name: Msuite
  role: 'Primary all-in-one toolkit: performs the 3-letter bisulfite alignment (-3
    -m BS), multi-mapping and PCR-duplicate filtering, and methylation calling'
  repo: https://github.com/hellosunking/Msuite/
- name: bowtie2
  role: Underline aligner invoked by Msuite for the converted-genome read mapping
  repo: https://github.com/BenLangmead/bowtie2
- name: samtools
  role: Converts Msuite's SAM-format alignment output into the final BAM file
  repo: http://samtools.sourceforge.net/
- name: build.index.sh
  role: Msuite utility that builds the bisulfite-converted Watson/Crick genome indices
    from the reference FASTA and RefSeq annotation before alignment
  repo: https://github.com/hellosunking/Msuite/
- name: Mviewer
  role: Downstream nucleotide-level, genotype-preserved visualization of the resulting
    BAM and methylation calls (compatible with Msuite alignment results)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.patter.2020.100127
    title: 'Msuite: A High-Performance and Versatile DNA Methylation Data-Analysis
      Toolkit'
  source_content_hash: sha256:15d31e830d5e49901c2a1f76abeae817ed41f20abaa4505d00343ad3e60f0a2d
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
  - build.index.sh
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# wgbs-read-alignment-bisulfite-mode

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aligns WGBS/bisulfite-seq FASTQ reads using Msuite in 3-letter mode (-3 -m BS) against a bisulfite-converted genome index built with build.index.sh, producing Msuite.final.bam and CpG methylation calls (Msuite.CpG.meth.call) with integrated multi-mapping and PCR-duplicate filtering.

## When to use

Use when raw WGBS/BS-seq FASTQ reads must be aligned to a reference genome while preserving cytosine methylation information, and the protocol is bisulfite (not TAPS/5hmC-CATCH).

## When NOT to use

- Data from TAPS/5hmC-CATCH or other bisulfite-free protocols with low non-CpG methylation — use Msuite's 4-letter mode (-4) instead, since 3-letter mode is required for BS-seq
- Input is already an aligned BAM/methylation call file rather than raw FASTQ — skip directly to methylation calling or Mviewer visualization
- A different aligner's output must be reused unchanged — this skill is specific to Msuite's 3-letter alignment and its built-in multi-mapping/duplicate filters

## Inputs

- Raw WGBS/BS-seq FASTQ files (single-end via -U, paired-end via -1/-2; comma-separated for multiple lanes), e.g., GSM1186665 human placental WGBS FASTQ
- Reference genome FASTA (single multi-fasta or per-chromosome directory), e.g., GRCh38/hg38
- RefSeq annotation file (mm9/mm10/hg19/hg38 included in package) for TSS-based QC

## Outputs

- Msuite.final.bam (alignment results, SAM converted to BAM via samtools)
- Msuite.CpG.meth.call (CpG methylation call file)
- TSS methylation-level QC profile (quality control around Transcription Start Sites)

## How to apply

First build genome indices with `build.index.sh GENOME.FA REFSEQ.txt Genome.ID` (e.g., hg38/GRCh38), which automatically incorporates the Lambda genome and writes indices to Msuite's `index` directory; ensure bowtie2 and samtools are in PATH. Because the data is BS-seq, you MUST run in 3-letter mode with `-m BS` (the `-4` mode is only for TAPS/5hmC-CATCH data with low non-CpG methylation), supplying read 1 (and read 2 for paired-end) FASTQ. Msuite converts all genomic cytosines to thymines, builds Watson/Crick chain indices, and uses bowtie2 under the hood — reported to be faster with lower memory usage than Bismark which also uses bowtie2. Msuite automatically discards multi-mapping reads with equal best scores on the same strand and treats reads with identical start/end/strand as PCR duplicates, keeping only the best-quality read. Convert the SAM output to BAM with samtools to obtain Msuite.final.bam, then proceed to methylation calling (Msuite.CpG.meth.call) and visualization, e.g., with Mviewer at candidate loci. Key tunables: `-q` minimum cycle quality (default 20), `--minsize` minimum read length (default 22), `-Q` minimum alignment score for methylation calling (default 0), and `-p` threads (default 4).

## Related tools

- **Msuite** (Primary all-in-one toolkit: performs the 3-letter bisulfite alignment (-3 -m BS), multi-mapping and PCR-duplicate filtering, and methylation calling) — https://github.com/hellosunking/Msuite/
- **bowtie2** (Underline aligner invoked by Msuite for the converted-genome read mapping) — https://github.com/BenLangmead/bowtie2
- **samtools** (Converts Msuite's SAM-format alignment output into the final BAM file) — http://samtools.sourceforge.net/
- **build.index.sh** (Msuite utility that builds the bisulfite-converted Watson/Crick genome indices from the reference FASTA and RefSeq annotation before alignment) — https://github.com/hellosunking/Msuite/
- **Mviewer** (Downstream nucleotide-level, genotype-preserved visualization of the resulting BAM and methylation calls (compatible with Msuite alignment results))

## Examples

```
build.index/build.index.sh GRCh38.fa refseq.hg38.txt hg38 && msuite -x hg38 -1 Read1.fq.gz -2 Read2.fq.gz -3 -m BS -o out.dir && samtools view -bS out.dir/Msuite.sam > Msuite.final.bam
```

## Evaluation signals

- Msuite.final.bam exists and is a valid BAM (samtools quickcheck/flagstat runs without error)
- Msuite.CpG.meth.call is produced with CpG-level methylation calls honoring -Q minimum alignment score (default 0)
- Multi-mapping filter applied: reads mapping to multiple locations on the same strand with equal best scores are absent from the BAM
- PCR duplicate filter applied: among reads with identical start/end/strand, only the best-quality read is retained
- Mapping efficiency and speed are competitive with (better than) Bismark run with bowtie2 on the same in silico SHERMAN-generated benchmark data

## Limitations

- 3-letter mode converts all cytosines (in reads and reference) to thymines, so it is designed for BS-seq; 4-letter mode is NOT interchangeable and only supports TAPS/5hmC-CATCH data with very low non-CpG methylation
- Msuite caps reads handled at 4,294,967,295 and has a --minsize default of 22, so very short or extremely large inputs need parameter attention
- The article notes accuracy of 4- and 3-letter modes and limitations of current sequencing-based protocols require validation with additional methods such as microarrays; no strong evidence that 4-letter mode is more accurate on inconsistent CpG sites
- Quality scores starting from 35 ('#') in FASTQ require adjusting the -q option (e.g., --phred33 -q 22)

## Evidence
