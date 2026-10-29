---
name: multi-mapping-read-filtering
description: Use when aligning converted methylation reads (3-letter bisulfite or
  4-letter TAPS/5hmC-CATCH mode) with Msuite and you must remove ambiguously mapped
  reads before methylation calling.
when_to_use_negative:
- Input is raw genomic (non-methylation) WGS data where full-length unique alignment
  is expected — standard aligner multi-mapping handling (e.g. bowtie2's default reporting)
  is more appropriate
- BS-seq data, which must use 3-letter mode with -m BS; this filter should not be
  applied as part of a 4-letter TAPS-style pipeline on bisulfite libraries
- Reads failing the -q 20 minimum cycle quality or --minsize 22 minimum length thresholds
  — they are removed before the multi-mapping rule applies, so filtering on multi-hits
  alone is moot for them
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0102
tools:
- name: Msuite
  role: Performs the 3-/4-letter converted alignment and internally screens for multi-mapping
    reads, keeping only unique best hits before methylation call
  repo: https://github.com/hellosunking/Msuite/
- name: bowtie2
  role: Underlying aligner Msuite uses for the strand-specific index searches whose
    multiple equal-best hits the filtering rule resolves
  repo: https://github.com/BenLangmead/bowtie2
- name: samtools
  role: Converts the filtered SAM alignment output (e.g. Msuite.rmdup.sam) into BAM
    format
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.patter.2020.100127
    title: 'Msuite: A High-Performance and Versatile DNA Methylation Data-Analysis
      Toolkit'
  source_content_hash: sha256:1c5f877f958c9b628a043f3a9660ac2642ac632c08bf82a88c90478f1c6abbba
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# Multi-mapping read filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A filtering rule in Msuite's methylation read-alignment pipeline: sequencing reads that map to multiple locations on the same strand with equal best alignment scores are discarded so that only uniquely best-mapping reads proceed to methylation calling.

## When to use

Use when aligning converted methylation reads (3-letter bisulfite or 4-letter TAPS/5hmC-CATCH mode) with Msuite and you must remove ambiguously mapped reads before methylation calling.

## When NOT to use

- Input is raw genomic (non-methylation) WGS data where full-length unique alignment is expected — standard aligner multi-mapping handling (e.g. bowtie2's default reporting) is more appropriate
- BS-seq data, which must use 3-letter mode with -m BS; this filter should not be applied as part of a 4-letter TAPS-style pipeline on bisulfite libraries
- Reads failing the -q 20 minimum cycle quality or --minsize 22 minimum length thresholds — they are removed before the multi-mapping rule applies, so filtering on multi-hits alone is moot for them

## Inputs

- FASTQ read files (single-end via -U or paired-end via -1/-2)
- Msuite genome indices for Watson and Crick strands built by build.index.sh GENOME.FA (or GENOME.DIR) REFSEQ.txt Genome.ID
- Cytosine-converted reads (CpG-context C-to-T for 4-letter TAPS mode, or all-C-to-T for 3-letter BS mode)

## Outputs

- Alignment SAM/BAM output (e.g. Msuite.rmdup.sam converted to BAM with samtools) containing only uniquely best-mapping reads
- Methylation call files restricted to uniquely mapped reads passing -Q score
- four_letter_verification_report.txt (4-letter mode validation report)

## How to apply

Run Msuite alignment against the Watson- and Crick-strand indices built by build.index.sh, keeping the default minimum quality score -q 20 and minimum read size --minsize 22 unless data quality dictates otherwise. After alignment, apply the ambiguity rule: a read mapping to multiple locations on the same strand with equal best scores is discarded, while reads with a unique best hit are retained for downstream methylation call (methylation calling itself is gated by the -Q minimum alignment score, default 0). The rationale is that cytosine-to-thymine conversion reduces sequence complexity, inflating spurious multi-hits that would corrupt per-cytosine methylation estimates. Follow with the related PCR-duplicate rule (identical start/end/strand keep best-quality read) so the final alignment output, e.g. Msuite.rmdup.sam, contains only unique, de-duplicated alignments. In the 4-letter mode, run 'cd OUTDIR; make' to execute the generated makefile and then verify the rule took effect by confirming no multi-mapped reads remain in the outputs.

## Related tools

- **Msuite** (Performs the 3-/4-letter converted alignment and internally screens for multi-mapping reads, keeping only unique best hits before methylation call) — https://github.com/hellosunking/Msuite/
- **bowtie2** (Underlying aligner Msuite uses for the strand-specific index searches whose multiple equal-best hits the filtering rule resolves) — https://github.com/BenLangmead/bowtie2
- **samtools** (Converts the filtered SAM alignment output (e.g. Msuite.rmdup.sam) into BAM format)

## Examples

```
msuite -4 -m TAPS -x Genome.ID -1 reads_R1.fq -2 reads_R2.fq -q 20 --minsize 22 -o OUTDIR && cd OUTDIR && make
```

## Evaluation signals

- No reads in the output SAM with multiple equal-best alignments on the same strand; all retained alignments are unique best hits
- Msuite.rmdup.sam contains no read pairs with identical start position, end position, and strand (PCR-duplicate check passing)
- All retained reads meet the -q 20 minimum cycle quality and --minsize 22 minimum length defaults
- Only reads with alignment score at or above the -Q threshold (default 0) contribute to methylation calls
- In 4-letter mode, four_letter_verification_report.txt exists and records that CpH cytosines are preserved in the converted reference while filtering held

## Limitations

- The rule discards reads mapping to multiple locations on the same strand with equal best scores, which loses coverage in repetitive or low-complexity regions — aggravated because C-to-T conversion reduces nucleotide complexity
- No strong evidence that 4-letter mode (with this filtering) is more accurate on inconsistent CpG sites; further validation with methods such as microarrays is still needed
- Real TAPS datasets can have imperfect chemical treatment, which may leave residual unmethylated CpG-context conversions that the alignment and filtering are not designed to resolve

## Evidence
