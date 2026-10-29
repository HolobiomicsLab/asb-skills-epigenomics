---
name: pet-to-peak-interaction-mapping
description: Use when aligned, PCR-deduplicated ChIA-PET PETs in bedpe format must
  be assigned to MACS2-called anchor peak pairs to generate candidate intra-/inter-chromosomal
  loops.
when_to_use_negative:
- PETs have not yet been linker-trimmed, aligned (bwa) and PCR-duplicate-removed —
  bedpe2Interaction expects bedpe output from buildBedpe/removeDup
- No ChIP-seq-like peak list is available for the factor, since ChIA-PET2 loop calling
  depends on peak-calling sensitivity
- Input is not ChIA-PET PET data (e.g., Hi-C contact matrices or already-called loops)
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: ChIA-PET2
  role: Provides buildTagAlign and bedpe2Interaction to convert bedpe PETs to tagAlign
    and assign PETs to peak pairs as candidate interactions
  repo: https://github.com/GuipengLi/ChIA-PET2
- name: MACS2
  role: Calls the ChIP-seq-like anchor peaks (default -q 0.05) onto which PETs are
    mapped
  repo: https://github.com/macs3-project/MACS
- name: bedtools
  role: pairToBed -type both underlies the PET-to-peak-pair assignment and is used
    for anchor occupancy overlap checks
  repo: https://github.com/arq5x/bedtools2
- name: MICC
  role: Consumes the peak-pair interaction bedpe files via MICC2.R to estimate loop
    significance (default CUTOFFPET=2)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkw809
    title: 'ChIA-PET2: a versatile and flexible pipeline for ChIA-PET data analysis'
  source_content_hash: sha256:1a8fba8ddfba56143efe7bb2f4e100515c574563a990179168bcd7ba4ca6f27c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/GuipengLi/ChIA-PET2
  repo_url: https://github.com/GuipengLi/ChIA-PET2
  tools:
  - ChIA-PET2
  - MACS2
  - MICC
  - R
  - bedtools
  - bwa
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# pet-to-peak-interaction-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns ChIA-PET paired-end tags from a deduplicated bedpe file to ChIP-seq-like anchor peaks (called by MACS2 on buildTagAlign output) to form candidate peak-pair chromatin interactions for MICC significance testing.

## When to use

Use when aligned, PCR-deduplicated ChIA-PET PETs in bedpe format must be assigned to MACS2-called anchor peak pairs to generate candidate intra-/inter-chromosomal loops.

## When NOT to use

- PETs have not yet been linker-trimmed, aligned (bwa) and PCR-duplicate-removed — bedpe2Interaction expects bedpe output from buildBedpe/removeDup
- No ChIP-seq-like peak list is available for the factor, since ChIA-PET2 loop calling depends on peak-calling sensitivity
- Input is not ChIA-PET PET data (e.g., Hi-C contact matrices or already-called loops)

## Inputs

- Deduplicated PET records in bedpe format (from ChIA-PET2 buildBedpe/removeDup)
- MACS2-called anchor peak BED list built from the buildTagAlign tag file
- Slop/extension window parameters (default -S 100 bp, -E 500 bp)

## Outputs

- Candidate peak-pair interactions in bedpe format (interactions.intra.bedpe and interactions.inter.bedpe)
- PET count per candidate interaction for downstream MICC CUTOFFPET filtering

## How to apply

First convert the deduplicated bedpe to a tagAlign with ChIA-PET2's buildTagAlign and call ChIP-seq-like anchor peaks with MACS2 (ChIA-PET2 default '-M -q 0.05'), since ChIA-PET2 assumes loop anchors lie at ChIP-seq-like peaks. Then run bedpe2Interaction, which wraps pairToBed (-type both) so a candidate interaction is kept only when both PET ends fall near a peak, applying the default slop of 100 bp (-S) and 500 bp anchor extension (-E). The step emits interactions.intra.bedpe and interactions.inter.bedpe with per-pair PET counts, which downstream MICC2.R filters at CUTOFFPET (default 2) to estimate FDR for each loop. Users can adjust thresholds to balance sensitivity versus specificity of loop calling.

## Related tools

- **ChIA-PET2** (Provides buildTagAlign and bedpe2Interaction to convert bedpe PETs to tagAlign and assign PETs to peak pairs as candidate interactions) — https://github.com/GuipengLi/ChIA-PET2
- **MACS2** (Calls the ChIP-seq-like anchor peaks (default -q 0.05) onto which PETs are mapped) — https://github.com/macs3-project/MACS
- **bedtools** (pairToBed -type both underlies the PET-to-peak-pair assignment and is used for anchor occupancy overlap checks) — https://github.com/arq5x/bedtools2
- **MICC** (Consumes the peak-pair interaction bedpe files via MICC2.R to estimate loop significance (default CUTOFFPET=2))

## Examples

```
Rscript MICC2.R interactions.intra.bedpe interactions.inter.bedpe miccOUT CUTOFFPET
```

## Evaluation signals

- Every candidate interaction has both PET ends overlapping an anchor peak within the -S 100 bp slop / -E 500 bp extension windows (pairToBed -type both criterion)
- Output splits cleanly into interactions.intra.bedpe and interactions.inter.bedpe with a PET count column accepted by MICC2.R
- Feeding the output to MICC2.R with CUTOFFPET=2 and FDR <= 0.05 reproduces loop counts comparable to the reported ~25,155 intra-chromosomal interactions on the GM12878 CTCF dataset
- Independent ENCODE CTCF/RAD21/SMC3 ChIP-seq overlap shows anchor occupancy near reported levels (CTCF 99.9%, RAD21 98.8%, SMC3 99.7%)
- PET counts per candidate interaction are >= the CUTOFFPET threshold before MICC significance estimation

## Limitations

- Loop calling depends on ChIP-seq-like peak-calling sensitivity: anchors outside called peaks cannot form interactions, since the strategy assumes anchors lie at ChIP-Seq-like peaks
- Default slop (100 bp) and anchor extension (500 bp) trade off sensitivity versus specificity; non-default thresholds change the candidate set and downstream FDR
- No software version stated for ChIA-PET2 or its dependencies (bwa, samtools, bedtools, MACS2), complicating exact reproduction

## Evidence
