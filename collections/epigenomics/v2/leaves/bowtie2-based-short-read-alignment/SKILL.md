---
name: bowtie2-based-short-read-alignment
description: Use when short paired-end or single-end methylation sequencing reads
  (36-100 bp FASTQ, e.g., SHERMAN-simulated TAPS data) must be aligned to a reference
  genome such as hg19 as the core step of a bisulfite or bisulfite-free methylation
  analysis.
when_to_use_negative:
- 'Input reads are from a BS-seq (bisulfite) library: the README mandates 3-letter
  mode with `-m BS`, so running the 4-letter alignment would be incorrect.'
- 'The library has substantial non-CpG methylation: 4-letter mode ONLY supports processing
  of TAPS/5hmC-CATCH data where non-CpG methylation is very low (e.g., most somatic
  tissues in human).'
- The goal is to validate 4-letter accuracy at inconsistent CpG sites — the article
  explicitly lacks strong evidence that the 4-letter mode is more accurate there,
  so this skill cannot serve as that validation.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3674
tools:
- name: bowtie2
  role: Underline aligner invoked by Msuite to map converted methylation reads against
    the Watson and Crick genome indices
  repo: https://github.com/BenLangmead/bowtie2
- name: Msuite
  role: Wraps bowtie2 in 3-/4-letter C-conversion modes, handles index building (build.index.sh),
    multimapping and PCR-duplicate filtering, and downstream methylation calling
  repo: https://github.com/hellosunking/Msuite/
- name: samtools
  role: Converts the SAM alignment output into BAM format
  repo: http://samtools.sourceforge.net/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2020.100127
    title: 'Msuite: A High-Performance and Versatile DNA Methylation Data-Analysis
      Toolkit'
  source_content_hash: sha256:166a5ffc43f4d95326b50622cbaed66520619372f5735bed01bdc5e60060ec11
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/hellosunking/Msuite
  repo_url: https://github.com/BenLangmead/bowtie2
  tools:
  - Msuite
  - R
  - bowtie2
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3674
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# bowtie2-based-short-read-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aligns methylation sequencing reads (BS-seq or TAPS) to a reference genome using bowtie2 as the underline aligner inside Msuite's 3-letter (C-to-T converted) or 4-letter alignment mode, producing the SAM/BAM alignments that feed methylation calling.

## When to use

Use when short paired-end or single-end methylation sequencing reads (36-100 bp FASTQ, e.g., SHERMAN-simulated TAPS data) must be aligned to a reference genome such as hg19 as the core step of a bisulfite or bisulfite-free methylation analysis.

## When NOT to use

- Input reads are from a BS-seq (bisulfite) library: the README mandates 3-letter mode with `-m BS`, so running the 4-letter alignment would be incorrect.
- The library has substantial non-CpG methylation: 4-letter mode ONLY supports processing of TAPS/5hmC-CATCH data where non-CpG methylation is very low (e.g., most somatic tissues in human).
- The goal is to validate 4-letter accuracy at inconsistent CpG sites — the article explicitly lacks strong evidence that the 4-letter mode is more accurate there, so this skill cannot serve as that validation.

## Inputs

- Paired-end or single-end FASTQ reads (e.g., SHERMAN-simulated TAPS reads, 36-100 bp; up to 4,294,967,295 reads)
- Reference genome sequence as one multi-fasta file or a directory of per-chromosome sequences (e.g., hg19)
- RefSeq annotation for the genome (mm9/mm10/hg19/hg38 files included with Msuite) for TSS methylation QC

## Outputs

- Aligned SAM files converted to BAM via samtools
- Methylation call tables computed from the alignments (Msuite.CpG.meth.call, Msuite.CpH.meth.call)
- Mapping statistics and TSS methylation QC/visualization outputs (e.g., R-generated plots such as methylation_density_comparison.png)

## How to apply

First build genome indices with `build.index/build.index.sh GENOME.FA REFSEQ.txt Genome.ID` — this utility automatically incorporates the Lambda genome (for conversion-rate estimation) and writes indices to the `index` directory under the Msuite root; RefSeq annotation is included for mm9/mm10/hg19/hg38 and is used for TSS methylation QC. Msuite converts all genomic cytosines to thymines (3-letter mode, default) and builds two bowtie2 indices for the Watson and Crick chains; use `-4` instead for TAPS/5hmC-CATCH data. Run `Msuite -1 read1.fq -2 read2.fq -x hg19 -4 -m TAPS -o out.dir` (add `-m BS` for bisulfite data), then run `make` in the output directory; samtools converts the SAM alignments to BAM. Key parameters: `-q` minimum per-cycle quality score (default 20, i.e., 1% error rate), `--minsize` minimum read size kept for alignment (default 22 bp), and `-Q` minimum alignment score for methylation calling (default 0). During filtering, reads mapped to multiple locations on the same strand with equal best scores are discarded (only unique best hits kept), and reads with identical start/end/strand are treated as PCR duplicates with only the best-quality copy retained. BS-seq data MUST use 3-letter mode with `-m BS`; 4-letter mode is reserved for libraries where non-CpG methylation is very low.

## Related tools

- **bowtie2** (Underline aligner invoked by Msuite to map converted methylation reads against the Watson and Crick genome indices) — https://github.com/BenLangmead/bowtie2
- **Msuite** (Wraps bowtie2 in 3-/4-letter C-conversion modes, handles index building (build.index.sh), multimapping and PCR-duplicate filtering, and downstream methylation calling) — https://github.com/hellosunking/Msuite/
- **samtools** (Converts the SAM alignment output into BAM format) — http://samtools.sourceforge.net/

## Examples

```
Msuite -1 read1.fq -2 read2.fq -x hg19 -4 -m TAPS -o out.dir
```

## Evaluation signals

- Only reads with unique best hits survive: reads mapped to multiple locations on the same strand with equal best scores must be discarded.
- PCR duplicates (identical start and end positions and strand) collapse to the single read with the best sequencing quality.
- Per-cycle quality filtering respects -q (default 20 = 1% error rate), --minsize (default 22 bp), and methylation calls use only reads meeting -Q (default 0).
- On the SHERMAN-simulated benchmark, deduced-vs-preset methylation density residuals should be near-zero (at the simulated 0.1% sequencing-error level) for both CpG (20% preset conversion) and CpH (0.5% preset conversion).
- Indices must exist before any run and must have been built with the Lambda genome incorporated so conversion rates can be estimated.

## Limitations

- 4-letter mode ONLY supports TAPS/5hmC-CATCH data with very low non-CpG methylation; BS-seq data MUST use 3-letter mode with `-m BS`.
- No strong evidence that the 4-letter mode is more accurate at inconsistent CpG sites; 4- vs 3-letter accuracy requires further validation with additional methods such as microarrays.
- The real TAPS dataset had imperfect chemical treatment — chemical-treatment quality bounds the accuracy achievable regardless of alignment mode.
- Pre-compiled binaries (g++ v4.8.5, Linux x86_64) may need recompilation if libc++ is too old; Msuite handles at most 4,294,967,295 reads.

## Evidence

- [discussion] Even though Msuite and Bismark both utilize bowtie2 as the underline aligner, Msuite shows higher speed and lower memory usage.: "Even though Msuite and Bismark both utilize bowtie2 as the underline aligner, Msuite shows higher speed and lower memory usage."
- [discussion] it first converts all the genomic cytosines to thymines (hence, the converted genome only contains three letters: adenine, guanine, and thymine), then builds two indices for Watson and Crick chains: "it first converts all the genomic cytosines to thymines (hence, the converted genome only contains three letters: adenine, guanine, and thymine), then builds two indices for Watson and Crick chains"
