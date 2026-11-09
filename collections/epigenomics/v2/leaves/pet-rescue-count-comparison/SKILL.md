---
name: pet-rescue-count-comparison
description: Use when processing bridge-linker ChIA-PET raw reads and you need to
  measure the sensitivity gain of linker-searching at 1 allowed mismatch (-e 1) versus
  the strict 0-mismatch setting (-e 0), expressed as rescued PET counts and valid
  PET ratio from the trim.stat summaries.
when_to_use_negative:
- Input is half-linker ChIA-PET data rather than the bridge-linker protocol — use
  a half-linker trimming strategy instead of the -m 1 bridge-linker mode
- The goal is significant chromatin loop calling or allele-specific analysis rather
  than quantifying linker-trimming sensitivity
- Linker sequences differ from the bridge linker ACGCGATATCTTATCTGACT used in this
  protocol; the rescue benchmark does not transfer
edam_operation: null
tools:
- name: ChIA-PET2
  role: trimLinker step trims linker sequences from paired-end FASTQ files and reports
    the PET rescue statistics in the trim.stat summary
  repo: https://github.com/GuipengLi/ChIA-PET2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkw809
    title: 'ChIA-PET2: a versatile and flexible pipeline for ChIA-PET data analysis'
  source_content_hash: sha256:8668423dc586815d3b1a0e512b7d17a7815f20f07c56deac7a5f7c6e037eec9b
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# pet-rescue-count-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how many paired-end tags (PETs) are rescued and how much the valid PET ratio improves when ChIA-PET2's trimLinker step allows one mismatch in the bridge-linker sequence versus zero mismatches, using the GM12878 CTCF ChIA-PET dataset as a benchmark (>16 million rescued PETs, 12.85% improvement).

## When to use

Use when processing bridge-linker ChIA-PET raw reads and you need to measure the sensitivity gain of linker-searching at 1 allowed mismatch (-e 1) versus the strict 0-mismatch setting (-e 0), expressed as rescued PET counts and valid PET ratio from the trim.stat summaries.

## When NOT to use

- Input is half-linker ChIA-PET data rather than the bridge-linker protocol — use a half-linker trimming strategy instead of the -m 1 bridge-linker mode
- The goal is significant chromatin loop calling or allele-specific analysis rather than quantifying linker-trimming sensitivity
- Linker sequences differ from the bridge linker ACGCGATATCTTATCTGACT used in this protocol; the rescue benchmark does not transfer

## Inputs

- Paired-end ChIA-PET FASTQ(.gz) reads (GM12878 CTCF, GSE72816 / GSM970213)
- Bridge-linker sequence ACGCGATATCTTATCTGACT (first 15 5' bases split into linkerA and linkerB)

## Outputs

- output.trim.stat summary files for the -e 0 and -e 1 runs
- Rescued PET count and valid PET ratio delta between the two mismatch settings

## How to apply

Retrieve the GM12878 CTCF ChIA-PET paired-end FASTQ files from GEO accession GSE72816 / GSM970213. Run ChIA-PET2's trimLinker step in bridge-linker mode (-m 1) using the first 15 bases of each 5' end of the bridge linker ACGCGATATCTTATCTGACT — linkerA=ACGCGATATCTTATC and linkerB=AGTCAGATAAGATAT — with -k 0 (default keep-empty) and minimum trimmed read length 15 (-l 15), first with -e 0 then with -e 1. Parse the output.trim.stat summary files from both runs to extract the rescued PET counts and the valid PET ratio. Compare the two settings: the -e 1 run should rescue >16 million additional PETs and yield roughly a 12.85% improvement in the valid PET ratio, confirming that mismatch-tolerant linker searching substantially increases usable PETs for downstream alignment and loop calling.

## Related tools

- **ChIA-PET2** (trimLinker step trims linker sequences from paired-end FASTQ files and reports the PET rescue statistics in the trim.stat summary) — https://github.com/GuipengLi/ChIA-PET2

## Examples

```
ChIA-PET2 trimLinker -m 1 -a ACGCGATATCTTATC -b AGTCAGATAAGATAT -e 1 -k 0 -l 15 -f1 read1.fq.gz -f2 read2.fq.gz
```

## Evaluation signals

- The -e 1 run rescues approximately >16 million PETs relative to the -e 0 run
- The valid PET ratio improves by roughly 12.85% at 1 allowed mismatch compared to 0 mismatches
- output.trim.stat files are produced for both runs and parse cleanly into comparable summary counts
- Rescued PET count at -e 1 must be strictly greater than at -e 0; equality would indicate the mismatch option was not applied
- All other trimLinker parameters (-m 1, -k 0, -l 15, linkerA/linkerB) are identical between the two runs so the comparison is controlled

## Limitations

- The benchmark (>16 million rescued PETs, 12.85% valid PET ratio improvement) is specific to the GM12878 CTCF ChIA-PET dataset and will not reproduce exactly on other datasets or linkers
- No software version of ChIA-PET2 is stated in the article, and no changelog was found in the repository, so exact reproducibility may depend on which release is used
- Allowing more linker mismatches may increase false linker matches, so the rescue gain should be weighed against trimming specificity

## Evidence

- [other] benchmark rescue finding on GM12878 CTCF: [evidence span withheld — anchor status: not_found]
