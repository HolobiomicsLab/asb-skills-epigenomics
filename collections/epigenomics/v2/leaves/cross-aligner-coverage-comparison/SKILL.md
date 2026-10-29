---
name: cross-aligner-coverage-comparison
description: Use when two alignment sets (e.g., a Msuite.final.bam and published reference
  BAMs) exist for the same murine ESC TAPS/WGBS samples and you need region-level
  (CpG island) read-coverage comparison rather than overall mapping-efficiency metrics.
when_to_use_negative:
- Input is BS-seq (bisulfite) data processed with Msuite's 4-letter mode — per the
  README, BS-protocol data MUST use 3-letter mode with -m BS, so coverage gains inferred
  this way would not reflect the intended 4-letter/TAPS configuration
- The goal is whole-genome mapping-efficiency, speed, or memory benchmarking against
  Bismark/BWA-meth rather than region-resolved (CpG island) coverage comparison
- The two alignment sets derive from different samples or different reference assemblies
  — per-island coverage ratios would confound biological and aligner differences
tools:
- name: Msuite
  role: Provides extract.meth.in.region to extract per-CpG C-count/T-count within
    CpG island intervals, and produced the candidate Msuite.final.bam being compared
  repo: https://github.com/hellosunking/Msuite/
- name: samtools
  role: Used alongside extract.meth.in.region to query the two alignment sets (SAM/BAM
    handling) when extracting covered CpG sites per island
  repo: http://samtools.sourceforge.net/
- name: R
  role: Used for data visualization of per-island coverage (ratio distribution/box
    plot) comparing Msuite versus reference alignments
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.patter.2020.100127
    title: 'Msuite: A High-Performance and Versatile DNA Methylation Data-Analysis
      Toolkit'
  source_content_hash: sha256:9b659e869a793b4f397a5c75b6dc149bb8d3fb5d33f21bca9d924fcc10b2ca19
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
  - R
  - samtools
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# cross-aligner-coverage-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how mapped-read coverage over CpG islands differs between Msuite alignments and a published alignment strategy (Liu et al.) for the same TAPS/WGBS samples on the mm10/GRCm38 reference, isolating the gain on highly methylated islands.

## When to use

Use when two alignment sets (e.g., a Msuite.final.bam and published reference BAMs) exist for the same murine ESC TAPS/WGBS samples and you need region-level (CpG island) read-coverage comparison rather than overall mapping-efficiency metrics.

## When NOT to use

- Input is BS-seq (bisulfite) data processed with Msuite's 4-letter mode — per the README, BS-protocol data MUST use 3-letter mode with -m BS, so coverage gains inferred this way would not reflect the intended 4-letter/TAPS configuration
- The goal is whole-genome mapping-efficiency, speed, or memory benchmarking against Bismark/BWA-meth rather than region-resolved (CpG island) coverage comparison
- The two alignment sets derive from different samples or different reference assemblies — per-island coverage ratios would confound biological and aligner differences

## Inputs

- Msuite.final.bam from the real-data (TAPS/WGBS murine ESC) alignment task
- Published reference alignments for the same murine samples (Liu et al.; GEO GSE129673)
- CpG island annotation for the mm10/GRCm38 assembly
- Msuite installation with the extract.meth.in.region utility and built mm10 genome indices (built via build.index.sh from genome FASTA + RefSeq annotation)

## Outputs

- Per-CpG extracted methylation records (C-count and T-count) for every CpG island interval, for both alignment sets
- Merged per-island coverage comparison table (coverage normalized by island length, with hypermethylated islands flagged)
- R visualization (ratio distribution / box plot of Msuite-vs-reference per-island coverage, analogous to Figure S3)

## How to apply

Load the Msuite.final.bam produced by the real-data alignment task, the published reference alignments for the same murine samples (GSE129673 murine ESCs), and the CpG island annotation for mm10/GRCm38 (files for mm9/mm10/hg19/hg38 are bundled with Msuite). Run the Msuite utility extract.meth.in.region together with samtools to extract covered CpG sites with their C-count and T-count inside every CpG island interval for BOTH alignment sets, then aggregate per-island read/CpG coverage normalized by island length and merge into a single comparison table, flagging hypermethylated islands separately. In R, visualize the per-island coverage of Msuite versus the reference alignments (e.g., ratio distribution or box plot highlighting hypermethylated islands), analogous to the supplementary Figure S3. The rationale is that mapping gains from Msuite's screening of multi-mapping reads and PCR duplicates (only unique best hits kept; only the best-quality read among identical start/end/strand duplicates kept) should concentrate in CpG-dense, highly methylated regions that 3-letter bisulfite-style alignment resolves poorly.

## Related tools

- **Msuite** (Provides extract.meth.in.region to extract per-CpG C-count/T-count within CpG island intervals, and produced the candidate Msuite.final.bam being compared) — https://github.com/hellosunking/Msuite/
- **samtools** (Used alongside extract.meth.in.region to query the two alignment sets (SAM/BAM handling) when extracting covered CpG sites per island) — http://samtools.sourceforge.net/
- **R** (Used for data visualization of per-island coverage (ratio distribution/box plot) comparing Msuite versus reference alignments)

## Examples

```
build.index/build.index.sh GENOME.FA(or GENOME.DIR) REFSEQ.txt Genome.ID
```

## Evaluation signals

- Msuite alignments show greater per-island read/CpG coverage than the reference alignments after normalization by island length
- The enrichment is largest on hypermethylated CpG islands, with the qualitative pattern of supplementary Figure S3 reproduced
- The merged comparison table contains an entry for every CpG island interval from the mm10/GRCm38 annotation in both alignment sets
- Per-CpG C-count/T-count extraction succeeded for both alignment sets (extract.meth.in.region plus samtools ran over Msuite.final.bam and the published BAMs)
- Per-island coverage ratios cluster near or above 1, with the right tail (gains) concentrated in the flagged hypermethylated island subset

## Limitations

- The real TAPS dataset had imperfect chemical treatment, which can affect coverage and methylation-call quality even when the 4-letter mode still produces high-quality, fast results
- No strong evidence exists that the 4-letter mode is more accurate on inconsistent CpG sites; validation with additional methods (e.g., microarrays) is still required
- Msuite discards reads that map to multiple locations with equal best scores and PCR duplicates with identical start/end/strand, so coverage differences partly reflect read filtering policy, not only alignment accuracy

## Evidence

- [supplementary] Msuite yields greater read coverage on CpG islands in the mouse genome, with the largest gains observed for highly methylated CpG islands (Figure S3).: [evidence span withheld — anchor status: not_found]
- [methods] Use the Msuite utility extract.meth.in.region together with samtools to extract covered CpG sites with C-count and T-count within every CpG island interval for both alignment sets.: [evidence span withheld — anchor status: not_found]
