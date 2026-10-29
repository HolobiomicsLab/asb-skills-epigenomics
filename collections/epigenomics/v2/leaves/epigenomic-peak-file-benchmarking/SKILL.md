---
name: epigenomic-peak-file-benchmarking
description: Use when comparing peak files from different epigenomic technologies
  (ATAC-seq, DNase-seq, ChIP-seq, CUT&Run, CUT&Tag) against a gold-standard reference
  to quantify overlap, precision, recall, and annotation differences.
when_to_use_negative:
- Input is raw sequencing data (FASTQ) rather than called peak files — EpiCompare
  operates downstream on peak intervals, not reads
- Only one peak file is available with no reference or comparison group, since the
  skill's purpose is contrasting datasets or benchmarking against a gold-standard
- Input files are already processed feature tables or summary statistics rather than
  genomic interval (BED/narrowPeak) peak sets
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3170
tools:
- name: EpiCompare
  role: 'Primary R package: peak-file handling, overlap benchmarking, precision-recall
    plotting, consensus peak calling (compute_consensus_peaks), and HTML report generation'
  repo: https://github.com/neurogenomics/EpiCompare
- name: R
  role: Execution environment for running EpiCompare and Bioconductor genomic interval
    workflows
- name: Bioconductor
  role: Distribution platform for EpiCompare (requires >= v3.15) and underlying GRanges
    interval infrastructure
  repo: https://bioconductor.org/packages/release/bioc/html/EpiCompare.html
- name: ChIPseeker
  role: Functional annotation of peaks within the EpiCompare report (peak annotation
    and TSS enrichment)
- name: EpiArchives
  role: Public archive of example EpiCompare HTML reports (e.g. atac_dnase_example)
    with reproducible code
  repo: https://github.com/neurogenomics/EpiArchives
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad049
    title: 'EpiCompare: R package for the comparison and quality control of epigenomic
      peak files'
  source_content_hash: sha256:d934a1addec601f7e1c217f17d5134480cad79115ae923758cf0c4b28f26e188
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neurogenomics/EpiCompare
  tools:
  - Bioconductor
  - ChIPseeker
  - EpiArchives
  - EpiCompare
  - R
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# epigenomic-peak-file-benchmarking

## Summary

Benchmark epigenomic peak files (e.g. ATAC-seq vs DNase-seq, CUT&Tag vs ChIP-seq) using EpiCompare, which computes peak overlaps, precision-recall metrics, and functional annotations against a reference dataset in a single HTML report.

## When to use

Use when comparing peak files from different epigenomic technologies (ATAC-seq, DNase-seq, ChIP-seq, CUT&Run, CUT&Tag) against a gold-standard reference to quantify overlap, precision, recall, and annotation differences.

## When NOT to use

- Input is raw sequencing data (FASTQ) rather than called peak files — EpiCompare operates downstream on peak intervals, not reads
- Only one peak file is available with no reference or comparison group, since the skill's purpose is contrasting datasets or benchmarking against a gold-standard
- Input files are already processed feature tables or summary statistics rather than genomic interval (BED/narrowPeak) peak sets

## Inputs

- Query peak files as GRanges objects or BED file paths (e.g. K562 ATAC-seq and DNase-seq peaks)
- Reference peak set (e.g. ENCODE DNase-seq ENCFF185XRG or encode_H3K27ac) as GRanges or named list
- Optional Picard summary txt files for duplication-rate metrics (e.g. CnT_H3K27ac_picard)
- Genome build specification per file (hg19/hg38) and ENCODE blacklist regions

## Outputs

- Single interactive HTML report with General Metrics, Peak Overlap, and Functional Annotation sections
- precision_recall_curves.png comparing methods (Fig. 1e style)
- pr_metrics.csv with threshold-grid precision, recall, and F1 values
- Consensus GRanges/BED peak sets from compute_consensus_peaks (granges and consensusseeker methods)
- ChromHMM and ChIPseeker functional annotation plots (Fig. 1d style)

## How to apply

Load query peak files (e.g. K562 ATAC-seq and DNase-seq BED files) and a reference set (e.g. ENCODE DNase-seq ENCFF185XRG) as GRanges objects, optionally with EpiCompare::gather_files, and remove blacklisted (hg19/hg38) and non-standard chromosome peaks prior to analysis. Sweep a grid of overlap thresholds, intersecting each query with the reference at each threshold to compute precision (fraction of query peaks overlapping the reference), recall (fraction of reference peaks recovered), and F1. Plot precision-recall curves for all methods on one figure to reproduce the method distinction shown in Fig. 1e, and complement it with ChromHMM annotations (Fig. 1d), Upset plots, and correlation plots from the EpiCompare report. For consensus peak calling across replicate peak-file groups, use compute_consensus_peaks with method='granges' (fast interval overlap) or method='consensusseeker' (slower, models peak score distributions). Run the full analysis with a single EpiCompare() call (run_all=TRUE), supplying genome_build per file (e.g. peakfiles='hg19', reference='hg38') and an output_dir for the HTML report.

## Related tools

- **EpiCompare** (Primary R package: peak-file handling, overlap benchmarking, precision-recall plotting, consensus peak calling (compute_consensus_peaks), and HTML report generation) — https://github.com/neurogenomics/EpiCompare
- **R** (Execution environment for running EpiCompare and Bioconductor genomic interval workflows)
- **Bioconductor** (Distribution platform for EpiCompare (requires >= v3.15) and underlying GRanges interval infrastructure) — https://bioconductor.org/packages/release/bioc/html/EpiCompare.html
- **ChIPseeker** (Functional annotation of peaks within the EpiCompare report (peak annotation and TSS enrichment))
- **EpiArchives** (Public archive of example EpiCompare HTML reports (e.g. atac_dnase_example) with reproducible code) — https://github.com/neurogenomics/EpiArchives

## Examples

```
EpiCompare::EpiCompare(peakfiles=list(ATAC=atac_peaks, DNase=dnase_peaks), reference=list(DNase_ref=ENCFF185XRG), genome_build=list(peakfiles="hg19", reference="hg38"), run_all=TRUE, output_dir=tempdir())
```

## Evaluation signals

- precision-recall curves for ATAC-seq and DNase-seq render on a single plot and visibly distinguish the methods, reproducing Fig. 1e
- pr_metrics.csv parses with one row per (method, threshold) and precision/recall in [0,1], F1 equal to the harmonic mean of precision and recall
- Blacklisted-region and non-standard-chromosome peaks are removed before any overlap computation (report General Metrics reports the removed percentage)
- Both consensus outputs (method='granges' and method='consensusseeker') exist, parse as valid GRanges/BED, and contain a non-empty peak set covering the CnT and CnR H3K27ac intervals
- The final EpiCompare() run produces a single interactive HTML report in output_dir covering General Metrics, Peak Overlap, and Functional Annotation sections

## Limitations

- The two consensus strategies (granges vs consensusseeker) are benchmarked only descriptively in the paper — no numeric acceptance criterion is reported
- EpiCompare removes peaks in blacklisted regions and on non-standard chromosomes prior to analysis, so results differ from raw overlap counts
- Peak files with mismatched genome builds must be reconciled via genome_build_output, or overlaps will be invalid

## Evidence

- [results] Applying EpiCompare to the K562 open-chromatin datasets reveals a difference between ATAC-seq and DNase-seq, most notably in the ChromHMM annotations and the precision-recall plot: [evidence span withheld — anchor status: not_found]
- [abstract] EpiCompare combines downstream tools to compare, QC and benchmark epigenomic datasets: "EpiCompare combines a variety of downstream analysis tools to compare, quality control and benchmark different epigenomic datasets."
- [readme] Peaks in blacklisted regions and non-standard chromosomes removed prior to analysis: [evidence span withheld — anchor status: external_doc]
- [full_text] EpiCompare offers multiple methods for calling consensus peaks from groups of peak files via compute_consensus_peaks: [evidence span withheld — anchor status: not_found]
- [readme] Peak Overlap section includes Upset, precision-recall and correlation plots: [evidence span withheld — anchor status: external_doc]
