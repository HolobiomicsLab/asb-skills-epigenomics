---
name: peak-supported-anchor-filtering
description: Use when calling chromatin contact domains (CCDs) from CTCF ChIA-PET
  loops and you need to restrict the loop set to high-confidence, peak-supported interactions
  before merging into domain intervals.
when_to_use_negative:
- Calling peaks or loops themselves — this filter operates downstream on already-called
  ChIA-PIPE loops and peaks
- Annotating individual enhancer-promoter loops, which uses a different criterion
  (PET count ≥3 and peak support) and a custom Python classification script
- Data types where anchor peak support is not expected (e.g., non-ChIA-PET contact
  data without corresponding binding peaks)
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0607
- http://edamontology.org/topic_3169
tools:
- name: ChIA-PIPE
  role: Automated pipeline whose loop-calling and CCD-calling modules implement the
    both-anchor peak-support filter, 67th-percentile PET-count cutoff, and ≥25 kb
    continuous-coverage merging
  repo: https://github.com/TheJacksonLaboratory/ChIA-PIPE
- name: CPU
  role: ChIA-PET Utilities collection of modularized executables that produce the
    deduplicated, peak-supported loop data consumed by the CCD workflow
  repo: https://github.com/cheehongsg/CPU
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1126/sciadv.aay2078
    title: 'ChIA-PIPE: A fully automated pipeline for comprehensive ChIA-PET data
      analysis and visualization'
  source_content_hash: sha256:7f9af58ac4a8f6da6c57d401f6fbde7239e6d3bc2ea2810325edf84d0ea48fa6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TheJacksonLaboratory/ChIA-PIPE
  tools:
  - CPU
  - ChIA-PIPE
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0607
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# peak-supported-anchor-filtering

## Summary

A filter step within ChIA-PIPE's chromatin contact domain (CCD) calling workflow that retains only chromatin loops whose both anchors are supported by called binding peaks, as the basis for defining CCDs from CTCF ChIA-PET data.

## When to use

Use when calling chromatin contact domains (CCDs) from CTCF ChIA-PET loops and you need to restrict the loop set to high-confidence, peak-supported interactions before merging into domain intervals.

## When NOT to use

- Calling peaks or loops themselves — this filter operates downstream on already-called ChIA-PIPE loops and peaks
- Annotating individual enhancer-promoter loops, which uses a different criterion (PET count ≥3 and peak support) and a custom Python classification script
- Data types where anchor peak support is not expected (e.g., non-ChIA-PET contact data without corresponding binding peaks)

## Inputs

- ChIA-PIPE loop-calling output with PET cluster genomic coordinates, PET counts, and anchor peak-support annotations (BED-style loop file)
- Called binding peaks (e.g., CTCF ChIP-seq/ChIA-PET peaks) used to annotate loop anchors

## Outputs

- Filtered loop set (both-anchor peak-supported loops above the 67th-percentile PET-count cutoff)
- CCD BED file with chromosome, start, and end columns, each interval ≥25 kb of continuous loop coverage

## How to apply

Load the ChIA-PIPE loop-calling output, reading each PET cluster's genomic coordinates, PET count, and anchor peak-support annotations, then filter to retain only loops whose both anchors overlap called peaks. Apply the PET-count percentile cutoff: compute the 67th percentile of PET counts across the retained loops and keep only the top one-third of loops at or above that cutoff. Merge the surviving loops along each chromosome into maximal intervals of continuous loop coverage with no gaps or break points, and retain merged intervals of at least 25 kb as CCDs. The rationale is that requiring both-anchor peak support plus the 67th-percentile PET-count cutoff restricts CCD definition to the strongest, biologically grounded CTCF-mediated loops rather than all called interactions.

## Related tools

- **ChIA-PIPE** (Automated pipeline whose loop-calling and CCD-calling modules implement the both-anchor peak-support filter, 67th-percentile PET-count cutoff, and ≥25 kb continuous-coverage merging) — https://github.com/TheJacksonLaboratory/ChIA-PIPE
- **CPU** (ChIA-PET Utilities collection of modularized executables that produce the deduplicated, peak-supported loop data consumed by the CCD workflow) — https://github.com/cheehongsg/CPU

## Examples

```
git clone https://github.com/TheJacksonLaboratory/ChIA-PIPE.git && bash chia_pipe-master/0.chia_pipe_shell.sh -c chia_pipe-master/example_config_file.sh
```

## Evaluation signals

- Every emitted CCD interval spans at least 25,000 bp
- The loop set used for merging consists exclusively of loops whose both anchors overlap called peaks
- The retained loop set is exactly the top one-third of peak-supported loops with respect to PET count (loops at or above the 67th-percentile PET-count cutoff)
- Each merged CCD region has continuous loop coverage with no gaps or break points along the chromosome
- Output BED file contains only chromosome, start, and end columns per CCD

## Limitations

- Requires reliable peak calls at both anchors; loops lacking peak support at either anchor are discarded and cannot contribute to CCDs
- The 67th-percentile PET-count cutoff and 25 kb minimum CCD length are fixed defaults of the paper's method; datasets with few peak-supported loops may yield few or no CCDs
- No version, Docker image identifier, or reference genome build is stated for the CCD-calling step in the discussion section

## Evidence
