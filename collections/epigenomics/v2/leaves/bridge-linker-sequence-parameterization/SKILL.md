---
name: bridge-linker-sequence-parameterization
description: Use when trimming linkers from bridge-linker ChIA-PET paired-end FASTQ
  with ChIA-PET2 and you must set the linkerA/linkerB 15-mers and mismatch tolerance
  (-e 0 vs -e 1) to avoid losing valid PETs to sequencing errors.
when_to_use_negative:
- Processing half-linker ChIA-PET data — ChIA-PET2 supports both protocols but the
  ACGCGATATCTTATCTGACT bridge-linker A/B parameterization applies only to the bridge-linker
  protocol
- The library used a different linker sequence — the 15-nt linkerA/linkerB must be
  re-derived from the actual linker rather than reused
- Downstream stages (read alignment, PET filtering, PCR duplicate removal, peak or
  loop calling) — this skill covers only the linker-trimming step
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0091
tools:
- name: ChIA-PET2
  role: Provides the trimLinker subcommand used to trim bridge-linker sequences from
    pair-end FASTQ with configurable linkerA/linkerB, mismatch allowance (-e), minimum
    length (-l) and mode (-m); produces the trim.stat statistics used for the 0- vs
    1-mismatch comparison
  repo: https://github.com/GuipengLi/ChIA-PET2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkw809
    title: 'ChIA-PET2: a versatile and flexible pipeline for ChIA-PET data analysis'
  source_content_hash: sha256:41dc1d1295418c00c08c2107efb230158d55dc682f523277e9c355b3a0f23b55
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
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# bridge-linker-sequence-parameterization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parameterizes ChIA-PET2's trimLinker for bridge-linker ChIA-PET libraries by deriving 15-nt linkerA/linkerB half-sites from the 5' ends of the bridge linker ACGCGATATCTTATCTGACT and tuning the mismatch allowance (-e). On the GM12878 CTCF dataset, allowing 1 mismatch rescues >16 million PETs and yields a 12.85% improvement in valid PET ratio versus strict 0-mismatch trimming.

## When to use

Use when trimming linkers from bridge-linker ChIA-PET paired-end FASTQ with ChIA-PET2 and you must set the linkerA/linkerB 15-mers and mismatch tolerance (-e 0 vs -e 1) to avoid losing valid PETs to sequencing errors.

## When NOT to use

- Processing half-linker ChIA-PET data — ChIA-PET2 supports both protocols but the ACGCGATATCTTATCTGACT bridge-linker A/B parameterization applies only to the bridge-linker protocol
- The library used a different linker sequence — the 15-nt linkerA/linkerB must be re-derived from the actual linker rather than reused
- Downstream stages (read alignment, PET filtering, PCR duplicate removal, peak or loop calling) — this skill covers only the linker-trimming step

## Inputs

- Raw paired-end FASTQ or fastq.gz files from a bridge-linker ChIA-PET library (e.g. GM12878 CTCF, NCBI GEO GSE72816 / GSM970213)
- Bridge-linker sequence ACGCGATATCTTATCTGACT used in the ChIA-PET protocol

## Outputs

- Trimmed paired-end FASTQ files containing linker-free genomic reads (each read >= 15 nt)
- output.trim.stat summary files reporting rescued PET counts and valid PET ratios per -e setting

## How to apply

Run the ChIA-PET2 trimLinker subcommand in bridge-linker mode (-m 1) on the raw pair-end fastq(.gz) files. Split the 20-nt bridge linker ACGCGATATCTTATCTGACT into the first 15 bases of each 5' end: linkerA=ACGCGATATCTTATC and linkerB=AGTCAGATAAGATAT, set the minimum trimmed read length to 15 (-l 15) and keep-empty to 0 (-k 0). Execute the identical run twice, first allowing 0 mismatches (-e 0) and then 1 mismatch (-e 1); the rationale is that strict 0-mismatch linker searching discards PETs whose linker carries a single sequencing error, whereas -e 1 rescues them. Parse both output.trim.stat summaries, extract rescued PET counts and valid PET ratio, and compare -e 1 against -e 0. Validate against the benchmark that -e 1 rescues >16 million PETs and improves the valid PET ratio by ~12.85% on the GM12878 CTCF ChIA-PET data (GSE72816 / GSM970213).

## Related tools

- **ChIA-PET2** (Provides the trimLinker subcommand used to trim bridge-linker sequences from pair-end FASTQ with configurable linkerA/linkerB, mismatch allowance (-e), minimum length (-l) and mode (-m); produces the trim.stat statistics used for the 0- vs 1-mismatch comparison) — https://github.com/GuipengLi/ChIA-PET2

## Examples

```
ChIA-PET2 trimLinker -m 1 -a ACGCGATATCTTATC -b AGTCAGATAAGATAT -e 1 -k 0 -l 15 R1.fq.gz R2.fq.gz
```

## Evaluation signals

- The -e 1 output.trim.stat shows >16 million additional rescued PETs relative to the -e 0 run
- Valid PET ratio improves by approximately 12.85% at 1 allowed mismatch versus 0 mismatches on the GM12878 CTCF dataset
- linkerA (ACGCGATATCTTATC) and linkerB (AGTCAGATAAGATAT) are exactly the first 15 bases of each 5' end of the bridge linker ACGCGATATCTTATCTGACT
- All trimmed reads output by trimLinker are at least 15 nt, consistent with -l 15
- Both runs use identical settings except -e (0 vs 1), so the trim.stat comparison isolates the mismatch-tolerance effect

## Limitations

- The reported >16 million rescued PETs and 12.85% valid-PET improvement are specific to the GM12878 CTCF ChIA-PET dataset (GSE72816 / GSM970213); other datasets will yield different counts
- The benchmark only contrasts 0 and 1 mismatch; larger mismatch allowances were not evaluated in this workflow, and permissive settings may risk false linker detection
- If -k (keep-empty) or -l (minimum trimmed length, here 15) are changed, the valid PET ratio will no longer be comparable to the published benchmark

## Evidence

- [methods] trimLinker trims linker sequences in the pair-end fastq(.gz) files: "**trimLinker:** trim linker sequences in the pair-end fastq(.gz) files."
- [readme] ChIA-PET2 integrates all ChIA-PET analysis steps including linkers trimming: "ChIA-PET2 integrates all steps required for ChIA-PET data analysis, including linkers trimming, reads mapping, duplicates removing, peaks calling and chromatin loops calling."
- [intro] Linker trimming is the first common step of ChIA-PET raw data processing workflows: "A workflow for processing ChIA-PET raw data often requires these common steps, linker trimming, read alignment, paired-end tag (PET) filtering"
