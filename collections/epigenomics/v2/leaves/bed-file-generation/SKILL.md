---
name: bed-file-generation
description: Use when producing BED interval outputs from ChIA-PET loop data — e.g.,
  calling chromatin contact domains from CTCF ChIA-PET loops as chromosome/start/end
  BED records after peak-support and PET-count filtering.
when_to_use_negative:
- Input is raw ChIA-PET reads or unprocessed PET clusters that have not yet gone through
  ChIA-PIPE's linker filtering, read mapping, peak calling, and loop calling stages
- The goal is annotating individual enhancer-promoter loops (PET count ≥3 with peak
  support) rather than merging loops into domain-scale BED intervals
- Loops lack both-anchor peak-support annotations, since the CCD workflow exclusively
  uses both-anchor-peak-supported loops
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: ChIA-PIPE
  role: Automated pipeline whose loop-calling and CCD-calling modules produce the
    filtered, merged genomic intervals written as BED records
  repo: https://github.com/TheJacksonLaboratory/ChIA-PIPE
- name: CPU
  role: ChIA-PET Utilities collection of modularized executables integrated by ChIA-PIPE
    to process ChIA-PET data from raw reads to interactions feeding the BED output
  repo: https://github.com/cheehongsg/CPU
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1126/sciadv.aay2078
    title: 'ChIA-PIPE: A fully automated pipeline for comprehensive ChIA-PET data
      analysis and visualization'
  source_content_hash: sha256:8043508565093f28ac3e8a58b60426bd3f2be17198e3dbdff1961be87276e5d8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TheJacksonLaboratory/ChIA-PIPE
  tools:
  - CPU
  - ChIA-PIPE
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# BED file generation

## Summary

Generation of BED-format interval files within ChIA-PIPE's chromatin contact domain (CCD) calling workflow, where merged CTCF loop-coverage regions are written as chromosome, start, end records. This skill covers the filtering and merging logic that determines which genomic intervals are emitted to the BED file.

## When to use

Use when producing BED interval outputs from ChIA-PET loop data — e.g., calling chromatin contact domains from CTCF ChIA-PET loops as chromosome/start/end BED records after peak-support and PET-count filtering.

## When NOT to use

- Input is raw ChIA-PET reads or unprocessed PET clusters that have not yet gone through ChIA-PIPE's linker filtering, read mapping, peak calling, and loop calling stages
- The goal is annotating individual enhancer-promoter loops (PET count ≥3 with peak support) rather than merging loops into domain-scale BED intervals
- Loops lack both-anchor peak-support annotations, since the CCD workflow exclusively uses both-anchor-peak-supported loops

## Inputs

- ChIA-PIPE loop-calling output (PET cluster genomic coordinates, PET count, anchor peak-support annotations per loop)
- Called CTCF binding peaks used for anchor overlap support

## Outputs

- BED file of chromatin contact domains with chromosome, start, and end columns; every interval ≥ 25,000 bp of continuous loop coverage

## How to apply

Load the ChIA-PIPE loop-calling output with each PET cluster's genomic coordinates, PET count, and anchor peak-support annotations. Filter to retain only loops whose both anchors overlap called peaks (both-anchor peak support), then compute the 67th percentile of PET counts across retained loops and keep only the top one-third of loops at or above that cutoff. Merge the retained loops along each chromosome into maximal intervals of continuous loop coverage with no gaps or break points. Emit merged intervals of at least 25 kb as chromatin contact domains in a BED file with chromosome, start, and end columns. The 25-kb minimum length mirrors the paper's definition that CCDs are genomic regions of at least 25 kb with continuous coverage by loops.

## Related tools

- **ChIA-PIPE** (Automated pipeline whose loop-calling and CCD-calling modules produce the filtered, merged genomic intervals written as BED records) — https://github.com/TheJacksonLaboratory/ChIA-PIPE
- **CPU** (ChIA-PET Utilities collection of modularized executables integrated by ChIA-PIPE to process ChIA-PET data from raw reads to interactions feeding the BED output) — https://github.com/cheehongsg/CPU

## Examples

```
bash chia_pipe-master/0.chia_pipe_shell.sh -c chia_pipe-master/example_config_file.sh
```

## Evaluation signals

- Every emitted CCD interval spans at least 25,000 bp
- The loop set used for merging consists exclusively of both-anchor-peak-supported loops
- Retained loops all have PET counts at or above the 67th-percentile cutoff (top one-third by PET count)
- Merged intervals contain continuous loop coverage with no gaps or break points along each chromosome
- BED file contains exactly chromosome, start, and end columns per record

## Limitations

- CCD calling is only supported for loops with both anchors supported by a binding peak (e.g., CTCF ChIA-PET); loops lacking both-anchor peak support are excluded
- The 67th-percentile PET-count and 25-kb minimum-length parameters are fixed defaults of the described workflow and may not suit datasets with different loop-count distributions
- Intervals shorter than 25 kb are discarded even if they show continuous loop coverage

## Evidence

- [discussion] CCDs are then defined as a genomic region of at least 25 kb in length that has continuous coverage by loops with no gaps or break points.: "CCDs are then defined as a genomic region of at least 25 kb in length that has continuous coverage by loops with no gaps or break points."
- [discussion] Top one-third loops by PET count retained via percentile cutoff: "using a percentile cutoff (67th percentile), only the top one-third of loops with respect to PET count are retained."
- [abstract] ChIA-PIPE calls chromatin contact domains: "ChIA-PIPE calls chromatin contact domains, resolves allele-specific peaks and loops, and annotates enhancer-promoter loops"
- [readme] Launch ChIA-PIPE via its shell script: "bash chia_pipe-master/0.chia_pipe_shell.sh -c chia_pipe-master/example_config_file.sh"
