---
name: dna-methylation-call-parsing
description: Use when methylation calls from a bisulfite (BS) or bisulfite-free (TAPS)
  library have been produced by Msuite and you need per-CpG methylation densities,
  or mode-to-mode (4-letter vs 3-letter) call agreement statistics.
when_to_use_negative:
- Data are plain (non-methylation) sequencing reads or already-processed feature tables
  such as bedGraph methylation tracks
- BS-seq data would be run through 4-letter mode — the README states 4-letter mode
  ONLY supports TAPS/5hmC-CATCH data with very low non-CpG methylation; BS data MUST
  use '-3 -m BS'
- Input reads are from ATAC-me or similar protocols requiring '-k nextera' rather
  than standard methylation calling
edam_operation: http://edamontology.org/operation_3206
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0622
tools:
- name: Msuite
  role: All-in-one toolkit that integrates quality control, sequencing read alignment,
    methylation call, and data visualization; outputs Msuite.CpG.meth.call
  repo: https://github.com/hellosunking/Msuite/
- name: samtools
  role: Converts Msuite SAM output into BAM format (Msuite.final.bam) alongside methylation
    calls
  repo: http://samtools.sourceforge.net/
- name: bowtie2
  role: Underlying aligner used by Msuite for 3-letter and 4-letter read mapping
  repo: https://github.com/BenLangmead/bowtie2
- name: R
  role: Used for data visualizations and computing Pearson correlation of per-CpG
    methylation densities
provenance:
  source_task_ids:
  - task_005
  - task_008
  source_papers:
  - doi: 10.1016/j.patter.2020.100127
    title: 'Msuite: A High-Performance and Versatile DNA Methylation Data-Analysis
      Toolkit'
  source_content_hash: sha256:9f4c7b3a9490934f74921fa59d0c32042085975b7d00e8aa6f71fb3969102ae6
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
  - Mviewer
  - R
  - bowtie2
  - samtools
  edam_operation: http://edamontology.org/operation_3206
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# dna-methylation-call-parsing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parse and compare per-CpG methylation call files (e.g., Msuite.CpG.meth.call) produced by Msuite's 3-letter or 4-letter alignment modes, quantifying agreement between modes via Pearson correlation of methylation densities and the fraction of discordant CpG sites.

## When to use

Use when methylation calls from a bisulfite (BS) or bisulfite-free (TAPS) library have been produced by Msuite and you need per-CpG methylation densities, or mode-to-mode (4-letter vs 3-letter) call agreement statistics.

## When NOT to use

- Data are plain (non-methylation) sequencing reads or already-processed feature tables such as bedGraph methylation tracks
- BS-seq data would be run through 4-letter mode — the README states 4-letter mode ONLY supports TAPS/5hmC-CATCH data with very low non-CpG methylation; BS data MUST use '-3 -m BS'
- Input reads are from ATAC-me or similar protocols requiring '-k nextera' rather than standard methylation calling

## Inputs

- Paired-end FASTQ files (e.g., TAPS data from GEO GSE129673 murine ESCs)
- Msuite genome index built with build.index.sh from mm10/GRCm38 FASTA plus RefSeq annotation
- Msuite.CpG.meth.call methylation call files from 4-letter (-4 -m TAPS) and 3-letter (-3 -m TAPS) runs

## Outputs

- Per-CpG methylation density table parsed from Msuite.CpG.meth.call
- Pearson's correlation of CpG methylation densities between analysis modes
- Fraction of CpG sites with >10% methylation difference between modes
- Summary statistics CSV (e.g., msuite_mode_comparison_stats.csv) with runtime, mapped-read deficit, correlation, and discordant-CpG counts

## How to apply

Run Msuite with '-m BS' (bisulfite) or '-m TAPS' (bisulfite-free) so it integrates alignment, methylation call, and visualization into one pipeline, producing Msuite.CpG.meth.call alongside the SAM-to-BAM-converted Msuite.final.bam (via samtools). Control call stringency with '-Q score' (minimum alignment score for a read to call methylation, default 0) and '-q score' (minimum cycle quality score, default 20). Parse Msuite.CpG.meth.call to extract per-CpG methylation densities; for the 4- vs 3-letter comparison on real murine ESC TAPS data, compute Pearson's correlation of densities across shared CpGs (expect R > 0.99) and the fraction of CpGs with >10% methylation difference (expect ~3.78%). Validation follows the Msuite paper's real-data benchmark on GSE129673 (murine ESCs) aligned to GRCm38/mm10 indices built with build.index.sh.

## Related tools

- **Msuite** (All-in-one toolkit that integrates quality control, sequencing read alignment, methylation call, and data visualization; outputs Msuite.CpG.meth.call) — https://github.com/hellosunking/Msuite/
- **samtools** (Converts Msuite SAM output into BAM format (Msuite.final.bam) alongside methylation calls) — http://samtools.sourceforge.net/
- **bowtie2** (Underlying aligner used by Msuite for 3-letter and 4-letter read mapping) — https://github.com/BenLangmead/bowtie2
- **R** (Used for data visualizations and computing Pearson correlation of per-CpG methylation densities)

## Examples

```
msuite -4 -m TAPS -x mm10 -1 sample_R1.fq -2 sample_R2.fq -p 32 -o taps_4letter.out
```

## Evaluation signals

- Pearson's R between 4-letter and 3-letter TAPS CpG methylation densities exceeds 0.99
- Mapped-read deficit of 4-letter mode relative to 3-letter mode is approximately 2.17%
- Approximately 3.78% of CpG sites show >10% methylation difference between modes
- Msuite.final.bam is a valid BAM (samtools quickcheck passes) and Msuite.CpG.meth.call is non-empty with parseable chromosome/position/methylation-density columns
- The 4-letter run completes in roughly half the wall-clock time of the 3-letter run (~290 vs ~440 min on 32 threads for the murine ESC TAPS data)

## Limitations

- There is no strong evidence that the 4-letter mode is more accurate than 3-letter mode on CpG sites where the two modes disagree; further validation with methods such as microarrays is needed
- The real TAPS dataset had imperfect chemical treatment, which may propagate into methylation calls
- Only 'TAPS' and 'BS' library protocols are acceptable to '-m'; other protocols require additional support

## Evidence
