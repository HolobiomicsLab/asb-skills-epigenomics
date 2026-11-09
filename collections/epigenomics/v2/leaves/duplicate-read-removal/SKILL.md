---
name: duplicate-read-removal
description: Use when a mapped-read BAM from a NEAT ChIPseq/RNAseq pipeline step must
  be deduplicated according to user-defined duplicate-management settings in AdvancedSettings.txt.
when_to_use_negative:
- Input is raw FASTQ rather than an aligned BAM — alignment (NEAT 'map') must happen
  first
- The library protocol requires retaining duplicates (e.g. ultra-low-input where amplification
  is expected) and the AdvancedSettings.txt policy explicitly keeps them
- Peak calling has already been performed; filtering precedes peakcalling in NEAT's
  pipeline order
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: SAMtools
  role: 'Executes the filter step: samtools view for fragment-size boundaries, samtools
    rmdup for duplicate management, samtools sort and samtools index on the output'
- name: NEAT
  role: Pipeline framework whose 'filter' module (between 'map' and 'peakcalling')
    reads AdvancedSettings.txt parameters and invokes the samtools commands
  repo: https://github.com/pschorderet/NEAT
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-016-0902-3
    title: 'NEAT: a framework for building fully automated NGS pipelines and analyses'
  source_content_hash: sha256:0bcd74430b8d5f7ed40d30e44edc4ce7730899222c0f84ec0f62f7a54b59b850
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pschorderet/NEAT
  tools:
  - NEAT
  - SAMtools
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# duplicate-read-removal

## Summary

Removes or manages PCR/optical duplicate reads from aligned BAM files using samtools rmdup, as implemented in NEAT's 'filter' module within its ChIPseq/RNAseq pipeline.

## When to use

Use when a mapped-read BAM from a NEAT ChIPseq/RNAseq pipeline step must be deduplicated according to user-defined duplicate-management settings in AdvancedSettings.txt.

## When NOT to use

- Input is raw FASTQ rather than an aligned BAM — alignment (NEAT 'map') must happen first
- The library protocol requires retaining duplicates (e.g. ultra-low-input where amplification is expected) and the AdvancedSettings.txt policy explicitly keeps them
- Peak calling has already been performed; filtering precedes peakcalling in NEAT's pipeline order

## Inputs

- aligned, coordinate-sorted .bam file from NEAT's 'map' module (bwa or bowtie output)
- duplicate-read management policy from AdvancedSettings.txt
- minimum and maximum fragment size thresholds from AdvancedSettings.txt

## Outputs

- filtered .bam file with duplicates removed/managed
- matching .bai index
- sorted BAM ready for NEAT's 'peakcalling' module

## How to apply

Read the duplicate-read management policy, along with minimum and maximum fragment sizes, that the user specified in the central AdvancedSettings.txt file. First apply samtools view to restrict the aligned BAM to the configured fragment-size boundaries, then run samtools rmdup to remove or manage duplicate reads per the specified policy. Sort the deduplicated alignments with samtools sort and generate the index with samtools index, producing the filtered .bam with its .bai companion. This step sits between NEAT's 'map' module (bwa or bowtie alignment) and 'peakcalling', so its output feeds directly into peak calling where duplicate inflates would bias enrichment.

## Related tools

- **SAMtools** (Executes the filter step: samtools view for fragment-size boundaries, samtools rmdup for duplicate management, samtools sort and samtools index on the output)
- **NEAT** (Pipeline framework whose 'filter' module (between 'map' and 'peakcalling') reads AdvancedSettings.txt parameters and invokes the samtools commands) — https://github.com/pschorderet/NEAT

## Examples

```
samtools view -q 20 in.bam | samtools rmdup - filtered.bam && samtools sort filtered.bam -o filtered.sorted.bam && samtools index filtered.sorted.bam
```

## Evaluation signals

- Filtered .bam output file exists and parses as a valid BAM file
- A corresponding .bai index was generated alongside the filtered BAM
- Duplicate-read count in the filtered BAM is reduced relative to the input BAM, consistent with the chosen rmdup policy
- All retained reads fall within the min/max fragment sizes configured in AdvancedSettings.txt
- The filtered BAM is coordinate-sorted so downstream peakcalling can consume it directly

## Limitations

- NEAT was developed for Mac OSx, so running the filter module elsewhere is not explicitly supported
- NEAT is still under development and may contain bugs (per the README)
- Filtering parameters are constrained to those expressible in AdvancedSettings.txt (duplicate policy and fragment size bounds); more complex dedup strategies are not offered

## Evidence

- [methods] The ‘filter’ module allows the user to specify filtering parameters (AdvancedSettings.txt) including how to manage duplicate reads, minimum and maximum size of fragments: "The ‘filter’ module allows the user to specify filtering parameters (AdvancedSettings.txt) including how to manage duplicate reads, minimum and maximum size of fragments"
- [intro] NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc): "NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc)"
- [abstract] NEAT projects are built around a unique and centralized file containing sample names, replicates, conditions, antibodies, alignment-, filtering- and peak calling parameters: "NEAT projects are built around a unique and centralized file containing sample names, replicates, conditions, antibodies, alignment-, filtering- and peak calling parameters"
- [readme] NEAT has been developed as four main modules: [ 1 ] Create a new project [ 2 ] Build a pipeline: "NEAT has been developed as four main modules: [ 1 ]       Create a new project [ 2 ]       Build a pipeline"
