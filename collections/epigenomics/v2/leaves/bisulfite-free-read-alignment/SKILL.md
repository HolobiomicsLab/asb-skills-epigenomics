---
name: bisulfite-free-read-alignment
description: Use when aligning reads from bisulfite-free protocols such as TAPS or
  5hmC-CATCH where only CpG-context cytosines are converted to thymines and non-CpG
  methylation is very low, so the reads retain 4-letter alphabet information.
when_to_use_negative:
- 'Data generated with BS-seq (bisulfite) protocols: the README states you MUST use
  3-letter mode with ''-m BS'', since 4-letter mode relies on CpG-specific conversion'
- 'Libraries with substantial non-CpG methylation (e.g., ESCs/germline for 5hmC-CATCH-type
  data): 4-letter mode only supports data where non-CpG methylation is very low'
- ATAC-me or similar assays where the user only wants cycle-restricted analysis without
  methylation call — use '--align-only' or '-k nextera' variants instead of assuming
  default 4-letter behavior
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3295
tools:
- name: Msuite
  role: All-in-one toolkit performing the 4-letter read alignment, multi-mapper and
    PCR-duplicate filtering, and methylation call
  repo: https://github.com/hellosunking/Msuite/
- name: bowtie2
  role: Underline aligner invoked by Msuite against the Watson- and Crick-strand converted
    genome indices
  repo: https://github.com/BenLangmead/bowtie2
- name: samtools
  role: Converts Msuite SAM alignment output into BAM format (Msuite.final.bam)
  repo: http://samtools.sourceforge.net/
provenance:
  source_task_ids:
  - task_007
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2020.100127
    title: 'Msuite: A High-Performance and Versatile DNA Methylation Data-Analysis
      Toolkit'
  source_content_hash: sha256:14ad555e4db438d1eb5d2f400eab89e4f7d3e95cd1790cf0e4fd6cb723b3218a
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
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# bisulfite-free-read-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Align sequencing reads from bisulfite-free methylation assays (e.g., TAPS, 5hmC-CATCH) using Msuite's 4-letter mode, which preserves CpG-context cytosines instead of converting all cytosines to thymines, enabling faster and more accurate alignment than 3-letter approaches.

## When to use

Use when aligning reads from bisulfite-free protocols such as TAPS or 5hmC-CATCH where only CpG-context cytosines are converted to thymines and non-CpG methylation is very low, so the reads retain 4-letter alphabet information.

## When NOT to use

- Data generated with BS-seq (bisulfite) protocols: the README states you MUST use 3-letter mode with '-m BS', since 4-letter mode relies on CpG-specific conversion
- Libraries with substantial non-CpG methylation (e.g., ESCs/germline for 5hmC-CATCH-type data): 4-letter mode only supports data where non-CpG methylation is very low
- ATAC-me or similar assays where the user only wants cycle-restricted analysis without methylation call — use '--align-only' or '-k nextera' variants instead of assuming default 4-letter behavior

## Inputs

- Paired-end or single-end FASTQ reads from TAPS/5hmC-CATCH libraries (e.g., GEO GSE129673 murine ESC TAPS data)
- Multi-fasta reference genome file or directory of chromosome sequences (e.g., GRCm38/mm10)
- RefSeq annotation file (mm9/mm10/hg19/hg38 included in the Msuite package)

## Outputs

- Msuite.final.bam (final mapped reads, via samtools SAM-to-BAM conversion)
- Msuite.rmdup.sam (duplicate-filtered alignments)
- Msuite.CpG.meth.call (per-CpG methylation calls)
- TSS methylation profile quality control and visualizations around RefSeq transcription start sites

## How to apply

First build genome indices with build.index.sh GENOME.FA REFSEQ.txt Genome.ID, which incorporates the Lambda genome and writes indices to the index directory under the Msuite root; in 4-letter mode the converted reference keeps separate Watson- and Crick-strand indices where only CpG-context cytosines become thymines (CpH cytosines remain unchanged), because chemical conversion has disrupted the reverse-complementary relationship between strands. Run 'msuite -4 -m TAPS -x index -1 read1.fq [-2 read2.fq] -p threads -o out.dir' letting Msuite auto-detect the sequencing cycle; default filters keep cycles with quality score >= 20 (-q 20) and reads of at least 22 bp (--minsize 22). Alignment uses bowtie2 as the underline aligner; multi-mapping reads that map to multiple locations on the same strand with equal best scores are discarded so only unique best hits are kept, and aligned reads with identical start/end positions and strand are treated as PCR duplicates keeping only the best-quality read. Execute the generated makefile ('cd OUTDIR; make') to produce the alignment (Msuite.final.bam/Msuite.rmdup.sam) and methylation outputs (Msuite.CpG.meth.call). Benchmarking on murine ESC TAPS data (GSE129673) against GRCm38 showed 4-letter mode is ~50% faster (~290 min vs ~440 min on 32 threads) with only a 2.17% mapped-read deficit and CpG methylation densities correlated at Pearson's R > 0.99 with 3-letter mode.

## Related tools

- **Msuite** (All-in-one toolkit performing the 4-letter read alignment, multi-mapper and PCR-duplicate filtering, and methylation call) — https://github.com/hellosunking/Msuite/
- **bowtie2** (Underline aligner invoked by Msuite against the Watson- and Crick-strand converted genome indices) — https://github.com/BenLangmead/bowtie2
- **samtools** (Converts Msuite SAM alignment output into BAM format (Msuite.final.bam)) — http://samtools.sourceforge.net/

## Examples

```
msuite -4 -m TAPS -x mm10 -1 reads_R1.fq -2 reads_R2.fq -p 32 -o OUTDIR && cd OUTDIR && make
```

## Evaluation signals

- Converted reference genomes retain cytosines at CpH positions (only CpG-context cytosines converted to thymine), e.g., recorded in a four_letter_verification_report.txt
- Msuite.rmdup.sam contains no reads with identical start/end/strand pairs (all PCR duplicates removed, best-quality kept)
- No reads mapping to multiple locations on the same strand with equal best scores remain in the output (unique-best-hit rule enforced)
- On murine ESC TAPS data, CpG methylation densities of 4-letter vs 3-letter runs should give Pearson's R > 0.99 with roughly 3.78% of CpG sites showing >10% methylation difference
- Mapped-read deficit of 4-letter vs 3-letter mode should be approximately 2.17% on comparable real TAPS data

## Limitations

- Although 4-letter mode demonstrated higher mapping accuracy on benchmark datasets (SHERMAN-simulated paired-/single-end 36-/100-bp reads), there is no strong evidence it is more accurate on inconsistent CpG sites; further validation against methods like microarrays is needed
- 4-letter mode supports only TAPS/5hmC-CATCH data where non-CpG methylation is very low; BS-seq data must go through 3-letter mode ('-m BS')
- On real TAPS data the chemical treatment was imperfect, so mapping efficiency is slightly lower than 3-letter mode (2.17% deficit on the murine ESC dataset)

## Evidence

- [discussion] reads that could be mapped to multiple locations in the same strand with equal best scores are discarded: "Msuite screens for multiple-mapping reads and only those with unique best hits are kept"
- [discussion] identical start and end positions and strand information as PCR duplicates: "Msuite also looks for aligned reads that have identical start and end positions and strand information as PCR duplicates, and only keeps the one with the best sequencing read quality."
- [readme] 4-letter mode ONLY supports processing of TAPS/5hmC-CATCH data where the non-CpG methylation is very low: "If your data is generated using BS-seq protocol, you MUST use the 3-letter mode and set `-m BS`."
- [readme] build.index.sh GENOME.FA(or GENOME.DIR) REFSEQ.txt Genome.ID: "Note that this utility will automatically incorporate the Lambda genome to build the genome indices."
- [readme] -q score         The minimum quality score to keep the cycle (default: 20): "--minsize size   Minimum read size to be kept for alignment (default: 22)"
- [discussion] Even though Msuite and Bismark both utilize bowtie2 as the underline aligner, Msuite shows higher speed and lower memory usage.: "Even though Msuite and Bismark both utilize bowtie2 as the underline aligner, Msuite shows higher speed and lower memory usage."
