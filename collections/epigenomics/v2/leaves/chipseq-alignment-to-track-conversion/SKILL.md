---
name: chipseq-alignment-to-track-conversion
description: Use when filtered .bam/.bai ChIPseq alignments (e.g. post-samtools filter
  output) must be turned into genome-browser track files and GRanges objects for visualization
  and downstream analysis.
when_to_use_negative:
- Input is still raw FASTQ or an unfiltered .bam that has not passed the NEAT 'filter'
  module (duplicate-read and fragment-size filtering) — run filtering and peakcalling
  first
- The goal is exploratory QC or metagene/DEG analysis rather than track-file generation
  (use the NEAT QC or 'Analyze project' modules instead)
edam_operation: http://edamontology.org/operation_3208
tools:
- name: NEAT
  role: Hosts the Wig, Bigwig and GRanges pipeline modules as automated, double-clickable
    pipeline steps
  repo: https://github.com/pschorderet/NEAT
- name: SAMtools
  role: Executes the upstream filtering (view/sort/rmdup/index) whose output .bam/.bai
    feeds track generation
- name: R
  role: Environment in which the GRanges module converts alignments into compact GRanges
    objects
- name: Bioconductor
  role: Consortium providing the GRanges infrastructure for downstream-analysis objects
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-016-0902-3
    title: 'NEAT: a framework for building fully automated NGS pipelines and analyses'
  source_content_hash: sha256:1b911069c9a2b1ee9d0410bc15792587579e009809debd4da393830da36fe37f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pschorderet/NEAT
  tools:
  - Bioconductor
  - NEAT
  - R
  - SAMtools
  edam_operation: http://edamontology.org/operation_3208
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# chipseq-alignment-to-track-conversion

## Summary

Converts NEAT's processed, filtered .bam/.bai ChIPseq alignments into visualization-ready genome-browser tracks — compressed small-size wiggle (.wig) files with bigwig counterparts — plus compact R/Bioconductor GRanges objects for downstream analysis.

## When to use

Use when filtered .bam/.bai ChIPseq alignments (e.g. post-samtools filter output) must be turned into genome-browser track files and GRanges objects for visualization and downstream analysis.

## When NOT to use

- Input is still raw FASTQ or an unfiltered .bam that has not passed the NEAT 'filter' module (duplicate-read and fragment-size filtering) — run filtering and peakcalling first
- The goal is exploratory QC or metagene/DEG analysis rather than track-file generation (use the NEAT QC or 'Analyze project' modules instead)

## Inputs

- Filtered .bam/.bai alignment files for ChIPseq data (e.g. truncated H3K4me3 ChIPseq test dataset from mouse embryonic stem cells), produced by the NEAT 'filter' module using samtools view/sort/rmdup/index
- AdvancedSettings.txt containing the per-base-pair bin size for wiggle binning

## Outputs

- Compressed small-size wiggle (.wig) track files binned at the configured genome-wide bin size
- Bigwig-format counterpart of the wiggle tracks for genome-browser loading (IGV/UCSC)
- Compact GRanges objects (R/Bioconductor) for downstream analysis

## How to apply

Run this step after the NEAT 'filter' module, whose parameters (duplicate-read management, minimum/maximum fragment size) are set in the centralized AdvancedSettings.txt file and executed via samtools view/sort/rmdup/index. Read the per-base-pair bin size from AdvancedSettings.txt, since wiggle compression is driven by genome-wide binning. Implement the Wig module to bin the alignment signal genome-wide at the configured bin size and write compressed small-size wiggle (.wig) files, then run the Bigwig module to emit the bigwig-format counterpart for genome-browser use. Separately, run the GRanges module in R/Bioconductor to convert the alignments into significantly smaller GRanges objects required by downstream analysis steps.

## Related tools

- **NEAT** (Hosts the Wig, Bigwig and GRanges pipeline modules as automated, double-clickable pipeline steps) — https://github.com/pschorderet/NEAT
- **SAMtools** (Executes the upstream filtering (view/sort/rmdup/index) whose output .bam/.bai feeds track generation)
- **R** (Environment in which the GRanges module converts alignments into compact GRanges objects)
- **Bioconductor** (Consortium providing the GRanges infrastructure for downstream-analysis objects)

## Evaluation signals

- Produced .wig files fall in the small size range reported (~4–6 Mb, versus ~4–6 Gb for the .bam input)
- Resulting GRanges objects are ~40–60 Mb, i.e. dramatically smaller than the source .bam files
- Bigwig output is emitted alongside the wiggle tracks and both are loadable in a genome browser such as IGV
- Binning resolution matches the per-base-pair bin size read from AdvancedSettings.txt
- Track generation occurs after filtering and peakcalling in the NEAT module ordering (filter → peakcalling → wig)

## Limitations

- NEAT operates on Mac OSx; no Linux/Windows support stated in the article
- NEAT was still under development; the README asks users to report bugs, and no changelog or version number is available
- Track outputs are part of a double-clickable pipeline whose concrete run commands are not documented in the article or README, so invocation details must be recovered from the Vignette tutorials

## Evidence

- [intro] NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc): "NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc)"
- [methods] GRanges module creates smaller objects: "The ‘GRanges’ module creates significantly smaller GRanges objects (compared to bam files), which are necessary for downstream analysis"
- [methods] filter module parameters in AdvancedSettings.txt: "The ‘filter’ module allows the user to specify filtering parameters (AdvancedSettings.txt) including how to manage duplicate reads, minimum and maximum size of fragments"
