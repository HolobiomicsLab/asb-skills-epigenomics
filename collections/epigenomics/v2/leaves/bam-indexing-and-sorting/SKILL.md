---
name: bam-indexing-and-sorting
description: Use when mapped reads (BAM) must be filtered by fragment-size thresholds,
  deduplicated, sorted, and indexed before ChIPseq/RNAseq peak calling in an NEAT-style
  pipeline.
when_to_use_negative:
- Input is still raw FASTQ and reads have not yet been mapped — use NEAT's 'map' module
  first
- The next step is QC, renaming, or ChIP-rx normalization rather than preparation
  for peakcalling — other NEAT modules handle these
- Only reorganizing existing .bam/.bai files for storage is needed — use NEAT's 'cleanfiles'
  module instead
tools:
- name: SAMtools
  role: Provides view, rmdup, sort, and index commands that implement fragment-size
    filtering, duplicate management, coordinate sorting, and BAM indexing in NEAT's
    'filter' module
- name: NEAT
  role: Framework that exposes the filter module as an automated pipeline step between
    'map' and 'peakcalling', parameterized via AdvancedSettings.txt
  repo: https://github.com/pschorderet/NEAT
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-016-0902-3
    title: 'NEAT: a framework for building fully automated NGS pipelines and analyses'
  source_content_hash: sha256:4c817b7b72394545f48142b0c0e77a74cfa8404c8d2b0240e7a3da870e790161
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pschorderet/NEAT
  tools:
  - NEAT
  - SAMtools
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# bam-indexing-and-sorting

## Summary

Filters, deduplicates, sorts, and indexes aligned BAM files using SAMtools as implemented in NEAT's 'filter' module, the pipeline step positioned between 'map' and 'peakcalling'. It enforces user-specified fragment-size and duplicate-read policies so that peak calling operates on clean, indexed alignments.

## When to use

Use when mapped reads (BAM) must be filtered by fragment-size thresholds, deduplicated, sorted, and indexed before ChIPseq/RNAseq peak calling in an NEAT-style pipeline.

## When NOT to use

- Input is still raw FASTQ and reads have not yet been mapped — use NEAT's 'map' module first
- The next step is QC, renaming, or ChIP-rx normalization rather than preparation for peakcalling — other NEAT modules handle these
- Only reorganizing existing .bam/.bai files for storage is needed — use NEAT's 'cleanfiles' module instead

## Inputs

- Aligned BAM file of mapped reads produced by NEAT's 'map' module (bwa or bowtie)
- User-specified filtering parameters from AdvancedSettings.txt (duplicate-read management policy, minimum and maximum fragment sizes)

## Outputs

- Filtered BAM file
-  accompanying .bai index file

## How to apply

Read the user's filtering parameters from the centralized AdvancedSettings.txt file, specifically the duplicate-read management policy and the minimum and maximum fragment sizes. Use samtools view to filter the aligned BAM according to the configured fragment-size boundaries, then apply samtools rmdup to remove or manage duplicate reads per the specified policy. Sort the filtered alignments with samtools sort and generate the .bai index with samtools index. The rationale is that NEAT's 'filter' module sits after 'map' and before 'peakcalling', so peak-calling algorithms receive only boundary-compliant, duplicate-managed, coordinate-sorted alignments; validation requires a valid .bam output accompanied by its .bai index.

## Related tools

- **SAMtools** (Provides view, rmdup, sort, and index commands that implement fragment-size filtering, duplicate management, coordinate sorting, and BAM indexing in NEAT's 'filter' module)
- **NEAT** (Framework that exposes the filter module as an automated pipeline step between 'map' and 'peakcalling', parameterized via AdvancedSettings.txt) — https://github.com/pschorderet/NEAT

## Examples

```
samtools view in.bam | samtools rmdup - - | samtools sort - filtered; samtools index filtered.bam
```

## Evaluation signals

- The filtered .bam output exists at the expected project folder path
- The .bam file parses as a valid BAM file
- A matching .bai index file was generated alongside the BAM
- All retained alignments fall within the minimum/maximum fragment-size boundaries set in AdvancedSettings.txt
- Duplicate reads were removed or managed according to the policy specified in AdvancedSettings.txt

## Limitations

- Filtering parameters (duplicate-read policy, fragment-size thresholds) are fixed per project in the centralized AdvancedSettings.txt; per-sample overrides are not described
- The workflow was demonstrated on truncated H3K4me3 ChIPseq and RNAseq test datasets from mouse embryonic stem cells
- NEAT is documented as still under development ('If you encounter any bug, please contact us') and its operating system support is limited to Mac OSx

## Evidence

- [methods] The ‘filter’ module allows the user to specify filtering parameters (AdvancedSettings.txt) including how to manage duplicate reads, minimum and maximum size of fragments: "The ‘filter’ module allows the user to specify filtering parameters (AdvancedSettings.txt) including how to manage duplicate reads, minimum and maximum size of fragments"
- [intro] NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc): "NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc)"
- [abstract] NEAT projects are built around a unique and centralized file containing sample names, replicates, conditions, antibodies, alignment-, filtering- and peak calling parameters: "NEAT projects are built around a unique and centralized file containing sample names, replicates, conditions, antibodies, alignment-, filtering- and peak calling parameters"
- [readme] NEAT can be run either on a cluster (qsub or bsub) via the command line or directly through the respective applications.: "NEAT can be run either on a cluster (qsub or bsub) via the command line or directly through the respective applications."
