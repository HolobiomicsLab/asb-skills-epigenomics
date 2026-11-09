---
name: four-letter-versus-three-letter-mapping-benchmarking
description: Use when a TAPS or 5hmC-CATCH dataset is available and a decision is
  needed between Msuite's 4-letter mode (faster, exploits higher nucleotide complexity)
  and the conventional 3-letter mode used by Bismark/BWA-meth.
when_to_use_negative:
- 'Data generated with a BS-seq protocol: the README mandates 3-letter mode with ''-m
  BS'', since 4-letter mode only supports TAPS/5hmC-CATCH data where non-CpG methylation
  is very low'
- Task is to validate which mode is more accurate at inconsistent CpG sites — the
  authors state they lack strong evidence for 4-letter superiority on such sites and
  recommend orthogonal methods (e.g., microarrays)
- Input reads are single-end or shorter than the 22-bp minimum read size enforced
  by Msuite's '--minsize' default
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0622
tools:
- name: Msuite
  role: All-in-one methylation toolkit run in both '-3 -m TAPS' and '-4 -m TAPS' modes
    to compare speed, mapping, and CpG methylation calls
  repo: https://github.com/hellosunking/Msuite/
- name: bowtie2
  role: Underlying aligner invoked by Msuite in both 3-letter and 4-letter modes
  repo: https://github.com/BenLangmead/bowtie2
- name: samtools
  role: Converts Msuite SAM output into the Msuite.final.bam used for mapped-read
    counting
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2020.100127
    title: 'Msuite: A High-Performance and Versatile DNA Methylation Data-Analysis
      Toolkit'
  source_content_hash: sha256:3b73d291265437e929668947ef52e1aea961339f6330dbab634fab7c44c9326b
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
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# four-letter-versus-three-letter-mapping-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark Msuite's 4-letter and 3-letter alignment modes on the same TAPS WGBS library from murine ESCs (GEO GSE129673, GRCm38/mm10) to quantify runtime, mapped-read, and CpG methylation-call trade-offs of the bisulfite-free 4-letter mode.

## When to use

Use when a TAPS or 5hmC-CATCH dataset is available and a decision is needed between Msuite's 4-letter mode (faster, exploits higher nucleotide complexity) and the conventional 3-letter mode used by Bismark/BWA-meth.

## When NOT to use

- Data generated with a BS-seq protocol: the README mandates 3-letter mode with '-m BS', since 4-letter mode only supports TAPS/5hmC-CATCH data where non-CpG methylation is very low
- Task is to validate which mode is more accurate at inconsistent CpG sites — the authors state they lack strong evidence for 4-letter superiority on such sites and recommend orthogonal methods (e.g., microarrays)
- Input reads are single-end or shorter than the 22-bp minimum read size enforced by Msuite's '--minsize' default

## Inputs

- Paired-end FASTQ files of murine ESC TAPS data (GEO GSE129673)
- Paired-end FASTQ files of murine ESC WGBS data (GEO GSE112520)
- GRCm38/mm10 reference genome FASTA
- mm10 RefSeq annotation file shipped with the Msuite package

## Outputs

- Msuite.final.bam aligned read files for each mode
- Msuite.CpG.meth.call per-CpG methylation-call tables for 4- and 3-letter runs
- Msuite run reports with total runtime and final mapped-read counts
- msuite_mode_comparison_stats.csv with runtime, mapped-read deficit, Pearson's R, and discordant-CpG fraction

## How to apply

Build GRCm38/mm10 genome indices with Msuite's build.index.sh utility (which automatically incorporates the Lambda genome) using the mm10 RefSeq annotation shipped in the package. Run Msuite twice on the same paired-end TAPS FASTQs: once with '-3 -m TAPS' and once with '-4 -m TAPS', both with '-p 32' threads and letting the sequencing cycle auto-detect; run the WGBS control (GSE112520) with '-3 -m BS'. Extract total runtime and mapped-read counts from the Msuite reports and Msuite.final.bam outputs, then parse Msuite.CpG.meth.call from both TAPS runs to compute the Pearson correlation of per-CpG methylation densities and the fraction of CpG sites with >10% methylation difference. On the real murine ESC TAPS data, 4-letter mode ran ~50% faster (~290 vs ~440 min on 32 threads) with a 2.17% mapped-read deficit, Pearson's R > 0.99, and only 3.78% of CpGs differing by >10%.

## Related tools

- **Msuite** (All-in-one methylation toolkit run in both '-3 -m TAPS' and '-4 -m TAPS' modes to compare speed, mapping, and CpG methylation calls) — https://github.com/hellosunking/Msuite/
- **bowtie2** (Underlying aligner invoked by Msuite in both 3-letter and 4-letter modes) — https://github.com/BenLangmead/bowtie2
- **samtools** (Converts Msuite SAM output into the Msuite.final.bam used for mapped-read counting)

## Examples

```
msuite -4 -x mm10 -1 sample_R1.fq -2 sample_R2.fq -m TAPS -p 32 -o taps_4letter_out
```

## Evaluation signals

- Pearson's R between 4-letter and 3-letter TAPS per-CpG methylation densities exceeds 0.99
- Mapped-read deficit of 4-letter mode relative to 3-letter mode is approximately 2.17%
- Approximately 3.78% of CpG sites show methylation differences higher than 10% between modes
- 4-letter run completes roughly 50% faster than 3-letter run (~290 vs ~440 min on 32 threads) on the same TAPS input
- WGBS control run completes only with '-3 -m BS' and produces expected TSS-region methylation QC profiles

## Limitations

- No strong evidence that the 4-letter mode is more accurate at CpG sites where the two modes disagree; validation with orthogonal methods (e.g., microarrays) is needed
- Real TAPS data had imperfect chemical treatment, so mode-comparison statistics reflect that imperfection
- 4-letter mode only supports TAPS/5hmC-CATCH data with very low non-CpG methylation (e.g., most human somatic tissues), not BS-seq
- Msuite handles at most 4,294,967,295 reads per sample and the -3/-4 and phred33/phred64 options are mutually exclusive

## Evidence
