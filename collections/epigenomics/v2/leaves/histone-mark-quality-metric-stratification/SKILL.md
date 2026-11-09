---
name: histone-mark-quality-metric-stratification
description: Use when you need to compute the fraction of narrowPeak peaks falling
  in ENCODE union DHS regions per ChIP-seq sample and compare overlap-ratio distributions
  across histone mark/factor categories as a QC stratification.
when_to_use_negative:
- The data are DNase-seq or the question concerns mapping, PBC, or FRiP QC rather
  than peak overlap with regulatory regions
- Peaks have not been called (no narrowPeak files available), since the ratio requires
  called peak intervals per sample
- The genome build of the peaks and the union DHS BED do not match (e.g. peaks on
  hg38 intersected against an hg19 DHS track without liftover)
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: ChiLin
  role: Pipeline whose QC statistics convention (id_dhs. JSON) is used to aggregate
    per-sample union-DHS overlap ratios; dependent data provide the union DHS BED
  repo: https://github.com/cfce/chilin
- name: bedtools
  role: Performs the peak-to-union-DHS intersection used to compute the per-sample
    overlap ratio
- name: Cistrome
  role: Source of the union DHS regions BED file packaged in the ChiLin dependent
    data (dhs entry)
- name: MACS2
  role: Default peak caller in ChiLin producing the narrowPeak peaks whose DHS overlap
    is measured
  repo: https://github.com/taoliu/MACS/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-016-1274-4
    title: 'ChiLin: a comprehensive ChIP-seq and DNase-seq quality control and analysis
      pipeline'
  source_content_hash: sha256:74e9eeadbd08745871d68237270b322cc3234f86d23234de4d8cbfdd22a17d93
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
yes_rate: 0.4444444444444444
n_claims: 9
---

# histone-mark-quality-metric-stratification

## Summary

Computes per-sample union-DHS overlap ratios for ChIP-seq peaks (via bedtools intersection against the ENCODE union DHS regions) and stratifies the resulting QC distributions by histone mark type, distinguishing acetylation marks (H3K27ac, H3K122ac, H3K18ac, H4K8ac, H4K91ac, H4K16ac, H4K12ac) from non-acetylation marks such as H3K9me3, H3K27me3, H3K36me3, and H3T11ph.

## When to use

Use when you need to compute the fraction of narrowPeak peaks falling in ENCODE union DHS regions per ChIP-seq sample and compare overlap-ratio distributions across histone mark/factor categories as a QC stratification.

## When NOT to use

- The data are DNase-seq or the question concerns mapping, PBC, or FRiP QC rather than peak overlap with regulatory regions
- Peaks have not been called (no narrowPeak files available), since the ratio requires called peak intervals per sample
- The genome build of the peaks and the union DHS BED do not match (e.g. peaks on hg38 intersected against an hg19 DHS track without liftover)

## Inputs

- id_sort_peaks.narrowPeak peak call files for each atlas sample
- union DHS regions BED file from the ChiLin dependent data (dhs entry, Cistrome DB)
- per-sample histone mark / factor category labels
- reference genome dependent data archive (e.g. hg38.tgz from cistrome.org)

## Outputs

- per-replicate *.enrich.dhs intermediate files
- id_dhs. QC JSON file with union-dhs-overlap field per sample
- dhs_overlap_summary.tsv per-mark distribution table

## How to apply

Load the per-sample peak call files (id_sort_peaks.narrowPeak) for the atlas samples together with the union DHS regions BED file from the ChiLin dependent data (dhs entry, sourced from the Cistrome DB, e.g. hg38.tgz downloaded from cistrome.org). Intersect each sample's sorted peaks against the union DHS regions with bedtools to obtain the per-sample overlap ratio (fraction of peaks in DHS), writing per-replicate *.enrich.dhs files as intermediates. Aggregate the per-sample and per-replicate ratios into the QC JSON (id_dhs.) following the ChiLin QC statistics convention. Group samples by histone mark / factor category and compute per-mark overlap distributions. Validate by checking that id_dhs. has a valid union-dhs-overlap field for every sample and that the per-mark medians reproduce the expected ordering from Fig. 5: acetylation marks tight and high (>0.75), H3T11ph high, H3K27me3/H3K36me3 intermediate, H3K9me3 lowest.

## Related tools

- **ChiLin** (Pipeline whose QC statistics convention (id_dhs. JSON) is used to aggregate per-sample union-DHS overlap ratios; dependent data provide the union DHS BED) — https://github.com/cfce/chilin
- **bedtools** (Performs the peak-to-union-DHS intersection used to compute the per-sample overlap ratio)
- **Cistrome** (Source of the union DHS regions BED file packaged in the ChiLin dependent data (dhs entry))
- **MACS2** (Default peak caller in ChiLin producing the narrowPeak peaks whose DHS overlap is measured) — https://github.com/taoliu/MACS/

## Examples

```
bedtools intersect -a id_sort_peaks.narrowPeak -b union_DHS.bed -u | wc -l; python -c "print(in_dhs/total_peaks)" > sample.enrich.dhs
```

## Evaluation signals

- id_dhs. JSON exists and contains a valid union-dhs-overlap field for every sample
- Per-mark medians in dhs_overlap_summary.tsv reproduce the Fig. 5 ordering: acetylation marks >0.75, H3T11ph high, H3K27me3/H3K36me3 intermediate, H3K9me3 lowest
- Acetylation marks (H3K27ac, H3K122ac, H3K18ac, H4K8ac, H4K91ac, H4K16ac, H4K12ac) show a fairly tight distribution with overlap >0.75
- Per-replicate *.enrich.dhs intermediate files are written for each replicate and each ratio lies in [0,1]

## Limitations

- DHS overlap ratios depend on the genome build of the dependent data; ChiLin supplies separate hg19, hg38, mm9 and mm10 archives that must match the peaks
- The published atlas pattern is tied to the 23,677-sample ChiLin quality atlas; small individual datasets (e.g. <200 peaks, where ChiLin skips some steps) may not reproduce the per-mark distributions
- No concrete ChiLin version is stated, so reproducing the exact distributions may require matching the historical reference data rather than the current tool version

## Evidence

- [other] Intersect each sample's sorted peaks against the union DHS regions with bedtools to compute the per-sample overlap ratio (fraction of peaks in DHS): [evidence span withheld — anchor status: not_found]
- [results] Most histone acetylation marks show a fairly tight distribution of high overlap with DHS (>0.75), while non-acetylation marks display fairly high (H3T11ph), intermediate (H3K27me3, H3K36me3), or low (H3K9me3) union DHS overlap ratios: [evidence span withheld — anchor status: not_found]
- [methods] FRiP score calculation step in the ChiLin QC pipeline: "step 7: calculate FRiP scores for each sample."
- [readme] ChiLin installs bedtools as a dependency used for interval operations: "conda install bwa=0.7.13 samtools=0.1.19 bedtools=2.17.0 seqtk ucsc-bedclip ucsc-bedgraphtobigwig ucsc-wigcorrelate ucsc-wigtobigwig fastqc numpy macs2=2.1.0"
