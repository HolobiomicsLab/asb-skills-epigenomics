---
name: long-read-fragment-extraction-pacbio
description: Use when preparing PacBio SMRT (or other long-read) BAM alignments plus
  a position-sorted, bgzipped VCF as input for diploid haplotype phasing with SpecHap;
  the phaser consumes fragments, not alignments.
when_to_use_negative:
- Input is already a fragment file or a phased VCF — extraction would duplicate preprocessing
- The downstream phaser is not fragment-based (e.g. trio/LD-based statistical phasing),
  which does not consume extractHAIRS output
- Variants come from a polyploid organism — SpecHap's graph bisection model does not
  fit increased ploidy
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: ExtractHair (HapCUT2-derived)
  role: Fragment extraction from PacBio BAM/VCF in long-read mode ('--pacbio 1');
    bundled as a SpecHap submodule
  repo: https://github.com/vibansal/HapCUT2
- name: SpecHap
  role: Downstream consumer of the sorted fragment file; run with --pacbio for spectral
    phasing
  repo: https://github.com/deepomicslab/SpecHap
- name: htslib
  role: Required dependency (pinned 26229a3 in submodule) for reading BAM/VCF during
    extraction
  repo: https://github.com/samtools/htslib
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab709
    title: 'SpecHap: a diploid phasing algorithm based on spectral graph theory'
  source_content_hash: sha256:84fb9221b6fc0304cfb2dca12b043b00190b041ee916237eef221bc43b700ec7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/deepomicslab/SpecHap
  repo_url: https://github.com/vibansal/HapCUT2
  tools:
  - DCHap
  - ExtractHair (HapCUT2-derived)
  - HapCUT2
  - PacBio SMRT
  - ReFHap
  - SpecHap
  - htslib
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# long-read-fragment-extraction-pacbio

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract read-fragment linkage data from PacBio SMRT long-read alignments using the HapCUT2-derived ExtractHair utility with '--pacbio 1' settings, producing the coordinate-sorted fragment file that SpecHap requires as phasing input.

## When to use

Use when preparing PacBio SMRT (or other long-read) BAM alignments plus a position-sorted, bgzipped VCF as input for diploid haplotype phasing with SpecHap; the phaser consumes fragments, not alignments.

## When NOT to use

- Input is already a fragment file or a phased VCF — extraction would duplicate preprocessing
- The downstream phaser is not fragment-based (e.g. trio/LD-based statistical phasing), which does not consume extractHAIRS output
- Variants come from a polyploid organism — SpecHap's graph bisection model does not fit increased ploidy

## Inputs

- Position-sorted, bgzipped VCF of heterozygous variant calls
- Coordinate-sorted BAM of PacBio SMRT long-read alignments
- Reference genome FASTA

## Outputs

- PacBio-format fragment file of read-allele linkages, coordinate-sorted via 'sort -n -k3'

## How to apply

Build the modified ExtractHair shipped as a SpecHap submodule (git submodule init && git submodule update; checkout htslib 26229a3 and samtools 255f97d; make in hair-src). Run extractHAIRS in long-read mode with '--pacbio 1' together with the BAM, VCF, and reference file arguments to convert aligned reads into a fragment file listing heterozygous alleles covered by each read. Sort the fragment file numerically on the third field ('sort -n -k3') so fragment order matches the coordinate-sorted, bgzipped VCF expected by SpecHap's PacBio mode. The rationale is that SpecHap's spectral linkage graph is built from these fragments, so extraction settings must match the sequencing platform's error/overlap profile. Confirm correctness by phasing downstream: a complete phased VCF within the reported resource envelope (e.g. axolotl ~32 Gbp genome, ~6 CPU hours, 945 MB peak memory).

## Related tools

- **ExtractHair (HapCUT2-derived)** (Fragment extraction from PacBio BAM/VCF in long-read mode ('--pacbio 1'); bundled as a SpecHap submodule) — https://github.com/vibansal/HapCUT2
- **SpecHap** (Downstream consumer of the sorted fragment file; run with --pacbio for spectral phasing) — https://github.com/deepomicslab/SpecHap
- **htslib** (Required dependency (pinned 26229a3 in submodule) for reading BAM/VCF during extraction) — https://github.com/samtools/htslib

## Examples

```
extractHAIRS --pacbio 1 --bam reads.bam --VCF variants.vcf.gz --ref genome.fa | sort -n -k3 > frags.sorted.txt
```

## Evaluation signals

- Fragment file is numerically sorted on column 3 and its coordinate order matches the position-sorted, bgzipped VCF
- SpecHap runs to completion on the fragment file without format errors, writing a phased VCF to the --out path
- Resource envelope consistent with published benchmark (e.g. axolotl: ~6 CPU hours, 945 MB peak memory, measured via /usr/bin/time -v)
- Fragments cover heterozygous SNVs with allele observations consistent with ~32x PacBio coverage of the source BAM

## Limitations

- No software version stated for SpecHap or the bundled ExtractHair; submodule pins (htslib 26229a3, samtools 255f97d) must be respected for reproducibility
- Fragment extraction supports only SNPs, short insertions and deletions; haplotype-resolved structural variation is not handled
- Phasing result depends on SpecHap's interval size parameter and there is no guarantee of optimal solutions

## Evidence

- [other] PacBio long-read fragment extraction settings: [evidence span withheld — anchor status: not_found]
- [other] Fragment sorting requirement: [evidence span withheld — anchor status: not_found]
