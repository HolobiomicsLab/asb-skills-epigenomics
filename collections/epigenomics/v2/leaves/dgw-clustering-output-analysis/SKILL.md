---
name: dgw-clustering-output-analysis
description: Use when you have DGW clustering outputs (config, warping paths, cluster
  BED files) plus gene-annotation POI coordinates and need to verify that warping
  improves alignment of TSS/first splicing sites.
when_to_use_negative:
- When DGW warping paths and cluster outputs have not yet been generated (run the
  DGW clustering/alignment pipeline first)
- When regions lack points of interest entirely — such regions are excluded via --ignore-no-poi-regions
  rather than analysed
- When the goal is to assess clustering quality of the epigenomic mark shapes themselves
  rather than landmark (TSS/FSS) alignment tightness
edam_operation: http://edamontology.org/operation_3436
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: DGW
  role: Produces the clustering outputs, warping paths, and gene-region/POI extraction
    used for the entropy analysis (dgw-extract-gene-regions --tss --splicing-site
    0)
  repo: github.com/lukauskas/dgw
- name: Python
  role: Open source language in which DGW is implemented and in which the histogram/entropy/Bootstrap
    analysis of POI distributions is performed
  repo: github.com/lukauskas/dgw
- name: UCSC genome browser
  role: Source of the knownGenes annotation table downloaded to obtain gene/TSS/FSS
    coordinates
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-016-1306-0
    title: 'DGW: an exploratory data analysis tool for clustering and visualisation
      of epigenomic marks'
  source_content_hash: sha256:d3c004cd0e52fc2cca9dcf4e4f76eb6f0eaaaf515be4c08411b6fe6deab31cee
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DGW
  - Python
  - UCSC genome browser
  edam_operation: http://edamontology.org/operation_3436
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# dgw-clustering-output-analysis

## Summary

Quantifies whether Dynamic Genome Warping (DGW) alignment tightens genomic landmark positioning by computing Shannon entropy decreases of transcription start site (TSS) and first splicing site (FSS) location distributions per DGW cluster, comparing raw versus warped point-of-interest positions.

## When to use

Use when you have DGW clustering outputs (config, warping paths, cluster BED files) plus gene-annotation POI coordinates and need to verify that warping improves alignment of TSS/first splicing sites.

## When NOT to use

- When DGW warping paths and cluster outputs have not yet been generated (run the DGW clustering/alignment pipeline first)
- When regions lack points of interest entirely — such regions are excluded via --ignore-no-poi-regions rather than analysed
- When the goal is to assess clustering quality of the epigenomic mark shapes themselves rather than landmark (TSS/FSS) alignment tightness

## Inputs

- DGW clustering outputs: {prefix}_config.dgw file, warping paths, and per-cluster BED files
- TSS POI coordinates file extracted from UCSC knownGenes table (dgw-extract-gene-regions --tss)
- FSS POI coordinates file extracted from UCSC knownGenes table (dgw-extract-gene-regions --splicing-site 0)
- DGW input data file of binned ChIP-Seq coverage (50 bp non-overlapping windows) from ENCODE K562 H3K4me3/H3K9ac peaks

## Outputs

- Per-cluster raw and warped TSS/FSS position histograms over bins
- Shannon entropy values for raw and warped TSS and FSS distributions per cluster
- Percent entropy decrease after warping per cluster and averaged across clusters, with 95% Bootstrap confidence intervals (e.g. TSS 1.72%, CI 0.83–2.81%; FSS 2.65%, CI 1.79–3.63%)

## How to apply

Extract TSS and first-splicing-site coordinates from UCSC knownGenes annotation using dgw-extract-gene-regions with the --tss and --splicing-site 0 flags. Load the DGW clustering outputs ({prefix}_config.dgw, warping paths, cluster BED files) together with the POI files and recover raw and warped POI positions for every cluster. Build histograms of TSS and FSS positions over bins for the raw data (rescaled to a common length) and for the warped data in each cluster, then compute the Shannon entropy of each histogram. Report the percent entropy decrease after warping per cluster and averaged across clusters; in the ENCODE K562 H3K4me3/H3K9ac study this yielded 12.91% (TSS) and 7.72% (FSS) decrease in the selected cluster, and 1.72% (TSS, 95% Bootstrap CI 0.83–2.81%) and 2.65% (FSS, 95% Bootstrap CI 1.79–3.63%) on average. Use --ignore-no-poi-regions to drop regions lacking POI entries, and require complete containment of POI regions within the main region when matching.

## Related tools

- **DGW** (Produces the clustering outputs, warping paths, and gene-region/POI extraction used for the entropy analysis (dgw-extract-gene-regions --tss --splicing-site 0)) — github.com/lukauskas/dgw
- **Python** (Open source language in which DGW is implemented and in which the histogram/entropy/Bootstrap analysis of POI distributions is performed) — github.com/lukauskas/dgw
- **UCSC genome browser** (Source of the knownGenes annotation table downloaded to obtain gene/TSS/FSS coordinates)

## Examples

```
dgw-extract-gene-regions --tss --splicing-site 0 knownGenes.txt genes.bed
```

## Evaluation signals

- Selected-cluster entropy decreases reproduce the reported values: 12.91% for TSS and 7.72% for FSS after warping
- Across-cluster average decreases reproduce 1.72% for TSS (95% Bootstrap CI 0.83–2.81%) and 2.65% for FSS (95% Bootstrap CI 1.79–3.63%)
- Raw histograms are rescaled to a common length before entropy comparison so raw-vs-warped entropies are comparable
- Every POI entry used is completely contained within the corresponding main region (complete-overlap containment check)
- Regions without any POI entries in the fss.poi file are excluded (--ignore-no-poi-regions) rather than contributing zero-count histograms

## Limitations

- Entropy decrease is small on average (1.72% TSS, 2.65% FSS across clusters); effect sizes vary by cluster and only reach double digits in selected clusters
- Analysis depends on annotation-derived POI coordinates (UCSC knownGenes); DGW alignment itself is unsupervised and does not use gene annotation
- The DGW repository is archived (GPL-3.0, no longer maintained) and requires Python 2.7 to run
- Peaks were filtered to lengths between 5 and 1000 bins and at least 10 counts, so results apply only to peaks passing these filters

## Evidence

- [methods] Selected-cluster entropy decreases: "we observe a decrease of 12.91 % for TSS and 7.72 % for FSS location distributions in the selected cluster after warping"
- [methods] POI extraction tooling: "the table set to `knownGenes`. Click `Get output` to get the data and save it to file."
- [methods] Ignore-no-POI-regions option: "--ignore-no-poi-regions -- Ignore all regions that do not have any entry in `fss.poi`"
- [methods] Complete containment requirement: "find all the regions in `poi_regions.bed` that overlap *completely* (i.e. *all* points in the points of interest region are contained within the main region)"
- [abstract] DGW aligns genomic landmarks: "DGW automatically recognises and aligns important genomic features such as transcription start sites and splicing sites from histone marks."
- [readme] DGW documentation pointer: "Please consult the documentation on https://dgw.readthedocs.org/ for information how to install and use it."
