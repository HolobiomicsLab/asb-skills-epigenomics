---
name: pet-count-percentile-filtering
description: Use when calling chromatin contact domains from CTCF ChIA-PET loop-calling
  output and you need to reduce noisy, low-support loops before merging intervals
  of continuous loop coverage.
when_to_use_negative:
- Input is raw PET reads or pre-loop-calling data — peak calling and loop calling
  must be completed first
- The goal is enhancer-promoter loop annotation, which uses a different filter (PET
  count ≥3 and peak support per loop) rather than the 67th-percentile domain filter
- Loops lack peak-support annotations for both anchors (e.g., non-CTCF factor data
  without matched peak calls), since the filter requires both-anchor peak support
edam_operation: http://edamontology.org/operation_3902
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: ChIA-PIPE
  role: Automated pipeline that performs peak calling and loop calling and applies
    the percentile-based CCD-calling filter
  repo: https://github.com/TheJacksonLaboratory/ChIA-PIPE
- name: CPU (ChIA-PET Utilities)
  role: Modularized executables integrated by ChIA-PIPE for processing ChIA-PET data
    underlying loop and peak outputs
  repo: https://github.com/cheehongsg/CPU
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1126/sciadv.aay2078
    title: 'ChIA-PIPE: A fully automated pipeline for comprehensive ChIA-PET data
      analysis and visualization'
  source_content_hash: sha256:a3296bb411da91faf5d8b04a3d5a922d225718ed0dff7dccf527dd8879bf033f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TheJacksonLaboratory/ChIA-PIPE
  tools:
  - CPU (ChIA-PET Utilities)
  - ChIA-PIPE
  edam_operation: http://edamontology.org/operation_3902
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# PET-count percentile filtering for CCD calling

## Summary

A filtering strategy within ChIA-PIPE's chromatin contact domain (CCD) calling that retains only CTCF ChIA-PET loops with both anchors supported by called peaks and only the top one-third of loops by PET count (67th-percentile cutoff), then merges surviving loops into ≥25 kb domains.

## When to use

Use when calling chromatin contact domains from CTCF ChIA-PET loop-calling output and you need to reduce noisy, low-support loops before merging intervals of continuous loop coverage.

## When NOT to use

- Input is raw PET reads or pre-loop-calling data — peak calling and loop calling must be completed first
- The goal is enhancer-promoter loop annotation, which uses a different filter (PET count ≥3 and peak support per loop) rather than the 67th-percentile domain filter
- Loops lack peak-support annotations for both anchors (e.g., non-CTCF factor data without matched peak calls), since the filter requires both-anchor peak support

## Inputs

- ChIA-PIPE loop-calling output (PET clusters with genomic coordinates and PET counts)
- Peak-support annotations for loop anchors (CTCF peaks from ChIA-PIPE peak calling)

## Outputs

- BED file (chromosome, start, end) of chromatin contact domains of at least 25 kb with continuous loop coverage

## How to apply

Load the ChIA-PIPE loop-calling output with each PET cluster's genomic coordinates, PET count, and anchor peak-support annotations. First filter the loop set to retain only loops whose both anchors overlap called peaks (both-anchor peak support). Next compute the 67th percentile of PET counts across the retained loops and keep only the top one-third of loops at or above that cutoff. Merge the retained loops along each chromosome into maximal intervals of continuous loop coverage with no gaps or break points, and retain merged intervals of at least 25 kb as chromatin contact domains written to a BED file with chromosome, start, and end columns.

## Related tools

- **ChIA-PIPE** (Automated pipeline that performs peak calling and loop calling and applies the percentile-based CCD-calling filter) — https://github.com/TheJacksonLaboratory/ChIA-PIPE
- **CPU (ChIA-PET Utilities)** (Modularized executables integrated by ChIA-PIPE for processing ChIA-PET data underlying loop and peak outputs) — https://github.com/cheehongsg/CPU

## Examples

```
git clone https://github.com/TheJacksonLaboratory/ChIA-PIPE.git; bash chia_pipe-master/0.chia_pipe_shell.sh -c chia_pipe-master/example_config_file.sh
```

## Evaluation signals

- Every emitted CCD interval spans at least 25,000 bp
- The loop set used for merging consists exclusively of both-anchor-peak-supported loops
- Retained loops are at or above the 67th-percentile PET-count cutoff (top one-third by PET count)
- Merged intervals show continuous loop coverage with no gaps or break points along each chromosome
- Output BED file contains exactly three columns: chromosome, start, end

## Limitations

- The filter is defined for CTCF ChIA-PET loops with peak-annotated anchors; applicability to other factors or data types lacking matched peak calls is not established in this context
- Only the top one-third of loops by PET count survive, so weaker but genuine loops are excluded from domain definitions
- This article section does not state a ChIA-PIPE version, Docker image, or reference genome build for the CCD-calling step

## Evidence
