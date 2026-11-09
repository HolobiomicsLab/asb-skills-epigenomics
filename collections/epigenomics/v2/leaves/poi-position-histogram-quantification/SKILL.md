---
name: poi-position-histogram-quantification
description: Use when you have DGW clustering outputs (config, warping paths, cluster
  BEDs) plus POI coordinate files and need a quantitative, per-cluster measure of
  whether warping tightened TSS/FSS alignment.
when_to_use_negative:
- When the clustering/alignment step has not yet been run — there are no warped positions
  or warping paths to compare against raw ones
- When the analysed regions contain no points of interest; regions with no entry in
  the fss.poi file must be ignored (--ignore-no-poi-regions) rather than scored
- When assessing alignment quality for features other than genomic landmarks (TSS/FSS)
  — the histogram entropy metric is defined over POI position distributions
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: DGW
  role: Produces the clustering outputs, warping paths, and warped POI positions that
    the histogram-entropy analysis quantifies; dgw-extract-gene-regions (--tss, --splicing-site
    0) supplies the POI coordinates
  repo: github.com/lukauskas/dgw
- name: Python
  role: Language in which the histogram building, Shannon entropy computation, and
    Bootstrap confidence interval estimation are implemented (DGW is an open source
    Python package requiring Python 2.7)
  repo: github.com/lukauskas/dgw
- name: UCSC genome browser
  role: Source of the knownGenes annotation table from which TSS and splicing-site
    coordinates are downloaded
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-016-1306-0
    title: 'DGW: an exploratory data analysis tool for clustering and visualisation
      of epigenomic marks'
  source_content_hash: sha256:95fa7454c8ff2a7d415a1fa0146abd3f2faf43ae82d26234da8350a9f5f67e8e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DGW
  - Python
  - UCSC genome browser
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# POI position histogram quantification

## Summary

Quantifies the quality of DGW alignment of genomic landmarks by building binned histograms of point-of-interest (TSS and first splicing site, FSS) positions before and after warping and computing the Shannon entropy decrease. Percent entropy reduction (e.g. 12.91% TSS, 7.72% FSS in the selected cluster) provides a numeric confirmation that DGW tightens landmark alignment.

## When to use

Use when you have DGW clustering outputs (config, warping paths, cluster BEDs) plus POI coordinate files and need a quantitative, per-cluster measure of whether warping tightened TSS/FSS alignment.

## When NOT to use

- When the clustering/alignment step has not yet been run — there are no warped positions or warping paths to compare against raw ones
- When the analysed regions contain no points of interest; regions with no entry in the fss.poi file must be ignored (--ignore-no-poi-regions) rather than scored
- When assessing alignment quality for features other than genomic landmarks (TSS/FSS) — the histogram entropy metric is defined over POI position distributions

## Inputs

- DGW clustering outputs ({prefix}_config.dgw file, warping paths, cluster BED files)
- TSS point-of-interest coordinates extracted from UCSC knownGenes annotation (POI file)
- First splicing site (FSS, --splicing-site 0) point-of-interest coordinates (POI file)
- poi_regions BED file mapping POI regions to DGW-analysed regions

## Outputs

- Binned histograms of raw (length-rescaled) and warped TSS/FSS position distributions per cluster
- Shannon entropy per histogram, raw and warped
- Percent entropy decrease per cluster and averaged across clusters, with 95% Bootstrap confidence intervals

## How to apply

First extract TSS and first-splicing-site coordinates from the UCSC knownGenes annotation using dgw-extract-gene-regions with the --tss and --splicing-site 0 options. Load the DGW outputs ({prefix}_config.dgw, warping paths, cluster BED files) together with the POI files and recover raw and warped POI positions for every cluster; use the complete-overlap containment requirement to match poi_regions to DGW regions and drop regions without POIs. Build histograms of POI positions over bins, rescaling raw data to a common length and using warped positions per cluster. Compute the Shannon entropy of each histogram and the percent entropy decrease after warping, per cluster and averaged across clusters, with 95% Bootstrap confidence intervals (expected averages: 1.72% TSS, CI 0.83–2.81%; 2.65% FSS, CI 1.79–3.63%). A positive entropy decrease indicates that warping concentrated landmark positions, i.e. tighter alignment.

## Related tools

- **DGW** (Produces the clustering outputs, warping paths, and warped POI positions that the histogram-entropy analysis quantifies; dgw-extract-gene-regions (--tss, --splicing-site 0) supplies the POI coordinates) — github.com/lukauskas/dgw
- **Python** (Language in which the histogram building, Shannon entropy computation, and Bootstrap confidence interval estimation are implemented (DGW is an open source Python package requiring Python 2.7)) — github.com/lukauskas/dgw
- **UCSC genome browser** (Source of the knownGenes annotation table from which TSS and splicing-site coordinates are downloaded)

## Examples

```
dgw-extract-gene-regions --tss --splicing-site 0 knownGenes.txt tss.poi fss.poi
```

## Evaluation signals

- Selected-cluster percent entropy decreases should reproduce the reported values: 12.91% for TSS and 7.72% for FSS after warping
- Averages across clusters should fall within the reported 95% Bootstrap CIs: 1.72% (0.83–2.81%) for TSS and 2.65% (1.79–3.63%) for FSS
- Every cluster region used must contain at least one POI entry — regions absent from the fss.poi file must be excluded
- POI-to-region matching must satisfy complete overlap: all points in a poi_regions.bed entry are contained within the main DGW region
- Raw POI histograms must be computed on positions rescaled to a common length so they are comparable with warped histograms

## Limitations

- DGW is written in Python and needs Python 2.7 to run, which constrains the environment for the analysis scripts
- The dtwco dependency repository is no longer maintained and is read-only as of 2020-10-06
- Entropy decrease is measured only on regions that contain POIs; regions lacking POI entries are ignored, which biases the metric towards landmark-rich regions

## Evidence

- [methods] Extract TSS and first-splicing-site POI coordinates from the UCSC knownGenes annotation using dgw-extract-gene-regions (--tss and --splicing-site 0): "the table set to `knownGenes`. Click `Get output` to get the data and save it to file."
- [methods] Reported entropy decreases in the selected cluster quantify tighter landmark alignment after warping: "we observe a decrease of 12.91 % for TSS and 7.72 % for FSS location distributions in the selected cluster after warping"
- [methods] DGW is an open source Python package requiring Python 2.7: "DGW is written in Python and needs Python 2.7 to run."
- [methods] POI-region matching requires complete containment of the POI region within the main region: "find all the regions in `poi_regions.bed` that overlap *completely* (i.e. *all* points in the points of interest region are contained within the main region)"
- [readme] The DGW library is open source and documented, with installation-from-source guidance: "This is the repository for the open source DGW library developped by Saulius Lukauskas."
