---
name: genomic-interval-merging
description: Use when peak- or loop-supported genomic intervals must be collapsed
  into contiguous domains, e.g., deriving CCDs from ChIA-PIPE CTCF loop BED output
  where overlapping/adjacent loops define continuous coverage.
when_to_use_negative:
- Input intervals are single-nucleotide or point features (e.g., SNPs) where a 25-kb
  minimum-length domain definition does not apply
- The goal is enhancer-promoter loop annotation, which uses the full loop set (PET
  count ≥3 and peak support) rather than percentile-filtered merged intervals
- Loop data lack anchor peak-support annotations, so both-anchor peak-support filtering
  cannot be performed
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: ChIA-PIPE
  role: Automated pipeline whose loop-calling output feeds CCD calling; merges both-anchor-peak-supported,
    top-third PET-count loops into ≥25 kb chromatin contact domains
  repo: https://github.com/TheJacksonLaboratory/ChIA-PIPE
- name: CPU
  role: ChIA-PET Utilities collection of modularized executables providing the loop-
    and peak-calling modules whose output is merged into CCDs
  repo: https://github.com/cheehongsg/CPU
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1126/sciadv.aay2078
    title: 'ChIA-PIPE: A fully automated pipeline for comprehensive ChIA-PET data
      analysis and visualization'
  source_content_hash: sha256:c026da65029f44b2f1cb656684f069f3b1625ac41b8a6d98d65f034524b97494
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TheJacksonLaboratory/ChIA-PIPE
  tools:
  - CPU
  - ChIA-PIPE
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# genomic-interval-merging

## Summary

Merges filtered genomic intervals (e.g., ChIA-PET loop anchors) along each chromosome into maximal regions of continuous coverage with no gaps, then applies a minimum-length filter — as used by ChIA-PIPE to call chromatin contact domains (CCDs) from CTCF ChIA-PET data.

## When to use

Use when peak- or loop-supported genomic intervals must be collapsed into contiguous domains, e.g., deriving CCDs from ChIA-PIPE CTCF loop BED output where overlapping/adjacent loops define continuous coverage.

## When NOT to use

- Input intervals are single-nucleotide or point features (e.g., SNPs) where a 25-kb minimum-length domain definition does not apply
- The goal is enhancer-promoter loop annotation, which uses the full loop set (PET count ≥3 and peak support) rather than percentile-filtered merged intervals
- Loop data lack anchor peak-support annotations, so both-anchor peak-support filtering cannot be performed

## Inputs

- ChIA-PIPE loop-calling output (PET clusters with genomic coordinates, PET counts, and anchor peak-support annotations)
- CTCF ChIA-PET peak calls used for both-anchor peak-support filtering

## Outputs

- BED file of chromatin contact domains with chromosome, start, and end columns, each spanning at least 25 kb

## How to apply

Start from the ChIA-PIPE loop-calling output, reading each PET cluster's coordinates, PET count, and anchor peak-support annotations, and retain only loops whose both anchors overlap called peaks (both-anchor peak support). Apply a percentile cutoff: compute the 67th percentile of PET counts across the retained loops and keep only the top one-third of loops at or above that cutoff, so that domain boundaries reflect the strongest interactions rather than background looping. Then merge the retained loops along each chromosome into maximal intervals of continuous loop coverage with no gaps or break points, and retain merged intervals of at least 25 kb as chromatin contact domains. Write the result as a BED file with chromosome, start, and end columns. The rationale is that CCDs represent regions densely spanned by high-confidence CTCF-mediated loops; merging collapses per-loop evidence into domain-level coordinates.

## Related tools

- **ChIA-PIPE** (Automated pipeline whose loop-calling output feeds CCD calling; merges both-anchor-peak-supported, top-third PET-count loops into ≥25 kb chromatin contact domains) — https://github.com/TheJacksonLaboratory/ChIA-PIPE
- **CPU** (ChIA-PET Utilities collection of modularized executables providing the loop- and peak-calling modules whose output is merged into CCDs) — https://github.com/cheehongsg/CPU

## Examples

```
git clone https://github.com/TheJacksonLaboratory/ChIA-PIPE.git
```

## Evaluation signals

- Every emitted CCD interval spans at least 25,000 bp
- The loop set used for merging consists exclusively of loops with both-anchor peak support
- All retained loops are at or above the 67th-percentile PET-count cutoff (top one-third of loops)
- Output is a valid BED file with exactly chromosome, start, and end columns
- No merged CCD contains internal gaps or break points in loop coverage on any chromosome

## Limitations

- Rejection of loops below the 67th-percentile PET-count cutoff means weaker loops do not contribute to CCDs, so domains may be shorter or split relative to those built from all loops
- The CCD definition requires continuous loop coverage with no gaps; a single uncovered base between loops breaks the domain
- The article section does not state a specific reference genome version or ChIA-PIPE version/commit for reproducibility of CCD coordinates

## Evidence
