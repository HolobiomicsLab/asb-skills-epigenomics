---
name: chromatin-contact-domain-calling
description: Use when ChIA-PIPE loop-calling output (PET clusters with coordinates,
  PET counts, and anchor peak-support flags) from CTCF ChIA-PET data must be converted
  into chromatin contact domain (CCD) intervals.
when_to_use_negative:
- Input is raw ChIA-PET reads — this skill operates downstream of linker filtering,
  read mapping, peak calling, and loop calling, not on FASTQ data
- The goal is enhancer-promoter loop annotation or allele-specific peak/loop resolution,
  which ChIA-PIPE implements as separate analyses with different criteria (e.g., PET
  count >=3 loops, phased SNPs)
- Loops lack anchor peak-support annotations, so the both-anchor-supported filter
  cannot be evaluated
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: ChIA-PIPE
  role: Automated pipeline that performs upstream peak and loop calling and whose
    loop-annotation workflow includes the CCD-calling step reconstructed here
  repo: https://github.com/TheJacksonLaboratory/ChIA-PIPE
- name: CPU
  role: ChIA-PET Utilities, the collection of modularized executables integrated by
    ChIA-PIPE for processing ChIA-PET data into interactions
  repo: https://github.com/cheehongsg/CPU
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1126/sciadv.aay2078
    title: 'ChIA-PIPE: A fully automated pipeline for comprehensive ChIA-PET data
      analysis and visualization'
  source_content_hash: sha256:31089a2318bf818dfbab1c71344205f1803b64e04b9ba9aba07b5e7d928ceb22
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TheJacksonLaboratory/ChIA-PIPE
  tools:
  - CPU
  - ChIA-PIPE
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# chromatin-contact-domain-calling

## Summary

Reconstructs the ChIA-PIPE CCD-calling algorithm, which defines chromatin contact domains from CTCF ChIA-PET data by merging high-confidence, both-anchor-peak-supported loops into contiguous genomic intervals. This turns discrete loop calls into domain-scale annotations without additional sequencing input.

## When to use

Use when ChIA-PIPE loop-calling output (PET clusters with coordinates, PET counts, and anchor peak-support flags) from CTCF ChIA-PET data must be converted into chromatin contact domain (CCD) intervals.

## When NOT to use

- Input is raw ChIA-PET reads — this skill operates downstream of linker filtering, read mapping, peak calling, and loop calling, not on FASTQ data
- The goal is enhancer-promoter loop annotation or allele-specific peak/loop resolution, which ChIA-PIPE implements as separate analyses with different criteria (e.g., PET count >=3 loops, phased SNPs)
- Loops lack anchor peak-support annotations, so the both-anchor-supported filter cannot be evaluated

## Inputs

- ChIA-PIPE loop-calling output for CTCF ChIA-PET data (PET clusters with genomic coordinates, PET counts, and anchor peak-support annotations)
- ChIA-PIPE-called binding peak coordinates used to annotate loop anchors

## Outputs

- BED file of chromatin contact domains with chromosome, start, and end columns; each interval spans at least 25 kb of continuous loop coverage

## How to apply

Load the ChIA-PIPE loop-calling output and filter the loop set to retain only loops whose both anchors overlap called peaks (both-anchor peak support). From the retained loops, compute the 67th percentile of PET counts and keep only the top one-third of loops at or above that cutoff, since ChIA-PIPE deliberately retains only the strongest loops before merging. Merge the retained loops along each chromosome into maximal intervals of continuous loop coverage with no gaps or break points. Retain merged intervals of at least 25 kb as chromatin contact domains and write them to a BED file with chromosome, start, and end columns. The rationale is that domains built exclusively from top-quartile-by-PET-count, peak-anchored loops are robust against weak or spurious interactions that survive loop calling (PET count >=3 with peak support).

## Related tools

- **ChIA-PIPE** (Automated pipeline that performs upstream peak and loop calling and whose loop-annotation workflow includes the CCD-calling step reconstructed here) — https://github.com/TheJacksonLaboratory/ChIA-PIPE
- **CPU** (ChIA-PET Utilities, the collection of modularized executables integrated by ChIA-PIPE for processing ChIA-PET data into interactions) — https://github.com/cheehongsg/CPU

## Examples

```
bash chia_pipe-master/0.chia_pipe_shell.sh -c chia_pipe-master/example_config_file.sh
```

## Evaluation signals

- Every emitted CCD interval in the output BED file spans at least 25,000 bp
- The loop set used for merging consists exclusively of both-anchor-peak-supported loops
- The retained loop set contains only loops at or above the 67th-percentile PET-count cutoff (top one-third)
- Each CCD interval covers loops continuously, with no gaps or break points within an interval

## Limitations

- The percentile cutoff (67th percentile) is dataset-relative: the retained PET-count threshold depends on the distribution of the input loop set, so CCD boundaries are not comparable across libraries run with different loop sets
- Only loops with both anchors supported by called peaks enter CCD construction, so domains cannot form from loops with unsupported anchors
- The article does not state a versioned, separately hosted script for this step, so reproduction relies on re-implementing the described procedure

## Evidence
