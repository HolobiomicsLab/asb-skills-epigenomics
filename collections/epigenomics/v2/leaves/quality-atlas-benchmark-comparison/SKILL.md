---
name: quality-atlas-benchmark-comparison
description: Use when ChIP-seq narrowPeak samples (e.g. from a histone mark atlas)
  need a QC metric scored against ENCODE union DHS regions, or when reproducing Fig.
  5 per-mark DHS overlap distributions.
when_to_use_negative:
- Input peaks are from a non-ChIP-seq/DNase-seq assay (e.g. RNA-seq) or are unsorted/un-called
  reads rather than narrowPeak files, since the ratio is defined over called peaks.
- No genome-matched union DHS BED reference is available for the sample's assembly
  (hg19/hg38/mm9/mm10 dependent data), so the overlap ratio would be meaningless.
- The goal is general ChIP-seq QC (FRiP, PBC, NSC/RSC) rather than the DHS-overlap
  atlas benchmark specifically; ChiLin's other QC steps cover those metrics.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: ChiLin
  role: pipeline that produces the narrowPeak inputs and aggregates per-sample/per-replicate
    DHS overlap ratios into the QC JSON (id_dhs) using its QC statistics convention
  repo: https://github.com/cfce/chilin
- name: bedtools
  role: intersects each sample's sorted peaks against the union DHS regions BED to
    compute the fraction of peaks in DHS
- name: Cistrome
  role: source of the union DHS reference data packaged in the ChiLin dependent data
  repo: https://bitbucket.org/cistrome/cistrome-applications-harvard
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-016-1274-4
    title: 'ChiLin: a comprehensive ChIP-seq and DNase-seq quality control and analysis
      pipeline'
  source_content_hash: sha256:cd95c40813d64ed919d9dbe69a4ffd81486f3e413c43ae3ec6da78149d274bbd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfce/chilin
  tools:
  - ChiLin
  - Cistrome
  - bedtools
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# quality-atlas-benchmark-comparison

## Summary

Computes per-sample union DNase hypersensitive site (DHS) overlap ratios from ChiLin peak calls and benchmarks them against the ChiLin quality atlas distributions across histone mark categories, verifying that acetylation marks show the expected high overlap pattern.

## When to use

Use when ChIP-seq narrowPeak samples (e.g. from a histone mark atlas) need a QC metric scored against ENCODE union DHS regions, or when reproducing Fig. 5 per-mark DHS overlap distributions.

## When NOT to use

- Input peaks are from a non-ChIP-seq/DNase-seq assay (e.g. RNA-seq) or are unsorted/un-called reads rather than narrowPeak files, since the ratio is defined over called peaks.
- No genome-matched union DHS BED reference is available for the sample's assembly (hg19/hg38/mm9/mm10 dependent data), so the overlap ratio would be meaningless.
- The goal is general ChIP-seq QC (FRiP, PBC, NSC/RSC) rather than the DHS-overlap atlas benchmark specifically; ChiLin's other QC steps cover those metrics.

## Inputs

- per-sample sorted peak files (id_sort_peaks.narrowPeak) from ChiLin peak calling
- union DHS regions BED file from ChiLin dependent data (Cistrome DB dhs entry)
- sample-to-histone-mark / factor category mapping
- replicate grouping for per-replicate *.enrich.dhs intermediates

## Outputs

- QC JSON file id_dhs with a valid union-dhs-overlap field per sample
- per-replicate *.enrich.dhs intermediate files
- dhs_overlap_summary.tsv with per-mark DHS overlap distributions (e.g. medians)

## How to apply

Load the per-sample sorted peak files (id_sort_peaks.narrowPeak) for the atlas samples together with the union DHS regions BED file packaged in the ChiLin dependent data (the dhs entry sourced from the Cistrome DB). Intersect each sample's peaks against the union DHS regions with bedtools to compute the per-sample overlap ratio (fraction of peaks falling in DHS), writing per-replicate *.enrich.dhs files as intermediates. Aggregate the per-sample and per-replicate ratios into the QC JSON id_dhs following the ChiLin QC statistics convention, then group samples by histone mark/factor category and compute per-mark DHS overlap distributions. Validate against the reported ordering: tight high overlap (>0.75) for acetylation marks (H3K27ac, H3K122ac, H3K18ac, H4K8ac, H4K91ac, H4K16ac, H4K12ac), high for H3T11ph, intermediate for H3K27me3/H3K36me3, and lowest for H3K9me3.

## Related tools

- **ChiLin** (pipeline that produces the narrowPeak inputs and aggregates per-sample/per-replicate DHS overlap ratios into the QC JSON (id_dhs) using its QC statistics convention) — https://github.com/cfce/chilin
- **bedtools** (intersects each sample's sorted peaks against the union DHS regions BED to compute the fraction of peaks in DHS)
- **Cistrome** (source of the union DHS reference data packaged in the ChiLin dependent data) — https://bitbucket.org/cistrome/cistrome-applications-harvard

## Examples

```
bedtools intersect -a id_sort_peaks.narrowPeak -b union_dhs.bed -u | wc -l
```

## Evaluation signals

- id_dhs JSON exists and contains a valid union-dhs-overlap field for every sample.
- Per-mark medians in dhs_overlap_summary.tsv reproduce the Fig. 5 ordering: acetylation marks (H3K27ac, H3K122ac, H3K18ac, H4K8ac, H4K91ac, H4K16ac, H4K12ac) all >0.75 and H3K9me3 lowest.
- Acetylation-mark distributions are tight around >0.75 while non-acetylation marks spread across high (H3T11ph), intermediate (H3K27me3, H3K36me3), and low (H3K9me3) overlap.
- Every sample has a corresponding *.enrich.dhs per-replicate intermediate consistent with its ratio in the QC JSON.
- All overlap ratios fall in [0, 1], since they are fractions of peaks inside union DHS regions.

## Limitations

- The DHS overlap benchmark is genome-assembly dependent; the union DHS reference must match the sample's assembly (ChiLin ships hg19, hg38, mm9, mm10 dependent data via http://cistrome.org/chilin/_downloads/).
- The paper does not state software versions or provide concrete reproducibility commands for the atlas comparisons in the discussion; a reproducibility claim is stated but no environment specification or output artifacts are given there.
- Marks with few atlas samples may yield unstable per-mark distributions; peaks number thresholds matter elsewhere in ChiLin (e.g. motif analysis is skipped for peaks < 200).

## Evidence

- [other] DHS overlap pattern for acetylation marks: [evidence span withheld — anchor status: not_found]
- [other] workflow definition: [evidence span withheld — anchor status: not_found]
- [other] non-acetylation contrast: [evidence span withheld — anchor status: not_found]
- [abstract] atlas scope: "We have successfully compiled 23,677 profiles into a comprehensive quality atlas with fine classification for users."
- [methods] dependent data downloads: "wget -c http://cistrome.org/chilin/_downloads/hg19.tgz"
- [readme] bedtools version used by ChiLin install: "conda install bwa=0.7.13 samtools=0.1.19 bedtools=2.17.0 seqtk ucsc-bedclip ucsc-bedgraphtobigwig ucsc-wigcorrelate ucsc-wigtobigwig fastqc numpy macs2=2.1.0"
