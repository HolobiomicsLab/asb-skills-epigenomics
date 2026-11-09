---
name: chromatin-accessibility-data-processing
description: Use when paired-end chromatin accessibility reads (BAM from MNase-seq
  or ATAC-seq) must be classified by fragment length and converted to per-base midpoint-count
  tracks for nucleosome/TF occupancy modeling.
when_to_use_negative:
- Input is single-end sequencing or a genome without paired-end fragment insert sizes,
  since fragment-length classification (127–187 bp vs <80 bp) cannot be computed
- The goal is peak calling or footprint labeling rather than building HMM observables
  — the article treats MACS2 + FIMO as an alternative comparison approach, not this
  preprocessing route
- Data are already processed midpoint-count tracks or bedGraph observables; re-running
  classification would double-apply the Tn5 correction or re-threshold fragments
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: RoboCOP
  role: Pipeline that partitions fragments into nucFrags/shortFrags, computes midpoint-count
    signals, and consumes them as HMM observables; installed by compiling librobocop.so
    and setting cshared in config.ini
  repo: https://github.com/HarteminkLab/RoboCOP
- name: GCC
  role: Used via 'bash gccCompile' to build the shared library librobocop.so that
    the analysis pipeline invokes
  repo: https://github.com/HarteminkLab/RoboCOP
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkab553
    title: 'RoboCOP: jointly computing chromatin occupancy profiles for numerous factors
      from chromatin accessibility data'
  source_content_hash: sha256:8721ac973e85f11ab236c6cc0b84a760e79ac4e4ea65c25706a72fd0fa862ca7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HarteminkLab/RoboCOP
  tools:
  - GCC
  - NucleoATAC
  - RoboCOP
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# chromatin-accessibility-data-processing

## Summary

Transforms raw paired-end MNase-seq and ATAC-seq fragment alignments into the one-dimensional genomic midpoint-count signals (nucFrags and shortFrags tracks) that RoboCOP's multivariate HMM consumes as observables.

## When to use

Use when paired-end chromatin accessibility reads (BAM from MNase-seq or ATAC-seq) must be classified by fragment length and converted to per-base midpoint-count tracks for nucleosome/TF occupancy modeling.

## When NOT to use

- Input is single-end sequencing or a genome without paired-end fragment insert sizes, since fragment-length classification (127–187 bp vs <80 bp) cannot be computed
- The goal is peak calling or footprint labeling rather than building HMM observables — the article treats MACS2 + FIMO as an alternative comparison approach, not this preprocessing route
- Data are already processed midpoint-count tracks or bedGraph observables; re-running classification would double-apply the Tn5 correction or re-threshold fragments

## Inputs

- Aligned paired-end MNase-seq fragment alignments (BAM; GEO GSE153609)
- Aligned paired-end ATAC-seq fragment alignments (BAM; GEO GSE66386)
- RoboCOP configuration file (config.ini) with cshared path to librobocop.so

## Outputs

- fragment_midpoint_signals.bedgraph containing nucFrags and shortFrags midpoint-count tracks
- 1-D per-base midpoint-count signal tracks at single-nucleotide resolution, one per fragment class
- RoboCOP-ready observables for the multivariate HMM

## How to apply

Load aligned paired-end fragment alignments (BAM) using the RoboCOP analysis pipeline, then partition fragments by insert size: fragments within 157±30 bp (127–187 bp) are labeled nucFrags and fragments shorter than 80 bp are labeled shortFrags. For each fragment compute the midpoint ((start+end)/2) and accumulate midpoint counts per genomic base into a 1-D signal track at single-nucleotide resolution, one track per class. For ATAC-seq data (GEO GSE66386), subtract 8 bp from each fragment length to compensate for the Tn5 transposase 9-bp insertion duplication before computing midpoints; MNase-seq data (GEO GSE153609) requires no such offset. Export the resulting nucFrags and shortFrags midpoint-count tracks as bedGraph files and point the compiled RoboCOP engine (librobocop.so, built via bash gccCompile) at them through the cshared entry in config.ini.

## Related tools

- **RoboCOP** (Pipeline that partitions fragments into nucFrags/shortFrags, computes midpoint-count signals, and consumes them as HMM observables; installed by compiling librobocop.so and setting cshared in config.ini) — https://github.com/HarteminkLab/RoboCOP
- **GCC** (Used via 'bash gccCompile' to build the shared library librobocop.so that the analysis pipeline invokes) — https://github.com/HarteminkLab/RoboCOP

## Examples

```
git clone https://github.com/HarteminkLab/RoboCOP.git && cd RoboCOP/robocop && bash gccCompile && ls "$(pwd)/librobocop.so"
```

## Evaluation signals

- fragment_midpoint_signals.bedgraph exists and contains both nucFrags and shortFrags tracks
- Fragment-length thresholds used for classification match 157±30 bp (127–187 bp) for nucFrags and <80 bp for shortFrags
- The −8 bp Tn5 transposase 9-bp insertion duplication offset is applied to ATAC-seq fragment lengths before midpoint computation
- Midpoint tracks are per-base counts at single-nucleotide resolution with midpoint defined as (start+end)/2
- Downstream RoboCOP runs succeed, confirming cshared points to a valid absolute path of librobocop.so

## Limitations

- Fragment-length cutoffs (157±30 bp and <80 bp) are tuned for yeast MNase-seq/ATAC-seq and were not stated to generalize to other organisms
- ATAC-seq requires the −8 bp Tn5 correction; applying it to MNase-seq data (or omitting it for ATAC-seq) yields incorrect midpoints
- RoboCOP accuracy and comprehensiveness improve with sequencing depth; shallow data yield noisier midpoint signals

## Evidence

- [supplementary] fragments with length within 157±30 bp (127–187 bp) are labeled nucFrags, and fragments shorter than 80 bp are labeled shortFrags: [evidence span withheld — anchor status: not_found]
