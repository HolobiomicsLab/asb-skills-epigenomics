---
name: cross-assay-chromatin-accessibility-comparison
description: Use when two or more open-chromatin peak files (BED-derived GRanges from
  ATAC-seq, DNase-seq, or similar) must be benchmarked against a gold-standard reference
  peak set to quantify assay-level differences in peak overlap.
when_to_use_negative:
- Input is raw sequencing data (FASTQ) rather than processed peak interval files —
  EpiCompare works on peak files, not alignment or peak calling
- There is no reference/gold-standard peak set to benchmark against, since precision
  and recall are defined relative to the reference
- The goal is general QC of a single sample (blacklist fraction, duplication rate)
  rather than cross-assay comparison
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: EpiCompare
  role: 'Primary R package: peak-file handling, blacklist filtering, peak-overlap
    benchmarking, precision-recall plotting, ChromHMM annotation'
  repo: https://github.com/neurogenomics/EpiCompare
- name: R
  role: Runtime environment in which the threshold sweep and metric computation are
    scripted
- name: Bioconductor
  role: Distribution channel for EpiCompare (>= v3.15) and GRanges-based genomic interval
    infrastructure
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad049
    title: 'EpiCompare: R package for the comparison and quality control of epigenomic
      peak files'
  source_content_hash: sha256:9b3062bdde24de8cf86b6786223150df26060e91a68bb9b3e702eb1e7e7e2ce5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/neurogenomics/EpiCompare
  tools:
  - Bioconductor
  - EpiCompare
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cross-assay-chromatin-accessibility-comparison

## Summary

Benchmarks chromatin accessibility peak sets from different assays (e.g., ATAC-seq vs DNase-seq in K562) against a reference peak set by sweeping overlap thresholds and computing precision, recall, and F1, exposing assay-specific differences such as those seen in ChromHMM annotations and precision-recall plots (Fig. 1d–e).

## When to use

Use when two or more open-chromatin peak files (BED-derived GRanges from ATAC-seq, DNase-seq, or similar) must be benchmarked against a gold-standard reference peak set to quantify assay-level differences in peak overlap.

## When NOT to use

- Input is raw sequencing data (FASTQ) rather than processed peak interval files — EpiCompare works on peak files, not alignment or peak calling
- There is no reference/gold-standard peak set to benchmark against, since precision and recall are defined relative to the reference
- The goal is general QC of a single sample (blacklist fraction, duplication rate) rather than cross-assay comparison

## Inputs

- Query peak files as GRanges or BED paths (e.g., K562 ATAC-seq peaks, K562 DNase-seq peaks)
- Reference/gold-standard peak set (e.g., ENCODE DNase-seq ENCFF185XRG) as GRanges or BED
- Named list of peakfiles and named list of reference (EpiCompare 'peakfiles'/'reference' parameters)
- Genome build specification (e.g., list(peakfiles='hg19', reference='hg38'))

## Outputs

- precision_recall_curves.png (PR curves for ATAC-seq vs DNase-seq on one plot)
- pr_metrics.csv (threshold-grid precision, recall, F1 values)
- Optional interactive HTML EpiCompare report including precision-recall plots and ChromHMM annotations

## How to apply

Load the query peak files (K562 ATAC-seq and K562 DNase-seq) and the DNase-seq reference (ENCFF185XRG) as GRanges objects via EpiCompare's peak-file handling; EpiCompare removes blacklisted-region and non-standard-chromosome peaks before analysis. Sweep a grid of overlap thresholds, and at each threshold intersect each query peak file with the reference using EpiCompare's peak-overlap benchmarking routine. Compute precision (fraction of query peaks overlapping the reference), recall (fraction of reference peaks recovered), and F1 (harmonic mean) at every threshold. Draw precision-recall curves for both assays on a single plot to reproduce the ATAC-vs-DNase distinction of Fig. 1e, and inspect ChromHMM annotations (Fig. 1d) for corroborating functional differences.

## Related tools

- **EpiCompare** (Primary R package: peak-file handling, blacklist filtering, peak-overlap benchmarking, precision-recall plotting, ChromHMM annotation) — https://github.com/neurogenomics/EpiCompare
- **R** (Runtime environment in which the threshold sweep and metric computation are scripted)
- **Bioconductor** (Distribution channel for EpiCompare (>= v3.15) and GRanges-based genomic interval infrastructure)

## Examples

```
EpiCompare::EpiCompare(peakfiles = list(ATAC=K562_ATAC, DNase=K562_DNase), reference = list(ENCFF185XRG = dnase_ref), genome_build = list(peakfiles="hg19", reference="hg38"), output_dir = tempdir())
```

## Evaluation signals

- pr_metrics.csv contains a row per overlap threshold with monotonically changing precision/recall values across the sweep
- Precision-recall plot renders both ATAC-seq and DNase-seq curves on a single axes, reproducing the assay separation of Fig. 1e
- Query and reference peak sets have consistent genome builds after EpiCompare lift/liftover handling (no mixed hg19/hg38 intervals)
- Blacklisted-region and non-standard-chromosome peaks are absent from the analyzed intervals
- F1 equals the harmonic mean of the reported precision and recall at every threshold

## Limitations

- Peaks in blacklisted regions and on non-standard chromosomes are removed prior to analysis, so metrics are conditional on this filtering
- Older EpiCompare versions may have different functions/parameters; the README notes version-dependent documentation differences
- Results depend on the choice of reference peak set and the grid of overlap thresholds swept

## Evidence

- [abstract] abstract: "EpiCompare combines a variety of downstream analysis tools to compare, quality control and benchmark different epigenomic datasets."
- [results] results: [evidence span withheld — anchor status: external_doc]
- [readme] readme_filtering: [evidence span withheld — anchor status: external_doc]
- [intro] intro_benchmarking: "To better understand the performance of these novel approaches, it is important that we systematically compare these technologies and benchmark against a "gold-standard"."
