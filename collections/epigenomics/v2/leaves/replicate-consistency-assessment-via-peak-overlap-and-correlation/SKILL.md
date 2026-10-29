---
name: replicate-consistency-assessment-via-peak-overlap-and-correlation
description: Use when QC-ing a ChIP-seq experiment with ≥2 biological replicates and
  you need to measure agreement before merging BAMs or trusting merged peak calls.
when_to_use_negative:
- Single-replicate ChIP-seq experiments — there is no second replicate to correlate
  or overlap against
- QC layers that only need library-level metrics (PBC, FastQC GC content, FRiP per
  sample) without inter-replicate agreement
- DNase-seq-only pipelines where MACS2 peak-based replicate overlap semantics do not
  apply as configured
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0091
tools:
- name: ChiLin
  role: Pipeline that orchestrates replicate peak calling and runs the overlap/correlation
    analysis as its step 5
  repo: https://github.com/cfce/chilin
- name: MACS2
  role: Default peak caller used per replicate before overlap computation, after predictd
    fragment-size estimation
  repo: https://github.com/taoliu/MACS
- name: wigCorrelate
  role: UCSC tool computing Pearson correlation across genome-normalized treatment
    bigWigs to yield id.cor
  repo: https://github.com/cfce/chilin
- name: bedClip
  role: Clips MACS2 bedgraphs to chromosome bounds before bigWig conversion for correlation
- name: bedGraphToBigWig
  role: Converts clipped replicate bedgraphs to bigWigs consumed by wigCorrelate
- name: bedtools
  role: Intersects replicate peak files to compute the percentage of overlapping peaks
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-016-1274-4
    title: 'ChiLin: a comprehensive ChIP-seq and DNase-seq quality control and analysis
      pipeline'
  source_content_hash: sha256:f34518b065eb5292a86324cfec07b2fa4145223eca533edca995a64260f1ec63
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Replicate consistency assessment via peak overlap and correlation

## Summary

ChiLin's ChIP-layer QC step (step 5) that quantifies ChIP-seq replicate consistency using two complementary metrics: Pearson correlation of genome-wide normalized signal via UCSC wigCorrelate, and the inter-replicate percentage of overlapping MACS2-called peaks.

## When to use

Use when QC-ing a ChIP-seq experiment with ≥2 biological replicates and you need to measure agreement before merging BAMs or trusting merged peak calls.

## When NOT to use

- Single-replicate ChIP-seq experiments — there is no second replicate to correlate or overlap against
- QC layers that only need library-level metrics (PBC, FastQC GC content, FRiP per sample) without inter-replicate agreement
- DNase-seq-only pipelines where MACS2 peak-based replicate overlap semantics do not apply as configured

## Inputs

- Treatment and control replicate FASTQ files (e.g. foxa1_t1.fastq/foxa1_c1.fastq demo pair)
- BAM files per replicate after BWA mapping and samtools unique-read filtering (MAPQ > 1)
- 4M-read subsampled BAMs (id_*_4000000.bam)
- Chromosome length file and union DHS/blacklist (velcro) regions from Cistrome DB for hg19/hg38/mm9/mm10
- MACS2 fragment-size estimate from a predictd run

## Outputs

- Per-replicate peak files (id_peaks.narrowPeak or broadPeak)
- Inter-replicate peak overlap file (id_0_1.overlap)
- Replicate correlation file (id.cor) from wigCorrelate on treatment bigWigs
- Replicate bigWig signal tracks (bedClip + bedGraphToBigWig from MACS2 bedgraphs)
- Entries in the plain-text/JSON QC summary reporting overlap/correlation values

## How to apply

Sub-sample each replicate BAM to 4,000,000 unique reads (MAPQ > 1 retained after BWA mapping with samtools) so replicates are compared at equal depth. Estimate fragment size with a MACS2 predictd run, then call peaks per replicate with MACS2 (narrow mode for TFs/active marks, broad mode for broad histone marks) using the configured FDR cutoff and keep_dup=1. Convert each replicate's MACS2 bedgraph to bigWig with bedClip and bedGraphToBigWig, then run UCSC wigCorrelate on the treatment bigWigs to obtain the Pearson correlation of reads-per-million-normalized signal. Compute the inter-replicate peak-overlap ratio from the per-replicate peak files, producing outputs like id_0_1.overlap and id.cor. Rationale: equal-depth subsampling and RPM normalization prevent depth differences from inflating correlation, while peak overlap captures agreement in discrete binding events.

## Related tools

- **ChiLin** (Pipeline that orchestrates replicate peak calling and runs the overlap/correlation analysis as its step 5) — https://github.com/cfce/chilin
- **MACS2** (Default peak caller used per replicate before overlap computation, after predictd fragment-size estimation) — https://github.com/taoliu/MACS
- **wigCorrelate** (UCSC tool computing Pearson correlation across genome-normalized treatment bigWigs to yield id.cor) — https://github.com/cfce/chilin
- **bedClip** (Clips MACS2 bedgraphs to chromosome bounds before bigWig conversion for correlation)
- **bedGraphToBigWig** (Converts clipped replicate bedgraphs to bigWigs consumed by wigCorrelate)
- **bedtools** (Intersects replicate peak files to compute the percentage of overlapping peaks)

## Examples

```
wigCorrelate id_treat_rep1.bw id_treat_rep2.bw > id.cor
```

## Evaluation signals

- QC summary contains a non-empty id.cor correlation value for the replicate pair
- Inter-replicate overlap output id_0_1.overlap exists and reports a non-zero overlap ratio
- Non-empty id_peaks.narrowPeak (or broadPeak) files exist for every replicate before overlap is computed
- Correlation was computed on RPM-normalized treatment bigWigs derived from equal-depth (4,000,000-read) subsampled BAMs, not raw BAMs
- Output file names match ChiLin's documented patterns (id.cor, id_0_1.overlap)

## Limitations

- No software version stated for ChiLin, complicating exact reproduction of correlation/overlap values
- Replicate peak calling requires a valid MACS2 fragment-size estimate; a poor predictd run degrades peak quality and thus both consistency metrics
- Correlation of RPM-normalized genome-wide signal can be dominated by depth artifacts if the 4M-read subsampling step is skipped

## Evidence

- [methods] step 5: call peak for replicates samples, and do replicates peak overlap/correlation analysis: "step 5: call peak for replicates samples, and do replicates peak overlap/correlation analysis"
- [readme] conda install: "conda install bwa=0.7.13 samtools=0.1.19 bedtools=2.17.0 seqtk ucsc-bedclip ucsc-bedgraphtobigwig ucsc-wigcorrelate ucsc-wigtobigwig fastqc numpy macs2=2.1.0 bioconductor-seqlogo"
- [abstract] abstract ChiLin automates: "ChiLin is a computational pipeline that automates the quality control and data analyses of ChIP-seq and DNase-seq data."
- [methods] FRiP: "step 7: calculate FRiP scores for each sample."
