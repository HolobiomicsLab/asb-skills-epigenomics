---
name: bwa-read-alignment-and-bam-processing
description: Use when starting from raw ChIP-seq or DNase-seq FASTQ files and you
  need aligned, sorted BAM files against hg19 as the substrate for MACS2 peak calling
  and FRiP-based quality control.
when_to_use_negative:
- Input data are already aligned BAMs or published peak BED files — ChiLin marks the
  mapping step as already-provided input, not something to redo
- Long-read sequencing data — BWA-MEM short-read mapping is designed for Illumina
  reads of ~70–100bp, and the article's scope is ChIP-seq/DNase-seq short-read QC
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3168
tools:
- name: BWA
  role: Default aligner in ChiLin step 2 that maps raw FASTQ reads against hg19 to
    produce the BAM files required by all downstream QC steps
  repo: https://github.com/lh3/bwa
- name: seqtk
  role: Sub-samples the aligned BAM-derived FASTQ data at stepwise depths (e.g. 1M–4M
    reads) for sequencing-depth stability experiments
  repo: https://github.com/lh3/seqtk
- name: ChiLin
  role: Pipeline wrapper whose step 2 performs bwa (default), bowtie or star mapping
    and cannot be skipped because it provides the necessary BAM files
  repo: https://github.com/cfce/chilin
- name: MACS2
  role: Downstream peak caller (ChiLin default, narrow type for TF data) that consumes
    the BWA-produced BAMs for fragment-size estimation and peak calling
  repo: https://github.com/taoliu/MACS
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-016-1274-4
    title: 'ChiLin: a comprehensive ChIP-seq and DNase-seq quality control and analysis
      pipeline'
  source_content_hash: sha256:a5875c9444e412642dbe9b6ce223d6d5df0f74ffa10c102ab46b6667477a2d3b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/bwa
  tools:
  - BWA
  - ChiLin
  - MACS2
  - seqtk
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5294117647058824
n_claims: 17
---

# bwa-read-alignment-and-bam-processing

## Summary

Map ChIP-seq/DNase-seq raw FASTQ reads to the hg19 reference genome with BWA (ChiLin's default aligner, v0.7.13 in the reference install) to produce the sorted BAM files that all downstream ChiLin QC steps — fragment-size estimation, PBC, peak calling, and FRiP score calculation — depend on.

## When to use

Use when starting from raw ChIP-seq or DNase-seq FASTQ files and you need aligned, sorted BAM files against hg19 as the substrate for MACS2 peak calling and FRiP-based quality control.

## When NOT to use

- Input data are already aligned BAMs or published peak BED files — ChiLin marks the mapping step as already-provided input, not something to redo
- Long-read sequencing data — BWA-MEM short-read mapping is designed for Illumina reads of ~70–100bp, and the article's scope is ChIP-seq/DNase-seq short-read QC

## Inputs

- Raw ChIP-seq or DNase-seq FASTQ files (Illumina short reads, ~100bp or less for BWA-backtrack)
- Indexed hg19 reference genome (cistrome.org chilin hg19.tgz download)

## Outputs

- Sorted BAM files aligned to hg19
- BAM files suitable for sub-sampling and MACS2 narrow-peak calling in ChiLin steps 3–7

## How to apply

First download ChiLin's hg19 dependent data (e.g. `wget -c http://cistrome.org/chilin/_downloads/hg19.tgz`) and index the reference with `bwa index`. Run BWA as step 2 of the ChiLin pipeline, which by default uses bwa (with bowtie or star as alternates); this step cannot be skipped because it supplies the BAM files required by every later step. Sort the resulting SAM output into BAM, then subsample the BAMs at stepwise sequencing depths (starting from 1M reads, per the ChiLin/seqtk convention) if a depth-stability assessment such as the FRiP Fig. 2d/2e comparison is the goal. The aligned BAMs feed MACS2 fragment-size estimation (step 3), PBC evaluation (step 4), replicate peak-overlap analysis (step 5), merged peak calling (step 6), and FRiP score computation (step 7).

## Related tools

- **BWA** (Default aligner in ChiLin step 2 that maps raw FASTQ reads against hg19 to produce the BAM files required by all downstream QC steps) — https://github.com/lh3/bwa
- **seqtk** (Sub-samples the aligned BAM-derived FASTQ data at stepwise depths (e.g. 1M–4M reads) for sequencing-depth stability experiments) — https://github.com/lh3/seqtk
- **ChiLin** (Pipeline wrapper whose step 2 performs bwa (default), bowtie or star mapping and cannot be skipped because it provides the necessary BAM files) — https://github.com/cfce/chilin
- **MACS2** (Downstream peak caller (ChiLin default, narrow type for TF data) that consumes the BWA-produced BAMs for fragment-size estimation and peak calling) — https://github.com/taoliu/MACS

## Examples

```
bwa index hg19.fa && bwa aln hg19.fa reads.fq > reads.sai && bwa samse hg19.fa reads.sai reads.fq | samtools sort -o sorted.bam -
```

## Evaluation signals

- Sorted BAM files are produced and non-empty; the article notes mapping 'cannot be skipped, because this provided necessary BAM files'
- Sub-sampled BAMs follow the ChiLin convention of stepwise depths starting from 1M reads (e.g. the 4M-read subsampled BAM files it produces)
- Downstream steps run cleanly on the BAMs: MACS2 fragment-size estimation (step 3) and FRiP scores (step 7, id_frip./*.frip outputs) are generated without missing-BAM errors
- For depth-stability use, FRiP computed on peaks called from the full-depth BAM remains approximately flat across depths while down-sampled peak calling degrades at shallow depths (Fig. 2d vs 2e pattern)

## Limitations

- BWA-backtrack applies to Illumina reads up to 100bp; BWA-MEM is recommended for 70–100bp and longer, so algorithm choice depends on read length
- ChiLin's mapping step requires pre-downloaded reference data (hg19/hg38/mm9/mm10 tgz packages) and a working install of pinned versions (bwa=0.7.13, samtools=0.1.19, macs2=2.1.0 in the reference conda environment)
- No ChiLin software version is stated in the article and the discussion notes no concrete commands or environment specification accompany the reproducibility claim, so exact reproduction may require inferring versions from the repo's install instructions

## Evidence

- [methods] Mapping (bwa/bowtie/star) to produce sorted BAM files as the mandatory step: "step 2: bwa(default), bowtie or star mapping, this step cannot be skipped, because this provided necessary BAM files."
- [intro] ChiLin uses MACS2 as the default peak caller consuming the aligned BAMs: "ChiLin uses MACS2 (https://github.com/taoliu/MACS/) as the default peak caller"
- [methods] hg19 reference data must be downloaded for alignment: "wget -c http://cistrome.org/chilin/_downloads/hg19.tgz"
- [readme] Pinned BWA version in the reference installation: "conda install bwa=0.7.13 samtools=0.1.19 bedtools=2.17.0 seqtk ucsc-bedclip"
- [readme] BWA invocation pattern for mapping short reads: [evidence span withheld — anchor status: external_doc]
