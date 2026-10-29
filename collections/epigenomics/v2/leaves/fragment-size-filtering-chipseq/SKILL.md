---
name: fragment-size-filtering-chipseq
description: Use when mapped BAM alignments from a NEAT ChIPseq or RNAseq pipeline
  need fragment-size bounds and duplicate-read handling applied before peakcalling,
  per parameters set in AdvancedSettings.txt.
when_to_use_negative:
- Input is raw FASTQ that has not yet been mapped — NEAT's 'map' module (bwa/bowtie)
  must run before filtering
- The goal is configuring how peaks are called rather than filtering reads — that
  is the separate 'peakcalling' module
- Reads still need renaming, unzipping, or QC — use NEAT's 'unzip', 'rename', or 'QC'
  modules first
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: SAMtools
  role: Provides view (fragment-size boundary filtering), rmdup (duplicate-read management),
    sort, and index used to implement the filter step
- name: NEAT
  role: Framework whose 'filter' module reads user parameters from AdvancedSettings.txt
    and orchestrates the samtools-based filtering between 'map' and 'peakcalling'
  repo: https://github.com/pschorderet/NEAT
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-016-0902-3
    title: 'NEAT: a framework for building fully automated NGS pipelines and analyses'
  source_content_hash: sha256:bce61232d83f40aacc808e9fdd3905bacccfedd3d2f817c4d672e45069ce25a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pschorderet/NEAT
  tools:
  - NEAT
  - SAMtools
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# fragment-size-filtering-chipseq

## Summary

Filters aligned ChIPseq/RNAseq BAM files in NEAT's 'filter' module by applying user-specified duplicate-read management and minimum/maximum fragment size thresholds, implemented with samtools view/rmdup/sort/index.

## When to use

Use when mapped BAM alignments from a NEAT ChIPseq or RNAseq pipeline need fragment-size bounds and duplicate-read handling applied before peakcalling, per parameters set in AdvancedSettings.txt.

## When NOT to use

- Input is raw FASTQ that has not yet been mapped — NEAT's 'map' module (bwa/bowtie) must run before filtering
- The goal is configuring how peaks are called rather than filtering reads — that is the separate 'peakcalling' module
- Reads still need renaming, unzipping, or QC — use NEAT's 'unzip', 'rename', or 'QC' modules first

## Inputs

- Aligned BAM file produced by NEAT's 'map' module (bwa or bowtie)
- Filtering parameters from AdvancedSettings.txt: duplicate-read management policy, minimum and maximum fragment sizes

## Outputs

- Filtered, sorted BAM file
- Companion .bai index file

## How to apply

Read the duplicate-read policy and the minimum and maximum fragment sizes the user configured in NEAT's central AdvancedSettings.txt file. Use samtools view to keep only alignments whose fragment sizes fall within the configured boundaries, then apply samtools rmdup according to the duplicate-management policy. Sort the filtered alignments with samtools sort and generate the companion .bai index with samtools index. The filter module sits between the 'map' step (bwa or bowtie) and the 'peakcalling' step in NEAT's pipeline, so its output must be a valid, sorted, indexed BAM ready for downstream peak calling.

## Related tools

- **SAMtools** (Provides view (fragment-size boundary filtering), rmdup (duplicate-read management), sort, and index used to implement the filter step)
- **NEAT** (Framework whose 'filter' module reads user parameters from AdvancedSettings.txt and orchestrates the samtools-based filtering between 'map' and 'peakcalling') — https://github.com/pschorderet/NEAT

## Examples

```
samtools view -b aligned.bam | samtools rmdup - filtered.bam && samtools sort filtered.bam -o filtered.sorted.bam && samtools index filtered.sorted.bam
```

## Evaluation signals

- Filtered .bam output file exists in the project folder
- Filtered output parses as a valid BAM file (e.g., samtools quickcheck or a fresh header read)
- A .bai index file accompanies the filtered .bam
- All retained alignments fall within the minimum/maximum fragment size boundaries set in AdvancedSettings.txt
- Duplicate reads are handled according to the duplicate-management policy specified in AdvancedSettings.txt

## Limitations

- Filtering parameters live in a single centralized AdvancedSettings.txt file, so changes affect the whole project rather than a single sample
- NEAT is still under development ('Devel mode'); bugs should be expected and reported
- Pipeline is targeted at ChIPseq/RNAseq from mouse embryonic stem cell test data (truncated H3K4me3 ChIPseq and RNAseq datasets); behavior on other data types is not validated here

## Evidence

- [methods] The 'filter' module allows the user to specify filtering parameters (AdvancedSettings.txt) including how to manage duplicate reads, minimum and maximum size of fragments: "The ‘filter’ module allows the user to specify filtering parameters (AdvancedSettings.txt) including how to manage duplicate reads, minimum and maximum size of fragments"
- [abstract] NEAT projects are built around a unique and centralized file containing sample names, replicates, conditions, antibodies, alignment-, filtering- and peak calling parameters as well as: "NEAT projects are built around a unique and centralized file containing sample names, replicates, conditions, antibodies, alignment-, filtering- and peak calling parameters as well as"
- [intro] NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc): "NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc)"
- [methods] The ‘map’ module maps reads using either bwa [18] or bowtie [19].: "The ‘map’ module maps reads using either bwa [18] or bowtie [19]."
