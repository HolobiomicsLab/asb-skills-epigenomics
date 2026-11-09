---
name: frip-score-calculation-on-subsampled-bams
description: 'Use when QC of ChIP-seq replicates requires FRiP scores: MACS2 narrow/broad
  peaks are available and read enrichment in peaks must be quantified on equally subsampled
  BAMs.'
when_to_use_negative:
- Input data is DNase-seq processed through the DNase-seq QC branch rather than ChIP-seq
  peak-based QC, where this peak-overlap FRiP step is not the defining metric
- Peaks have not yet been called — FRiP requires MACS2 narrowPeak/broadPeak files,
  so running it on unmapped or unfiltered BAMs alone is wrong
- BAMs are full-depth and not subsampled to 4,000,000 reads, making FRiP values incomparable
  across replicates with different library depths
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0092
tools:
- name: ChiLin
  role: Pipeline that orchestrates subsampling, peak calling, and FRiP calculation
    (step 7) and compiles the QC summary
  repo: https://github.com/cfce/chilin
- name: MACS2
  role: Default peak caller producing the narrowPeak/broadPeak files that define the
    peak regions used for FRiP
  repo: https://github.com/taoliu/MACS/
- name: bedtools
  role: Intersects peak files with the 4M-read subsampled BAMs to compute the fraction
    of reads in peaks
- name: samtools
  role: Retains uniquely mapped reads (mapping quality above 1) and supports subsampling
    BAMs before FRiP
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-016-1274-4
    title: 'ChiLin: a comprehensive ChIP-seq and DNase-seq quality control and analysis
      pipeline'
  source_content_hash: sha256:98146514606a2b9df141ca531c1bd005c1bb4eb9a62bea65f7af49e77f6fdcb9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfce/chilin
  tools:
  - ChiLin
  - MACS2
  - bedClip
  - bedGraphToBigWig
  - bedtools
  - samtools
  - wigCorrelate
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# frip-score-calculation-on-subsampled-bams

## Summary

Compute the Fraction of Reads in Peaks (FRiP) QC metric for each ChIP-seq treatment and control replicate by intersecting MACS2 peak calls with 4,000,000-read subsampled BAMs, as implemented in the ChiLin pipeline.

## When to use

Use when QC of ChIP-seq replicates requires FRiP scores: MACS2 narrow/broad peaks are available and read enrichment in peaks must be quantified on equally subsampled BAMs.

## When NOT to use

- Input data is DNase-seq processed through the DNase-seq QC branch rather than ChIP-seq peak-based QC, where this peak-overlap FRiP step is not the defining metric
- Peaks have not yet been called — FRiP requires MACS2 narrowPeak/broadPeak files, so running it on unmapped or unfiltered BAMs alone is wrong
- BAMs are full-depth and not subsampled to 4,000,000 reads, making FRiP values incomparable across replicates with different library depths

## Inputs

- Treatment and control replicate BAM files (mapped, unique reads with mapping quality > 1)
- 4,000,000-read subsampled BAMs (id_*_4000000.bam)
- MACS2 peak files (id_peaks.narrowPeak or broadPeak for broad marks)
- hg19/hg38/mm9/mm10 dependent data (chromosome lengths, union DHS/blacklist regions from Cistrome DB) for the surrounding QC module

## Outputs

- Per-replicate FRiP score files (e.g. id_treat_rep1.frip)
- FRiP values compiled into the plain-text/JSON QC summary

## How to apply

First map reads (BWA default in ChiLin), retain uniquely mapped reads (mapping quality above 1 via samtools), and subsample each replicate BAM to 4,000,000 reads (id_*_4000000.bam) so FRiP is comparable across replicates. Call peaks with MACS2 per replicate and on merged BAMs, choosing narrow mode for transcription factors/active marks or broad mode for broad histone marks, with the configured FDR cutoff and keep_dup=1; fragment size can be estimated from a MACS2 predictd run. Then intersect each treatment and control peak file with the corresponding 4M-read subsampled BAM using bedtools to compute FRiP (output pattern id_treat_rep1.frip). Compile the FRiP values into the plain-text/JSON QC summary alongside peak counts, DHS overlap, and replicate overlap/correlation metrics. Correct application is confirmed by a reported FRiP for every treatment and control replicate in the QC summary.

## Related tools

- **ChiLin** (Pipeline that orchestrates subsampling, peak calling, and FRiP calculation (step 7) and compiles the QC summary) — https://github.com/cfce/chilin
- **MACS2** (Default peak caller producing the narrowPeak/broadPeak files that define the peak regions used for FRiP) — https://github.com/taoliu/MACS/
- **bedtools** (Intersects peak files with the 4M-read subsampled BAMs to compute the fraction of reads in peaks)
- **samtools** (Retains uniquely mapped reads (mapping quality above 1) and supports subsampling BAMs before FRiP)

## Evaluation signals

- QC summary reports a FRiP value for every treatment and control replicate (output pattern id_treat_rep1.frip is non-empty)
- FRiP is computed on subsampled BAMs of exactly 4,000,000 reads (id_*_4000000.bam), not full-depth BAMs
- Merged peak files (id_peaks.narrowPeak / broadPeak) are non-empty before FRiP intersection
- FRiP values are consistent with the configured MACS2 peak mode (narrow for TFs/active marks, broad for broad histone marks) and FDR cutoff with keep_dup=1
- All metrics (peak counts, FRiP, DHS overlap, replicate overlap/correlation) co-occur in a single plain-text/JSON QC summary

## Limitations

- FRiP depends on peak-calling mode and FDR cutoff, so values are not comparable unless MACS2 settings (narrow vs broad, keep_dup=1) are fixed
- ChiLin version is not stated and reproducibility of results depends on environment; the project requires python=2 with pinned tool versions (bwa=0.7.13, samtools=0.1.19, bedtools=2.17.0, macs2=2.1.0)
- The pipeline is genome-dependent: hg19/hg38/mm9/mm10 dependent data must be downloaded from cistrome.org, so FRiP QC on other assemblies needs additional reference files

## Evidence

- [methods] FRiP step in ChiLin workflow: "step 7: calculate FRiP scores for each sample."
- [methods] Peak calling feeding FRiP: "step 5: call peak for replicates samples, and do replicates peak overlap/correlation analysis"
- [readme] Pinned tool versions used by the pipeline: "conda install bwa=0.7.13 samtools=0.1.19 bedtools=2.17.0 seqtk ucsc-bedclip ucsc-bedgraphtobigwig ucsc-wigcorrelate ucsc-wigtobigwig fastqc numpy macs2=2.1.0"
- [intro] MACS2 as the default peak caller: "ChiLin uses MACS2 (https://github.com/taoliu/MACS/) as the default peak caller"
