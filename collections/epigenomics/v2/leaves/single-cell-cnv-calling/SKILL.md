---
name: single-cell-cnv-calling
description: Use when starting from a scATAC-seq fragment count matrix (e.g., scDataSamp)
  and the goal is per-cell CNV inference, malignant/non-neoplastic cell classification,
  or linking genetic subclones to chromatin accessibility.
when_to_use_negative:
- Malignancies driven primarily by single nucleotide variants with no or rare CNVs
  — the article states Copy-scAT cannot be used for these
- Bulk or non-single-cell copy number data — the method is designed for single-cell
  epigenomic (scATAC-seq) input
- Analyses where a per-cell feature table or genotype matrix is already available
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Copy-scAT
  role: Core R package performing all CNV inference, clustering, annotation, and visualization
    steps
  repo: github.com/spcdot/CopyscAT
- name: R
  role: Runtime environment; README specifies installation and use within an R 4.0.0
    environment
  repo: github.com/spcdot/CopyscAT
- name: NMF
  role: Declared dependency of the CopyscAT package
- name: fastcluster
  role: Declared dependency used for clustering CNV profiles
- name: viridis
  role: Declared dependency for plotting/visualization
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1126/sciadv.abg6045
    title: 'Copy-scAT: Deconvoluting single-cell chromatin accessibility of genetic
      subclones in cancer'
  source_content_hash: sha256:8677fe49acbe93a2630a59b7ba49e6569873c1076543e88839b60840f62f7671
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Copy-scAT
  - NMF
  - R
  - fastcluster
  - viridis
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# single-cell-cnv-calling

## Summary

Call copy number variants (chromosome arm-level gains/losses and focal amplifications) from single-cell ATAC-seq data using the Copy-scAT R package, enabling discrimination of malignant from nonmalignant cells and deconvolution of genetic subclones without a paired control file.

## When to use

Use when starting from a scATAC-seq fragment count matrix (e.g., scDataSamp) and the goal is per-cell CNV inference, malignant/non-neoplastic cell classification, or linking genetic subclones to chromatin accessibility.

## When NOT to use

- Malignancies driven primarily by single nucleotide variants with no or rare CNVs — the article states Copy-scAT cannot be used for these
- Bulk or non-single-cell copy number data — the method is designed for single-cell epigenomic (scATAC-seq) input
- Analyses where a per-cell feature table or genotype matrix is already available

## Inputs

- Single-cell ATAC-seq fragment count matrix (e.g., packaged scDataSamp dataset)
- Cell cluster labels (e.g., scDataSampClusters)
- Expected pseudodiploid normals annotation for baseline correction

## Outputs

- Per-cell and per-arm annotated CNV calls (cnv_calls_per_cell.tsv)
- CNV cluster assignments and consensus genotype assignments
- Focal amplification (double minute) calls and LOH region annotations
- Neoplastic vs non-neoplastic cell labels and per-sample CNV distribution plots

## How to apply

In R 4.0.0, load the CopyscAT package and initialise the analysis environment with initialiseEnvironment, then filter low-quality cells with filterCells and normalize/collapse the fragment count matrix with normalizeMatrixN and collapseChrom3N. Identify CNV clusters via identifyCNVClusters and clusterCNV (which takes medianIQR, maxClust, minDiff parameters), then annotate arm-level gains and losses against expected pseudodiploid normals using annotateCNV4B (saveOutput=TRUE with an outputSuffix; annotateCNV4/4B accept sdCNV, filterResults, filterRange, minAlteredCellProp). Detect focal amplifications with getDoubleMinutes, label non-neoplastic cells with identifyNonNeoplastic, flag loss of heterozygosity regions with getLOHRegions, and smooth calls across clusters with smoothClusters before rendering per-sample plots with graphCNVDistribution. The rationale is that cells sharing a given CNV tend to cluster together in scATAC experiments, so accessibility data itself carries copy-number signal sufficient for per-cell genotyping without a control.

## Related tools

- **Copy-scAT** (Core R package performing all CNV inference, clustering, annotation, and visualization steps) — github.com/spcdot/CopyscAT
- **R** (Runtime environment; README specifies installation and use within an R 4.0.0 environment) — github.com/spcdot/CopyscAT
- **NMF** (Declared dependency of the CopyscAT package)
- **fastcluster** (Declared dependency used for clustering CNV profiles)
- **viridis** (Declared dependency for plotting/visualization)

## Examples

```
cnvResults <- clusterCNV(initialiseEnvironment(scDataSamp, scDataSampClusters), medianIQR=0.25, maxClust=8, minDiff=0.1); annotateCNV4B(cnvResults, expectedNormals, saveOutput=TRUE, outputSuffix="sample1")
```

## Evaluation signals

- Copy-scAT should infer all or most CNVs that were called with WGS benchmarking data in matched samples
- Malignant cells should show CNV calls while nonmalignant/pseudodiploid cells cluster as normals, and identifyNonNeoplastic should flag the latter consistently
- Cells sharing a given CNV should cluster together in the clusterCNV output (consistent CNV-defined subclones)
- Annotated output files (per-cell/per-arm CNVs with saveOutput=TRUE and outputSuffix) must be written and conform to cnv_calls_per_cell.tsv structure
- Focal amplification calls from getDoubleMinutes should correspond to known focal events (e.g., double minutes) in the sample

## Limitations

- Cannot be used with malignancies driven primarily by single nucleotide variants and having no or rare CNVs
- The optimal way to address CNV effects in downstream scATAC analyses is not yet identified; further studies are needed
- It remains undetermined whether CNVs prime cells to acquire specific chromatin accessibility states or stabilize chromatin states of genetic subclones
- Package is in early beta per the README — documentation and testing are still works in progress, and development requires R 4.0.0

## Evidence

- [abstract] Copy-scAT, an R package that uses single-cell epigenomic data to infer copy number variants (CNVs) that define cancer cells: "Here, we describe Copy-scAT, an R package that uses single-cell epigenomic data to infer copy number variants (CNVs) that define cancer cells."
- [intro] Objective to call focal amplifications and chromosome arm-level gains and losses: "we demonstrate the effectiveness of Copy-scAT in calling (i) focal amplifications and (ii) chromosome arm-level gains and losses"
- [methods] annotateCNV4B function signature against expected pseudodiploid normals: "annotateCNV4B(cnvResults, expectedNormals, saveOutput, maxClust2, outputSuffix, sdCNV, filterResults, filterRange, minAlteredCellProp)"
- [methods] clusterCNV function signature with clustering parameters: "clusterCNV(initialResultList, medianIQR, maxClust, minDiff)"
- [readme] README note on early beta status and R 4.0.0 requirement: "It is currently in early beta - documentation and testing are still works in progress. Development is ongoing with R 4.0.0, so please install within an R 4.0.0 environment."
