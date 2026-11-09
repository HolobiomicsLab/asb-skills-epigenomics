---
name: pcr-duplicate-removal-genomic-pets
description: Use when processing ChIA-PET paired-end FASTQ data and identical PETs
  arising from PCR amplification must be collapsed prior to MACS2 peak calling and
  MICC2 loop significance estimation.
when_to_use_negative:
- Input is raw FASTQ reads that still contain linker sequences — trimLinker and bwa
  alignment must run first
- Data are not PET-based ChIA-PET libraries (e.g. standard ChIP-Seq) where per-read
  duplicate handling differs
- The goal is to assess library complexity or PCR amplification rate — QCplots.R should
  be used instead of discarding duplicates silently
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: ChIA-PET2
  role: provides the removeDup command that collapses duplicate PETs in the bedpe
    file within the ChIA-PET2 pipeline
  repo: https://github.com/GuipengLi/ChIA-PET2
- name: bwa
  role: aligns trimmed paired-end reads to the reference genome; its SAM output is
    the direct input for building the bedpe that removeDup deduplicates
  repo: https://github.com/lh3/bwa
- name: samtools
  role: handles SAM/BAM manipulation around the alignment step feeding buildBedpe
  repo: https://github.com/samtools/samtools
- name: bedtools
  role: bed/bedpe file manipulation used downstream of duplicate removal (pairToBed
    -type both in bedpe2Interaction)
  repo: https://github.com/arq5x/bedtools2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkw809
    title: 'ChIA-PET2: a versatile and flexible pipeline for ChIA-PET data analysis'
  source_content_hash: sha256:221ea4a69b19aa8e9d6ea37ae74ae355d25f6c47708aded07ff6120a83715521
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
  - R
  - bedtools
  - bwa
  - samtools
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# pcr-duplicate-removal-genomic-pets

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Removes PCR duplicate paired-end tags (PETs) from ChIA-PET alignment output before peak calling and loop calling, using ChIA-PET2's removeDup on bedpe files built from bwa-aligned reads.

## When to use

Use when processing ChIA-PET paired-end FASTQ data and identical PETs arising from PCR amplification must be collapsed prior to MACS2 peak calling and MICC2 loop significance estimation.

## When NOT to use

- Input is raw FASTQ reads that still contain linker sequences — trimLinker and bwa alignment must run first
- Data are not PET-based ChIA-PET libraries (e.g. standard ChIP-Seq) where per-read duplicate handling differs
- The goal is to assess library complexity or PCR amplification rate — QCplots.R should be used instead of discarding duplicates silently

## Inputs

- bedpe file of paired-end tags built from two bwa-aligned SAM files (buildBedpe output, MAPQ >= 30)
- number of threads (N) for parallel duplicate removal

## Outputs

- deduplicated bedpe file of unique PETs for tagAlign construction, peak calling, and loop detection

## How to apply

After trimming linkers with trimLinker (mismatch = 0, minimum trimmed read length = 15) and aligning with bwa, build a bedpe file from the two SAM files using buildBedpe with a MAPQ cutoff of 30. Then run removeDup (with the desired number of threads, N) on the bedpe to collapse duplicate PETs that share identical coordinates, since PCR amplification artificially inflates interaction frequencies at the same genomic positions. The deduplicated bedpe is then converted to a tagAlign file (buildTagAlign) for MACS2 anchor peak calling at q-value 0.05 and fed to bedpe2Interaction and MICC2.R (PET count cutoff 2) for loop calling. Duplicate removal must precede peak and loop calling so that PET counts used for significance reflect independent ligation events rather than PCR artifacts.

## Related tools

- **ChIA-PET2** (provides the removeDup command that collapses duplicate PETs in the bedpe file within the ChIA-PET2 pipeline) — https://github.com/GuipengLi/ChIA-PET2
- **bwa** (aligns trimmed paired-end reads to the reference genome; its SAM output is the direct input for building the bedpe that removeDup deduplicates) — https://github.com/lh3/bwa
- **samtools** (handles SAM/BAM manipulation around the alignment step feeding buildBedpe) — https://github.com/samtools/samtools
- **bedtools** (bed/bedpe file manipulation used downstream of duplicate removal (pairToBed -type both in bedpe2Interaction)) — https://github.com/arq5x/bedtools2

## Examples

```
ChIA-PET2 removeDup -i pet.clusters.bedpe -o pet.nodup.bedpe -p 4
```

## Evaluation signals

- Count of unique PETs in the deduplicated bedpe is lower than the input and no PET appears with identical end coordinates more than once
- Downstream buildTagAlign and MACS2 (q-value 0.05) successfully produce anchor peaks from the deduplicated bedpe
- bedpe2Interaction + MICC2.R (PET cutoff 2) output retains the expected 13-column format with -log10(1-PostProbability) and FDR columns
- Validation loops are recovered, e.g. the ~1.6 kb MYC-locus loop supported by 30 PETs with anchors overlapping CTCF/RAD21/SMC3 occupancy
- QC plots (QCplots.R) show duplication rates consistent with the library and no unexpected loss of PETs

## Limitations

- No versions stated for required dependencies (bwa, samtools, bedtools, MACS2, g++) in the article, so exact duplicate-removal behavior may vary across versions
- Aggressive duplicate collapsing can reduce PET counts below the MICC2.R default cutoff of 2 for genuine short-range loops
- Loop calling depends on ChIP-Seq-like peak calling sensitivity, so errors propagated from upstream filtering affect downstream loops

## Evidence
