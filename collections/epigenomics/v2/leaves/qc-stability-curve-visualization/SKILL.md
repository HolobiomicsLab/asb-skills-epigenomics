---
name: qc-stability-curve-visualization
description: 'Use when evaluating whether a ChIP-seq QC metric such as FRiP is an
  artifact of sequencing depth: you have down-sampled BAMs (e.g., 1M, 2M, 4M reads)
  and want stability curves under two peak-calling settings.'
when_to_use_negative:
- You only need a single FRiP value per sample at full sequencing depth (ChiLin step
  7) rather than a depth-stability comparison
- Your QC metric is depth-independent by construction (e.g., PBC, FRiT-like ratios
  over fixed regions) so a stability curve adds no information
- You need QC reporting for batched samples rather than a methodological stability
  analysis
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0092
tools:
- name: BWA
  role: Maps raw FASTQ files of the selected low/mid/high-quality samples to hg19
    to produce the sorted BAM inputs
  repo: https://github.com/lh3/bwa
- name: seqtk
  role: Sub-samples reads at stepwise depths (from 1M reads) to generate the down-sampled
    BAM series
  repo: https://github.com/lh3/seqtk
- name: MACS2
  role: Calls narrow peaks per down-sampled BAM (Setting A) or once on the full-depth
    BAM (Setting B, narrow type per ChiLin TF default)
  repo: https://github.com/taoliu/MACS
- name: ChiLin
  role: Provides the sub-sampling convention, MACS2 defaults, and FRiP score calculation
    (step 7) with *.frip outputs
  repo: https://github.com/cfce/chilin
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-016-1274-4
    title: 'ChiLin: a comprehensive ChIP-seq and DNase-seq quality control and analysis
      pipeline'
  source_content_hash: sha256:aae450365b9690012a1810b579a6bff740d18ec8413f76069a014c8ebd0e0a75
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/bwa
  tools:
  - BWA
  - ChiLin
  - MACS2
  - seqtk
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# qc-stability-curve-visualization

## Summary

Constructs FRiP-versus-sequencing-depth stability curves by holding peak regions fixed across down-sampled BAM depths, distinguishing QC metrics that remain stable (peaks from all reads, Fig. 2e style) from metrics that degrade at shallow depth (peaks re-called per depth, Fig. 2d style).

## When to use

Use when evaluating whether a ChIP-seq QC metric such as FRiP is an artifact of sequencing depth: you have down-sampled BAMs (e.g., 1M, 2M, 4M reads) and want stability curves under two peak-calling settings.

## When NOT to use

- You only need a single FRiP value per sample at full sequencing depth (ChiLin step 7) rather than a depth-stability comparison
- Your QC metric is depth-independent by construction (e.g., PBC, FRiT-like ratios over fixed regions) so a stability curve adds no information
- You need QC reporting for batched samples rather than a methodological stability analysis

## Inputs

- Raw FASTQ files of low/mid/high-quality ChIP-seq samples
- Sorted BAM files mapped to hg19
- Stepwise down-sampled BAM files (from 1M reads upward, e.g., ChiLin's 4M-read subsampled BAMs)
- MACS2 narrow-peak BED files (per-depth under Setting A; single full-depth set under Setting B)

## Outputs

- Table of FRiP scores per sample per sequencing depth per setting (id_frip./*.frip style)
- Stability curve plots of FRiP versus sequencing depth, overlaying both settings, stratified by sample quality class

## How to apply

Map raw FASTQ files to hg19 with BWA to obtain sorted BAMs, then sub-sample each BAM at stepwise depths (starting at 1M reads, following the ChiLin sub-sampling convention such as its 4M-read outputs). Run two settings: Setting A calls narrow peaks with MACS2 independently on each down-sampled BAM, while Setting B calls peaks once on the full-depth BAM and holds those peak regions fixed across all depths. Compute FRiP scores (fraction of reads falling in peaks, ChiLin step 7) for every depth under both settings, collect the values into a table in the style of id_frip/*.frip outputs, then plot FRiP versus sequencing depth as stability curves per sample quality class (low/mid/high), overlaying the two settings. Accept the result only if Setting B curves are approximately flat while Setting A curves degrade at shallow depths, reproducing the reported Fig. 2d/2e pattern.

## Related tools

- **BWA** (Maps raw FASTQ files of the selected low/mid/high-quality samples to hg19 to produce the sorted BAM inputs) — https://github.com/lh3/bwa
- **seqtk** (Sub-samples reads at stepwise depths (from 1M reads) to generate the down-sampled BAM series) — https://github.com/lh3/seqtk
- **MACS2** (Calls narrow peaks per down-sampled BAM (Setting A) or once on the full-depth BAM (Setting B, narrow type per ChiLin TF default)) — https://github.com/taoliu/MACS
- **ChiLin** (Provides the sub-sampling convention, MACS2 defaults, and FRiP score calculation (step 7) with *.frip outputs) — https://github.com/cfce/chilin

## Examples

```
macs2 callpeak -t sub_4M.bam -f BAM -n sub_4M -g hs --nomodel; chilin fridge.py -i sub_4M.bam -p full_depth_peaks.bed
```

## Evaluation signals

- Setting B FRiP curves are approximately flat across all depths for every quality class (low/mid/high)
- Setting A FRiP degrades at shallow depths, matching the reported Fig. 2d behavior
- The FRiP table contains one value per sample per depth per setting, in id_frip./*.frip output style
- Sub-sampled BAMs follow the ChiLin convention of stepwise depths starting at 1M reads (e.g., 4M-read subsampled BAMs)
- Peak calls are narrow type (MACS2 narrow), matching ChiLin's TF default

## Limitations

- Stability conclusions depend on the chosen down-sampling grid and peak-caller parameters; the pattern shown is specific to MACS2 narrow peaks and hg19 mapping as used in the article
- ChiLin is extensible but does not state a software version, so exact command-level reproducibility of the FRiP stability comparison requires pinning dependencies (e.g., bwa=0.7.13, samtools=0.1.19, macs2=2.1.0 in the README conda setup)
- ChiLin automatically skips some steps for very few peaks (e.g., motif analysis when peaks < 200), which signals that shallow-depth samples with very few MACS2 peaks may produce unreliable FRiP values

## Evidence

- [methods] Setting B uses full-depth peaks: "step 7: calculate FRiP scores for each sample."
- [methods] Sub-sampling convention: "step 3: sub-sample bam files and do macs2 fragment size estimation."
- [intro] MACS2 default peak caller: "ChiLin uses MACS2 (https://github.com/taoliu/MACS/) as the default peak caller"
- [readme] seqtk sub-sampling: [evidence span withheld — anchor status: external_doc]
- [readme] Pinned tool versions in ChiLin environment: "conda install bwa=0.7.13 samtools=0.1.19 bedtools=2.17.0 seqtk ucsc-bedclip ucsc-bedgraphtobigwig ucsc-wigcorrelate ucsc-wigtobigwig fastqc numpy macs2=2.1.0"
