---
name: pcr-duplicate-removal
description: Use when processing methylation-seq alignments (e.g., Msuite SAM output)
  where PCR over-amplification may inflate read counts; reads sharing identical start/end
  coordinates and strand must be collapsed before methylation calling.
when_to_use_negative:
- Data is single-molecule or UMIdeduplicated already, so identical start/end/strand
  reads are not PCR artifacts
- Running BS-seq (bisulfite) data — Msuite requires the 3-letter mode with '-m BS'
  rather than the 4-letter TAPS/CATCH workflow this dedup step was validated in
- Reads differ in start or end position or strand — they are not PCR duplicates under
  Msuite's rule and must not be collapsed
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_0654
tools:
- name: Msuite
  role: Performs the read alignment and applies the PCR duplicate rule (identical
    start/end/strand; keeps the best-quality read) and multi-mapping rule within its
    pipeline
  repo: https://github.com/hellosunking/Msuite/
- name: samtools
  role: Converts the deduplicated SAM output into BAM format
- name: bowtie2
  role: Underlying aligner used by Msuite to produce the alignments that are subsequently
    deduplicated
  repo: https://github.com/BenLangmead/bowtie2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.patter.2020.100127
    title: 'Msuite: A High-Performance and Versatile DNA Methylation Data-Analysis
      Toolkit'
  source_content_hash: sha256:227992322c3f6978d10dffc8ae6958f4b62b0e68e8103de40f0ed316056e429d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/hellosunking/Msuite
  repo_url: https://github.com/hellosunking/Msuite
  tools:
  - Msuite
  - bowtie2
  - samtools
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# pcr-duplicate-removal

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Post-alignment duplicate removal step from the Msuite pipeline: aligned reads with identical start position, end position, and strand are flagged as PCR duplicates and only the copy with the best sequencing read quality is retained, ensuring downstream methylation calls are not biased by PCR-amplified fragments.

## When to use

Use when processing methylation-seq alignments (e.g., Msuite SAM output) where PCR over-amplification may inflate read counts; reads sharing identical start/end coordinates and strand must be collapsed before methylation calling.

## When NOT to use

- Data is single-molecule or UMIdeduplicated already, so identical start/end/strand reads are not PCR artifacts
- Running BS-seq (bisulfite) data — Msuite requires the 3-letter mode with '-m BS' rather than the 4-letter TAPS/CATCH workflow this dedup step was validated in
- Reads differ in start or end position or strand — they are not PCR duplicates under Msuite's rule and must not be collapsed

## Inputs

- Aligned SAM records from Msuite (e.g., Msuite.rmdup.sam) with start/end coordinates and strand flags
- Sequencing read quality scores (FASTQ Phred33/Phred64 cycles, minimum quality -q 20)
- 4-letter mode alignments generated with Msuite -4 -m TAPS against a CpG-context-converted genome index

## Outputs

- Deduplicated SAM/BAM alignment file (e.g., Msuite.rmdup.sam converted to BAM with samtools) containing one read per unique start/end/strand combination
- Multi-mapping-filtered alignment retaining only unique best hits

## How to apply

After Msuite produces alignments (e.g., via 'cd OUTDIR; make' to run the generated makefile), apply the duplicate rule to the aligned reads: group reads that have identical start positions, identical end positions, and identical strand information, and from each such group keep only the read with the best sequencing quality. This runs alongside the multi-mapping rule, in which reads mapping to multiple locations on the same strand with equal best scores are discarded, so that only unique best hits survive into the methylation call stage. Cycle-level quality is controlled upstream with '-q 20' (default, Phred) and '--minsize 22', so only sufficiently long, high-quality reads enter deduplication. Validate the result by checking that the deduplicated output (e.g., Msuite.rmdup.sam) contains no reads with identical start/end/strand pairs.

## Related tools

- **Msuite** (Performs the read alignment and applies the PCR duplicate rule (identical start/end/strand; keeps the best-quality read) and multi-mapping rule within its pipeline) — https://github.com/hellosunking/Msuite/
- **samtools** (Converts the deduplicated SAM output into BAM format)
- **bowtie2** (Underlying aligner used by Msuite to produce the alignments that are subsequently deduplicated) — https://github.com/BenLangmead/bowtie2

## Examples

```
msuite -4 -m TAPS -x Genome.ID -1 reads_1.fq -2 reads_2.fq -q 20 --minsize 22 -o OUTDIR; cd OUTDIR; make
```

## Evaluation signals

- Msuite.rmdup.sam contains no reads with identical start/end/strand triples
- four_letter_verification_report.txt exists and records that CpH cytosines are preserved in the converted reference
- No multi-mappers remain: reads that mapped to multiple locations on the same strand with equal best scores are absent from the output
- Only reads passing '-q 20' cycle quality and '--minsize 22' were kept for alignment before deduplication
- Total read counts after deduplication are consistent with PCR duplicate rates expected for the library

## Limitations

- The rule treats only reads with identical start AND end positions AND strand as duplicates — near-duplicates with differing coordinates are not removed
- On real TAPS datasets with imperfect chemical treatment, results remain usable, but the article notes further validation of 4-letter mode accuracy on inconsistent CpG sites is still needed
- Msuite caps input at 4,294,967,295 reads, and the deduplication runs inside the Msuite pipeline rather than as a standalone tool

## Evidence
