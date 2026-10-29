---
name: qc-aggregation
description: Use when per-sample peak calls (narrowPeak) and replicate-level QC intermediates
  must be rolled up into a ChiLin QC JSON and per-mark distributions (e.g., DHS overlap
  ratios grouped by histone mark type) for atlas-scale comparison.
when_to_use_negative:
- Input samples are not ChIP-seq/DNase-seq peak sets (e.g., RNA-seq counts or ATAC-seq
  without an appropriate union accessibility reference), since the DHS overlap ratio
  is calibrated against ENCODE union DHS regions
- You only need an upstream QC metric already produced by ChiLin (FastQC, PBC, FRiP);
  use the specific metric step instead of the DHS-overlap aggregation workflow
- Peak calls are merged across replicates before QC, so no per-replicate *.enrich.dhs
  intermediates can be produced and replicate-level aggregation is meaningless
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: ChiLin
  role: Pipeline whose QC statistics convention the id_dhs JSON and aggregation follow;
    installs the dependent data packaging the union DHS regions
  repo: https://github.com/cfce/chilin
- name: bedtools
  role: Intersects each sample's sorted narrowPeak against the union DHS BED to compute
    the per-sample overlap ratio
- name: Cistrome
  role: Source of the union DHS regions and the Cistrome Data Browser context for
    the quality atlas
- name: MACS2
  role: Default peak caller producing the narrowPeak files fed into the DHS overlap
    aggregation
  repo: https://github.com/taoliu/MACS
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-016-1274-4
    title: 'ChiLin: a comprehensive ChIP-seq and DNase-seq quality control and analysis
      pipeline'
  source_content_hash: sha256:c4f4f2134c594da7270e514ea9ebde722ce785d5ba839740d64718b397cb0ffa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfce/chilin
  tools:
  - ChiLin
  - Cistrome
  - MACS2
  - bedtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# qc--aggregation

## Summary

Aggregates per-sample and per-replicate ChiLin QC statistics — most notably the union DHS overlap ratio computed by intersecting id_sort_peaks.narrowPeak against the ENCODE union DHS BED — into a QC JSON (id_dhs) and per-histone-mark distribution summaries, enabling comparison against the ChiLin quality atlas.

## When to use

Use when per-sample peak calls (narrowPeak) and replicate-level QC intermediates must be rolled up into a ChiLin QC JSON and per-mark distributions (e.g., DHS overlap ratios grouped by histone mark type) for atlas-scale comparison.

## When NOT to use

- Input samples are not ChIP-seq/DNase-seq peak sets (e.g., RNA-seq counts or ATAC-seq without an appropriate union accessibility reference), since the DHS overlap ratio is calibrated against ENCODE union DHS regions
- You only need an upstream QC metric already produced by ChiLin (FastQC, PBC, FRiP); use the specific metric step instead of the DHS-overlap aggregation workflow
- Peak calls are merged across replicates before QC, so no per-replicate *.enrich.dhs intermediates can be produced and replicate-level aggregation is meaningless

## Inputs

- Per-sample peak call files (id_sort_peaks.narrowPeak) for atlas samples
- Union DHS regions BED file from ChiLin hg19/hg38 dependent data (dhs entry, Cistrome DB)
- Histone mark / factor category labels for sample grouping

## Outputs

- Per-replicate *.enrich.dhs intermediate files (per-sample DHS overlap fractions)
- QC JSON file id_dhs with a valid union-dhs-overlap field for every sample
- Per-mark DHS overlap distribution table dhs_overlap_summary.tsv (per-mark medians)

## How to apply

Intersect each sample's id_sort_peaks.narrowPeak against the union DHS regions BED from the ChiLin dependent data (dhs entry, sourced from Cistrome DB) with bedtools to compute the fraction of peaks inside DHS, writing per-replicate *.enrich.dhs intermediates. Aggregate these per-sample/per-replicate ratios into the QC JSON id_dhs following the ChiLin QC statistics convention, then group samples by histone mark/factor category and compute per-mark DHS overlap distributions. Validate against the expected Figure 5 ordering: acetylation marks (H3K27ac, H3K122ac, H3K18ac, H4K8ac, H4K91ac, H4K16ac, H4K12ac) show a tight distribution above 0.75, H3T11ph high, H3K27me3/H3K36me3 intermediate, and H3K9me3 lowest. This mirrors ChiLin's batch-oriented design, where replicate overlap/correlation and FRiP metrics are computed per sample then compiled into the QC database.

## Related tools

- **ChiLin** (Pipeline whose QC statistics convention the id_dhs JSON and aggregation follow; installs the dependent data packaging the union DHS regions) — https://github.com/cfce/chilin
- **bedtools** (Intersects each sample's sorted narrowPeak against the union DHS BED to compute the per-sample overlap ratio)
- **Cistrome** (Source of the union DHS regions and the Cistrome Data Browser context for the quality atlas)
- **MACS2** (Default peak caller producing the narrowPeak files fed into the DHS overlap aggregation) — https://github.com/taoliu/MACS

## Examples

```
bedtools intersect -a id_sort_peaks.narrowPeak -b union_DHS.bed -u | wc -l
```

## Evaluation signals

- id_dhs QC JSON exists and contains a valid union-dhs-overlap field for every sample
- Per-mark medians in dhs_overlap_summary.tsv reproduce the Fig. 5 ordering: acetylation marks > 0.75, H3K9me3 lowest
- Acetylation marks H3K27ac, H3K122ac, H3K18ac, H4K8ac, H4K91ac, H4K16ac, H4K12ac show a tight distribution of overlap > 0.75
- Every per-replicate *.enrich.dhs intermediate is present for samples with replicate peak calls
- DHS overlap ratios fall in the valid range [0, 1] for all samples

## Limitations

- ChiLin is an extensible suite without a stated software version; reproducibility is claimed but concrete environment specifications and output artifacts are not provided in all sections
- Union DHS overlap depends on the organism-specific dependent data packages (hg19/hg38/mm9/mm10 tgz downloads), so mixing genome builds invalidates cross-sample comparison
- Per-mark patterns are historical atlas references from public GEO samples (23,677 profiles); datasets from other sources may not follow the expected acetylation/non-acetylation ordering

## Evidence

- [abstract] ChiLin is a computational pipeline that automates the quality control and data analyses of ChIP-seq and DNase-seq data.: "ChiLin is a computational pipeline that automates the quality control and data analyses of ChIP-seq and DNase-seq data."
- [methods] step 5: call peak for replicates samples, and do replicates peak overlap/correlation analysis: "step 5: call peak for replicates samples, and do replicates peak overlap/correlation analysis"
- [methods] step 7: calculate FRiP scores for each sample.: "step 7: calculate FRiP scores for each sample."
- [readme] conda install bwa=0.7.13 samtools=0.1.19 bedtools=2.17.0 seqtk ucsc-bedclip ucsc-bedgraphtobigwig ucsc-wigcorrelate ucsc-wigtobigwig fastqc numpy macs2=2.1.0: "conda install bwa=0.7.13 samtools=0.1.19 bedtools=2.17.0 seqtk ucsc-bedclip ucsc-bedgraphtobigwig ucsc-wigcorrelate ucsc-wigtobigwig fastqc numpy macs2=2.1.0"
- [abstract] We have successfully compiled 23,677 profiles into a comprehensive quality atlas with fine classification for users.: "We have successfully compiled 23,677 profiles into a comprehensive quality atlas with fine classification for users."
