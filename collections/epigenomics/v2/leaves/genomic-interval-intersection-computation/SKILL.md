---
name: genomic-interval-intersection-computation
description: Use when scoring ChIP-seq peak calls against ENCODE union DHS regions
  to compute per-sample overlap ratios, or when grouping overlap ratios by histone
  mark / factor category to build a quality atlas distribution.
when_to_use_negative:
- The input regions are not called peaks against a DHS reference — e.g., raw FASTQ/BAM
  processing (use the mapping and peak-calling ChiLin steps first)
- You need generic interval arithmetic between two experimental BED sets, not overlap-with-open-chromatin
  QC benchmarking against the union DHS reference
- Peak counts are too small to form a meaningful per-sample or per-mark distribution
  (e.g., the pipeline convention of skipping optional steps when peaks number less
  than 200)
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_3308
tools:
- name: bedtools
  role: Intersects each sample's narrowPeak file against the union DHS BED and computes
    the per-sample overlap fraction
- name: ChiLin
  role: Wraps the DHS-overlap QC step, aggregates per-replicate *.enrich.dhs ratios
    into the id_dhs. QC JSON, and references the union DHS dependent data
  repo: https://github.com/cfce/chilin
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-016-1274-4
    title: 'ChiLin: a comprehensive ChIP-seq and DNase-seq quality control and analysis
      pipeline'
  source_content_hash: sha256:7d59f025e62b93edfa9fc90580b66a4d4411cd4005ac9cbbc3c70a96ae55605a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfce/chilin
  tools:
  - ChiLin
  - Cistrome
  - bedtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# Genomic interval intersection computation

## Summary

Computes the fraction of called ChIP-seq peaks (id_sort_peaks.narrowPeak) that fall inside a reference union DHS BED file using bedtools intersect, producing the per-sample union-DHS overlap ratio used as a QC metric in the ChiLin quality atlas.

## When to use

Use when scoring ChIP-seq peak calls against ENCODE union DHS regions to compute per-sample overlap ratios, or when grouping overlap ratios by histone mark / factor category to build a quality atlas distribution.

## When NOT to use

- The input regions are not called peaks against a DHS reference — e.g., raw FASTQ/BAM processing (use the mapping and peak-calling ChiLin steps first)
- You need generic interval arithmetic between two experimental BED sets, not overlap-with-open-chromatin QC benchmarking against the union DHS reference
- Peak counts are too small to form a meaningful per-sample or per-mark distribution (e.g., the pipeline convention of skipping optional steps when peaks number less than 200)

## Inputs

- Per-sample called peaks in narrowPeak format (id_sort_peaks.narrowPeak)
- Union DHS regions BED file from the ChiLin dependent data (dhs entry, Cistrome DB source)
- Species reference dependent data package (hg19/hg38/mm9/mm10 tgz from cistrome.org)

## Outputs

- Per-replicate overlap intermediate files (*.enrich.dhs)
- Per-sample QC JSON (id_dhs.) with a union-dhs-overlap field for every sample
- Per-mark DHS overlap distribution table (dhs_overlap_summary.tsv)

## How to apply

Run bedtools intersect between each sample's sorted peak file (id_sort_peaks.narrowPeak) and the union DHS regions BED packaged in the ChiLin dependent data (the dhs entry sourced from the Cistrome DB), counting peaks that overlap a DHS interval to compute the fraction of peaks in DHS. Write per-replicate intermediate *.enrich.dhs files, then aggregate per-sample and per-replicate ratios into the QC JSON file id_dhs. following the ChiLin QC statistics convention under the union-dhs-overlap field. Finally group samples by histone mark or factor and summarize per-mark distributions (e.g., dhs_overlap_summary.tsv). The rationale is that open-chromatin overlap is a positive QC signal: ChIP-seq peaks for real factors should concentrate in DHS-accessible regions, so a low ratio flags a poor experiment.

## Related tools

- **bedtools** (Intersects each sample's narrowPeak file against the union DHS BED and computes the per-sample overlap fraction)
- **ChiLin** (Wraps the DHS-overlap QC step, aggregates per-replicate *.enrich.dhs ratios into the id_dhs. QC JSON, and references the union DHS dependent data) — https://github.com/cfce/chilin

## Examples

```
bedtools intersect -a id_sort_peaks.narrowPeak -b union_dhs.bed -wa -c | awk '{n++; s+=$NF} END{print s/n}' > sample.enrich.dhs
```

## Evaluation signals

- Every sample has a valid numeric union-dhs-overlap field in id_dhs. and each ratio lies in [0, 1]
- Per-replicate *.enrich.dhs intermediate files exist for every replicate processed
- Per-mark medians in dhs_overlap_summary.tsv reproduce the expected ordering: acetylation marks (H3K27ac, H3K122ac, H3K18ac, H4K8ac, H4K91ac, H4K16ac, H4K12ac) above 0.75 with tight distributions, H3T11ph high, H3K27me3/H3K36me3 intermediate, H3K9me3 lowest
- The union DHS BED used matches the correct genome build dependent data package (hg19/hg38/mm9/mm10) for the samples being scored

## Limitations

- Requires species-specific dependent data (wget -c http://cistrome.org/chilin/_downloads/hg19.tgz, hg38.tgz, mm9.tgz, mm10.tgz); wrong build invalidates overlap ratios
- ChiLin environment is pinned and dated: python 2, bwa=0.7.13, samtools=0.1.19, bedtools=2.17.0, macs2=2.1.0; other versions may change intersection behavior
- The union DHS reference derives from ENCODE/Cistrome DB, so ratios are benchmark-specific; they are comparable across samples only when scored against the same reference

## Evidence

- [readme] chiLin depends on bedtools 2.17.0: "conda install bwa=0.7.13 samtools=0.1.19 bedtools=2.17.0 seqtk ucsc-bedclip ucsc-bedgraphtobigwig ucsc-wigcorrelate ucsc-wigtobigwig fastqc numpy macs2=2.1.0"
- [intro] ChiLin QC includes DHS overlap of peaks: "ChiLin, an integrated command line quality control and analysis pipeline for ChIP-seq or DNase-seq data"
- [methods] Peaks are required for downstream QC/annotation: "step 6: call peak for merged bam file, this step cannot be skipped, because this provided peak for annotation step"
- [methods] Genome dependent data are downloaded from Cistrome: "wget -c http://cistrome.org/chilin/_downloads/hg19.tgz"
- [abstract] QC reports compare against historical data atlas: "ChiLin generates comprehensive quality control reports that include comparisons with historical data"
