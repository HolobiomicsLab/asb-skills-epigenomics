---
name: cpg-context-cytosine-conversion
description: Use when aligning reads from bisulfite-free methylation protocols (TAPS,
  5hmC-CATCH, ATAC-me) where CpH methylation is very low, and full 3-letter C-to-T
  conversion would destroy strand-discriminating information.
when_to_use_negative:
- Data generated with BS-seq (bisulfite) protocols — the README mandates 3-letter
  mode with '-m BS'; 4-letter mode does not support it
- Samples with appreciable non-CpG methylation (e.g., plant or embryonic tissues)
  — 4-letter mode assumes non-CpG methylation is very low, as in most human somatic
  tissues
- General-purpose alignment where no cytosine-conversion semantics are needed — plain
  bowtie2 should be used instead
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3295
tools:
- name: Msuite
  role: Performs the CpG-context conversion, builds Watson/Crick indices, runs 4-letter
    alignment via '-4 -m TAPS', and applies multi-mapper and PCR-duplicate filters
  repo: https://github.com/hellosunking/Msuite/
- name: bowtie2
  role: Underlying aligner used by Msuite to search converted reads against the Watson-
    and Crick-strand indices
  repo: https://github.com/BenLangmead/bowtie2
- name: samtools
  role: Converts Msuite's SAM alignment output into BAM format
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.patter.2020.100127
    title: 'Msuite: A High-Performance and Versatile DNA Methylation Data-Analysis
      Toolkit'
  source_content_hash: sha256:41d47d70e71f4b5e75109ea84d2f8a12c2b79887cc85b49d70d62db4775b4e23
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# cpg-context-cytosine-conversion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implements Msuite's 4-letter alignment-mode conversion, in which only CpG-context cytosines (not CpH) are converted to thymines in both the reference genome and sequencing reads, preserving enough sequence information to align bisulfite-free (e.g., TAPS) reads without the 3-letter all-C-to-T collapse.

## When to use

Use when aligning reads from bisulfite-free methylation protocols (TAPS, 5hmC-CATCH, ATAC-me) where CpH methylation is very low, and full 3-letter C-to-T conversion would destroy strand-discriminating information.

## When NOT to use

- Data generated with BS-seq (bisulfite) protocols — the README mandates 3-letter mode with '-m BS'; 4-letter mode does not support it
- Samples with appreciable non-CpG methylation (e.g., plant or embryonic tissues) — 4-letter mode assumes non-CpG methylation is very low, as in most human somatic tissues
- General-purpose alignment where no cytosine-conversion semantics are needed — plain bowtie2 should be used instead

## Inputs

- Reference genome as a multi-fasta file (GENOME.FA) or a directory of per-chromosome sequences
- RefSeq annotation file (REFSEQ.txt, plain/Gzip/Bzip2) for TSS methylation QC
- Sequencing reads in FASTQ (paired-end Read1/Read2 or single-end -U) from a TAPS/5hmC-CATCH library with CpG cytosines converted to thymines

## Outputs

- Watson- and Crick-strand bowtie2 genome indices in the Msuite index directory
- Alignment output in SAM/BAM (e.g., Msuite.rmdup.sam) with multi-mappers and PCR duplicates removed
- Methylation calls and TSS visualization outputs (four_letter_verification_report.txt for conversion checks)

## How to apply

Build separate Watson- and Crick-strand bowtie2 indices from the reference using build.index.sh (GENOME.FA or GENOME.DIR, REFSEQ.txt, Genome.ID), which auto-incorporates the Lambda genome and writes to the index directory under the Msuite root; in 4-letter mode the reference conversion turns only CpG-context cytosines into thymines and leaves CpH cytosines intact, so Watson and Crick strands are no longer reverse-complementary and must be indexed and searched separately. Convert the sequencing reads the same way (CpG cytosines to T only), then run Msuite with '-4' and '-m TAPS', keeping default filtering parameters '-q 20' (minimum per-cycle quality) and '--minsize 22' (minimum read size). Execute the generated makefile ('cd OUTDIR; make') to produce the alignment and methylation outputs, then apply Msuite's filters: multi-mapping reads mapped to multiple same-strand loci with equal best scores are discarded, and reads with identical start/end/strand are treated as PCR duplicates with only the best-quality copy kept. The rationale is that CpH-context cytosines remain informative sequence letters, giving higher mapping accuracy than 3-letter alignment where all cytosines become thymines.

## Related tools

- **Msuite** (Performs the CpG-context conversion, builds Watson/Crick indices, runs 4-letter alignment via '-4 -m TAPS', and applies multi-mapper and PCR-duplicate filters) — https://github.com/hellosunking/Msuite/
- **bowtie2** (Underlying aligner used by Msuite to search converted reads against the Watson- and Crick-strand indices) — https://github.com/BenLangmead/bowtie2
- **samtools** (Converts Msuite's SAM alignment output into BAM format)

## Examples

```
build.index/build.index.sh GENOME.FA REFSEQ.txt.gz hg19; Msuite -4 -m TAPS -x hg19 -1 reads_R1.fq.gz -2 reads_R2.fq.gz -o OUTDIR; cd OUTDIR && make
```

## Evaluation signals

- four_letter_verification_report.txt exists and records that CpH-context cytosines are preserved in the converted reference genome
- Msuite.rmdup.sam contains no pairs of reads with identical start position, end position, and strand (PCR duplicates removed)
- No reads map to multiple same-strand locations with equal best scores in the final alignment
- Every read in output aligns with a per-cycle quality >= 20 (-q default) and length >= 22 bp (--minsize default)
- Watson and Crick conversions are no longer reverse-complementary; both strand indices were built and searched separately

## Limitations

- 4-letter mode only supports TAPS/5hmC-CATCH (and ATAC-me via '-k nextera') where non-CpG methylation is very low; BS-seq data must use 3-letter mode with '-m BS'
- Higher mapping accuracy of 4-letter mode was shown on benchmark (in silico SHERMAN) datasets; the paper notes no strong evidence it is more accurate on inconsistent CpG sites, and real TAPS data had imperfect chemical treatment
- The 4- vs 3-letter accuracy question requires further validation with orthogonal methods (e.g., microarrays)

## Evidence
