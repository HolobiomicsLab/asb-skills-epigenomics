---
name: genome-index-building
description: Use when preparing a reference genome (e.g., hg19/hg38, mm9/mm10 as multi-fasta
  or per-chromosome directory) before running Msuite alignment or the CT/GA-rich region
  benchmark, so converted indices exist for 3-letter or 4-letter TAPS/BS modes.
when_to_use_negative:
- The reference genome or annotation is not yet in FASTA/RefSeq form (e.g., GFF/GTF-only
  annotation) — convert formats first
- You only intend to align with Bismark or BWA-meth without Msuite — those tools build
  their own bowtie2-based conversion indices
- You are re-running Msuite on an existing sample and an index for that Genome.ID
  already exists under the Msuite index directory
edam_operation: http://edamontology.org/operation_0227
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0622
tools:
- name: Msuite
  role: Provides build.index.sh, which converts cytosines (3- or 4-letter mode) and
    builds Watson/Crick bowtie2 indices plus the Lambda genome into the Msuite index
    directory
  repo: https://github.com/hellosunking/Msuite/
- name: bowtie2
  role: Underlying aligner for which the FM indices are built; used by Msuite for
    read alignment against the constructed indices
  repo: https://github.com/BenLangmead/bowtie2
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1016/j.patter.2020.100127
    title: 'Msuite: A High-Performance and Versatile DNA Methylation Data-Analysis
      Toolkit'
  source_content_hash: sha256:d336db6bb4b6331f7ad4b05a8f3afa5ce3a1b3c4c3b232cff0ecea98d4f24f28
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
  - BWA-meth
  - Bismark
  - Msuite
  - bowtie2
  - samtools
  edam_operation: http://edamontology.org/operation_0227
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# genome-index-building

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Build Msuite-compatible genome indices from a reference FASTA and RefSeq annotation using build.index.sh, which converts cytosines (3- or 4-letter modes) and constructs separate Watson- and Crick-strand bowtie2 FM indices for DNA methylation read alignment.

## When to use

Use when preparing a reference genome (e.g., hg19/hg38, mm9/mm10 as multi-fasta or per-chromosome directory) before running Msuite alignment or the CT/GA-rich region benchmark, so converted indices exist for 3-letter or 4-letter TAPS/BS modes.

## When NOT to use

- The reference genome or annotation is not yet in FASTA/RefSeq form (e.g., GFF/GTF-only annotation) — convert formats first
- You only intend to align with Bismark or BWA-meth without Msuite — those tools build their own bowtie2-based conversion indices
- You are re-running Msuite on an existing sample and an index for that Genome.ID already exists under the Msuite index directory

## Inputs

- Reference genome sequence as a multi-fasta file (GENOME.FA) or a directory of per-chromosome sequences; gzip/bzip2 compression supported with matching suffix
- RefSeq annotation file (REFSEQ.txt) for the genome build; pre-included for mm9/mm10/hg19/hg38, downloadable from UCSC for other species
- User-defined Genome.ID string naming the genome

## Outputs

- Msuite genome indices written to the 'index' directory under the Msuite root, named by Genome.ID
- Converted genome indices for Watson and Crick strands (3-letter: all C->T; 4-letter/TAPS: only CpG cytosines -> T, CpH cytosines preserved), including the automatically incorporated Lambda genome

## How to apply

Run build.index/build.index.sh GENOME.FA(or GENOME.DIR) REFSEQ.txt Genome.ID, where the genome is either one multi-fasta file or a directory of per-chromosome sequences; Gzip/Bzip2-compressed inputs are accepted if they carry .gz/.bz2 suffixes. The utility automatically incorporates the Lambda genome (for spike-in calibration) and writes indices to the 'index' directory under the Msuite root, using the chosen Genome.ID as the name passed later to 'Msuite -x'. The RefSeq annotation (files included for mm9/mm10/hg19/hg38; others from UCSC) is used to profile methylation around Transcription Start Sites as a quality control. For 4-letter mode indices, only CpG-context cytosines are converted to thymines while CpH cytosines are preserved, and separate indices are built for the Watson and Crick strands because C-to-T conversion disrupts their reverse-complementarity. Matched Bismark and BWA-meth indices should also be prepared when benchmarking against these 3-letter aligners.

## Related tools

- **Msuite** (Provides build.index.sh, which converts cytosines (3- or 4-letter mode) and builds Watson/Crick bowtie2 indices plus the Lambda genome into the Msuite index directory) — https://github.com/hellosunking/Msuite/
- **bowtie2** (Underlying aligner for which the FM indices are built; used by Msuite for read alignment against the constructed indices) — https://github.com/BenLangmead/bowtie2

## Examples

```
build.index/build.index.sh GENOME.FA REFSEQ.txt hg38
```

## Evaluation signals

- The index directory under the Msuite root contains indices for the specified Genome.ID, including the automatically incorporated Lambda genome
- For 4-letter (TAPS) indices, a verification report (e.g., four_letter_verification_report.txt) records that CpH cytosines are preserved in the converted reference while only CpG cytosines became thymines
- Watson- and Crick-strand indices are both built, since C-to-T conversion breaks reverse-complementarity between strands
- Simulated reads (e.g., SHERMAN TAPS reads with 20% C->T at CpG and 0.5% at CpH) align successfully with '-4 -m TAPS' and yield no multi-mappers or duplicate start/end/strand pairs in Msuite.rmdup.sam after the default filters (-q 20, --minsize 22)

## Limitations

- The utility requires both a genome sequence and RefSeq annotation; the annotation is mandatory because it feeds the TSS-methylation quality control
- Index building must be repeated per genome build; only mm9/mm10/hg19/hg38 annotation files ship with the package, others must be fetched from UCSC
- Msuite may fail to run normally if the pre-compiled binaries are incompatible with the system libc++ version, requiring recompilation with g++ > 4.8 before indices and alignments can be used

## Evidence
