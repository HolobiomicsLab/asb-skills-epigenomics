---
name: macs2-peak-calling
description: Use when ChIP-seq reads are already mapped to BAMs and QC requires MACS2
  peak calls (narrow mode for TFs/active marks, broad for histone marks) to drive
  FRiP, replicate overlap, and annotation metrics.
when_to_use_negative:
- 'Input is raw FASTQ that has not been mapped: BWA/bowtie/star mapping (step 2) cannot
  be skipped and must supply the BAM files MACS2 requires.'
- Only sequence-level QC is needed (FastQC GC content and library contamination, step
  1) — peak calling adds no value without mapped BAMs.
- The goal is down-sampling stability analysis of FRiP depth curves (Fig. 2d/2e) using
  seqtk — that requires a separate sub-sampling series rather than the fixed 4M-read
  subsample.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3168
tools:
- name: MACS2
  role: 'Default peak caller (2.1.0 in ChiLin env): predictd fragment size estimation
    on subsampled BAMs, narrow/broad peak calling on replicates and merged BAMs with
    the configured FDR cutoff and keep_dup=1, and bedgraph output for bigWig conversion'
  repo: https://github.com/taoliu/MACS/
- name: ChiLin
  role: Pipeline that orchestrates the MACS2 steps (3, 5, 6) via 'chilin simple' and
    compiles the resulting peak metrics into id_summary.txt and id_report.pdf
  repo: https://github.com/cfce/chilin
- name: BWA
  role: Default mapper (step 2, cannot be skipped) producing the BAM inputs that MACS2
    peak calling consumes
  repo: https://github.com/lh3/bwa
provenance:
  source_task_ids:
  - task_007
  - task_008
  - task_003
  source_papers:
  - doi: 10.1186/s12859-016-1274-4
    title: 'ChiLin: a comprehensive ChIP-seq and DNase-seq quality control and analysis
      pipeline'
  source_content_hash: sha256:04bd9d294abcb834aeebad18a00d6dc33d77aa0e159610452170c208cf3e8cc9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/taoliu/MACS
  tools:
  - BWA
  - ChiLin
  - MACS2
  - bedClip
  - bedGraphToBigWig
  - bedtools
  - samtools
  - seqtk
  - wigCorrelate
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# macs2-peak-calling

## Summary

Runs MACS2 (ChiLin's default peak caller, pinned to 2.1.0) to estimate fragment size from 4M-read subsampled BAMs and call narrow/broad peaks on replicates and merged BAMs, producing the narrowPeak/broadPeak files that feed FRiP scoring, replicate overlap/correlation, and meta-region annotation QC.

## When to use

Use when ChIP-seq reads are already mapped to BAMs and QC requires MACS2 peak calls (narrow mode for TFs/active marks, broad for histone marks) to drive FRiP, replicate overlap, and annotation metrics.

## When NOT to use

- Input is raw FASTQ that has not been mapped: BWA/bowtie/star mapping (step 2) cannot be skipped and must supply the BAM files MACS2 requires.
- Only sequence-level QC is needed (FastQC GC content and library contamination, step 1) — peak calling adds no value without mapped BAMs.
- The goal is down-sampling stability analysis of FRiP depth curves (Fig. 2d/2e) using seqtk — that requires a separate sub-sampling series rather than the fixed 4M-read subsample.

## Inputs

- Treatment and control ChIP-seq replicate BAM files mapped with BWA and filtered for unique reads (mapping quality above 1)
- 4M-read subsampled BAM files (id_*_4000000.bam) for fragment size estimation
- Peak-caller configuration: narrow vs broad mode (-p), FDR cutoff, keep_dup=1, e.g. via 'chilin simple -p narrow -r tf'
- Species-specific dependent data (e.g. hg19.tgz with chromosome length and union DHS/blacklist regions) for downstream peak annotation

## Outputs

- Per-replicate and merged narrowPeak/broadPeak peak files (e.g. id_peaks.narrowPeak)
- MACS2 fragment size estimate from the predictd run on 4M-read subsampled BAMs
- MACS2 bedgraph-derived bigWig signal tracks (via bedClip and bedGraphToBigWig) used for wigCorrelate replicate consistency
- FRiP files per sample (e.g. id_treat_rep1.frip) and replicate overlap/correlation outputs (id_0_1.overlap, id.cor)

## How to apply

After BWA mapping (step 2, which cannot be skipped because it supplies the BAM files) and filtering for unique reads (samtools mapping quality above 1), subsample each replicate BAM to 4,000,000 reads (id_*_4000000.bam) and run a MACS2 predictd pass on these subsamples to estimate fragment size (step 3). Call peaks with MACS2 per replicate (step 5) — narrow mode for TFs and active marks, broad mode for broad histone marks, e.g. 'chilin simple ... -p narrow -r tf' — using the configured FDR cutoff and keep_dup=1. Then call peaks on the merged BAM (step 6), which cannot be skipped because its peaks feed the bedAnnotate.py meta-region annotation (step 8). Convert MACS2 bedgraphs to bigWigs with bedClip and bedGraphToBigWig so UCSC wigCorrelate can compute Pearson correlations of per-million-normalized signal between replicates. Intersect per-replicate peak files with the 4M-read subsampled BAMs using bedtools to obtain FRiP scores (step 7) and compute inter-replicate peak-overlap outputs (id_0_1.overlap, id.cor).

## Related tools

- **MACS2** (Default peak caller (2.1.0 in ChiLin env): predictd fragment size estimation on subsampled BAMs, narrow/broad peak calling on replicates and merged BAMs with the configured FDR cutoff and keep_dup=1, and bedgraph output for bigWig conversion) — https://github.com/taoliu/MACS/
- **ChiLin** (Pipeline that orchestrates the MACS2 steps (3, 5, 6) via 'chilin simple' and compiles the resulting peak metrics into id_summary.txt and id_report.pdf) — https://github.com/cfce/chilin
- **BWA** (Default mapper (step 2, cannot be skipped) producing the BAM inputs that MACS2 peak calling consumes) — https://github.com/lh3/bwa

## Examples

```
chilin simple -u user -s species --threads 8 -i id -o output -t treat1.fastq,treat2.fastq -c control1.fastq,control2.fastq -p narrow -r tf
```

## Evaluation signals

- Step 6 merged-BAM calling yields a non-empty narrowPeak/broadPeak file (e.g. id_peaks.narrowPeak) that the bedAnnotate.py meta-region step consumes.
- FRiP outputs (id_treat_rep1.frip, id_ctrl_rep1.frip) exist for every replicate, computed by bedtools intersection of peaks with the 4M-read subsampled BAMs.
- Replicate consistency outputs id_0_1.overlap and id.cor are produced, with wiggle correlations comparable to the paper's references (e.g. TRRAP 0.14, H3K4me3 0.02).
- The fragment size comes from a MACS2 predictd run on 4M-read subsampled BAMs (step 3) rather than an arbitrary user-supplied value.
- Case-study QC values (e.g. H3K27me3 FRiP 0.04-0.32%, STAT6 FRiP <1% with 45 peaks) fall within the paper's reported reference ranges.

## Limitations

- Merged-BAM peak calling (step 6) cannot be skipped because it provides the peaks required for the annotation step.
- MACS2 is pinned to version 2.1.0 in the ChiLin conda environment; other versions may not reproduce the paper's QC values.
- Fragment size estimation runs on 4M-read subsampled BAMs, and downstream MDSeqPos motif analysis is auto-skipped when the peak number is under 200.

## Evidence

- [intro] macs2_default_caller: "ChiLin uses MACS2 (https://github.com/taoliu/MACS/) as the default peak caller"
- [methods] step3_fragment_size: "step 3: sub-sample bam files and do macs2 fragment size estimation."
- [methods] step5_replicate_calling: "step 5: call peak for replicates samples, and do replicates peak overlap/correlation analysis"
- [methods] step6_merged_calling: "step 6: call peak for merged bam file, this step cannot be skipped, because this provided peak for annotation step"
- [readme] readme_macs2_version: "conda install bwa=0.7.13 samtools=0.1.19 bedtools=2.17.0 seqtk ucsc-bedclip ucsc-bedgraphtobigwig ucsc-wigcorrelate ucsc-wigtobigwig fastqc numpy macs2=2.1.0 bioconductor-seqlogo"
