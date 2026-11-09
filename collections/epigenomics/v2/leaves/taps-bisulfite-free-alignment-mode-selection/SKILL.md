---
name: taps-bisulfite-free-alignment-mode-selection
description: Decision support for choosing 3-letter vs 4-letter Msuite alignment when
  input is TAPS, 5hmC-CATCH, or conventional BS-seq read data and non-CpG methylation
  levels determine the correct mode.
when_to_use_negative:
- Data generated with a conventional bisulfite (BS-seq) protocol — the README states
  4-letter mode only supports TAPS/5hmC-CATCH data where non-CpG methylation is very
  low, so BS data MUST use 3-letter mode with -m BS
- Data from tissues with substantial non-CpG methylation (e.g., many embryonic/germinal
  cell types), since the 4-letter mode's assumption of very low CpH methylation would
  be violated
- ATAC-me or similar protocols that instead require setting -k nextera for library-kit
  handling rather than a 3-letter/4-letter mode decision
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_3168
tools:
- name: Msuite
  role: Provides the -3 (3-letter) and -4 (4-letter) alignment modes and the -m TAPS/BS
    protocol flag whose correct combination this skill decides
  repo: https://github.com/hellosunking/Msuite/
- name: bowtie2
  role: Underlying aligner used by Msuite in both modes; the article notes Msuite
    still achieves higher speed and lower memory than Bismark despite the same aligner
  repo: https://github.com/BenLangmead/bowtie2
- name: samtools
  role: Converts Msuite SAM alignment output into BAM format in the downstream pipeline
  repo: http://samtools.sourceforge.net/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2020.100127
    title: 'Msuite: A High-Performance and Versatile DNA Methylation Data-Analysis
      Toolkit'
  source_content_hash: sha256:e6ad100f5a915d82ff19fcb958989803377cf9f2aa93716e60da4aa4d5bcf350
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
  - BWA-meth
  - Bismark
  - Msuite
  - bowtie2
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# taps-bisulfite-free-alignment-mode-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Guides selection between Msuite's 3-letter (bisulfite-style cytosine-to-thymine conversion) and 4-letter alignment modes when analyzing DNA methylation libraries, particularly bisulfite-free TAPS/5hmC-CATCH data where the 4-letter mode is specifically optimized.

## When to use

Decision support for choosing 3-letter vs 4-letter Msuite alignment when input is TAPS, 5hmC-CATCH, or conventional BS-seq read data and non-CpG methylation levels determine the correct mode.

## When NOT to use

- Data generated with a conventional bisulfite (BS-seq) protocol — the README states 4-letter mode only supports TAPS/5hmC-CATCH data where non-CpG methylation is very low, so BS data MUST use 3-letter mode with -m BS
- Data from tissues with substantial non-CpG methylation (e.g., many embryonic/germinal cell types), since the 4-letter mode's assumption of very low CpH methylation would be violated
- ATAC-me or similar protocols that instead require setting -k nextera for library-kit handling rather than a 3-letter/4-letter mode decision

## Inputs

- Paired-end or single-end FASTQ sequencing reads from TAPS, 5hmC-CATCH, or BS-seq libraries
- Reference genome sequence (multi-FASTA file or per-chromosome directory, e.g., mm9/mm10/hg19/hg38)
- RefSeq annotation file (for TSS methylation QC; gzip/bzip2 compressed accepted with proper suffix)

## Outputs

- Aligned reads (SAM converted to BAM via samtools) with unique-best-hit and PCR-duplicate filtering
- Methylation calls at cytosine sites (only reads meeting -Q minimum alignment score are called)
- Quality-control output including methylation profiling around RefSeq transcription start sites
- TSS visualization output generated with R

## How to apply

First determine the library protocol: for BS-seq data, Msuite MUST be run in 3-letter mode (-3, the default) with -m BS, because the converted genome contains only adenine, guanine, and thymine; for TAPS or 5hmC-CATCH data where non-CpG methylation is very low (e.g., most somatic tissues in human), the 4-letter mode (-4 with -m TAPS) is appropriate and preserves higher nucleotide complexity. Build genome indices beforehand with build.index/build.index.sh, supplying the genome FASTA (or directory), RefSeq annotation, and a Genome.ID; the utility automatically incorporates a Lambda genome. Run the msuite command with the chosen mode flag (-3 is mutually exclusive with -4), the index via -x, reads via -1/-2 (paired-end) or -U (single-end), and -m TAPS/BS. Decision rationale: the 4-letter mode showed higher mapping accuracy on benchmark datasets and completed much faster on real TAPS data, but the article notes there is no strong evidence it is more accurate on inconsistent CpG sites, and validation with orthogonal methods (e.g., microarrays) is still pending. If uncertain about protocol identity, default to 3-letter mode, which also enables direct comparison with Bismark and BWA-meth.

## Related tools

- **Msuite** (Provides the -3 (3-letter) and -4 (4-letter) alignment modes and the -m TAPS/BS protocol flag whose correct combination this skill decides) — https://github.com/hellosunking/Msuite/
- **bowtie2** (Underlying aligner used by Msuite in both modes; the article notes Msuite still achieves higher speed and lower memory than Bismark despite the same aligner) — https://github.com/BenLangmead/bowtie2
- **samtools** (Converts Msuite SAM alignment output into BAM format in the downstream pipeline) — http://samtools.sourceforge.net/

## Examples

```
msuite -4 -m TAPS -x hg19 -1 read1.fq -2 read2.fq -o out.dir
```

## Evaluation signals

- Mode flags are mutually exclusive: the invocation contains exactly one of -3 or -4, and -4 never co-occurs with BS-seq input data
- For TAPS data the command includes -m TAPS; for bisulfite data -m BS with 3-letter mode only
- On simulated TAPS benchmarks, 4-letter mode should show higher mapping accuracy than 3-letter (Msuite 4-letter 95.75% vs 3-letter 96.19% mapping efficiency on 100-bp reads per Table 2 tolerance), with Msuite faster and lower-memory than Bismark and BWA-meth
- Multi-mapping reads with equal best scores on the same strand are discarded and PCR duplicates (identical start/end/strand) reduced to the single best-quality read in the output
- Index build directory contains Watson and Crick chain indices plus an automatically incorporated Lambda genome for the specified Genome.ID

## Limitations

- Although the 4-letter mode demonstrated higher mapping accuracy on benchmark datasets, the authors found no strong evidence that it is more accurate on inconsistent CpG sites
- Accuracy of the 4- and 3-letter modes needs further validation with additional methods such as microarrays
- 4-letter mode only supports TAPS/5hmC-CATCH data where non-CpG methylation is very low (e.g., most somatic tissues in human)
- On a real TAPS dataset with imperfect chemical treatment, the 4-letter mode still produced high-quality results, but chemical-treatment quality remains a caveat

## Evidence
