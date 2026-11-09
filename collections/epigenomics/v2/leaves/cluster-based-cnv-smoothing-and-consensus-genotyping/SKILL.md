---
name: cluster-based-cnv-smoothing-and-consensus-genotyping
description: Use when scATAC fragment matrices have already been normalized (normalizeMatrixN/collapseChrom3N),
  CNV clusters identified (identifyCNVClusters/clusterCNV), and per-cluster CNV calls
  need smoothing into consensus genotypes before plotting or per-cell annotation output.
when_to_use_negative:
- The malignancy is driven primarily by single nucleotide variants with no or rare
  CNVs — Copy-scAT explicitly cannot be applied in this setting.
- Input is raw scATAC fragments that have not been filtered and normalized (filterCells/normalizeMatrixN/collapseChrom3N
  must run first).
- The goal is scRNA-seq CNV calling — Copy-scAT fills the gap that no dedicated tool
  existed for scATAC data, not transcriptomic inference.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: Copy-scAT
  role: R package providing identifyCNVClusters, clusterCNV, annotateCNV4B, getDoubleMinutes,
    getLOHRegions, smoothClusters and graphCNVDistribution used for cluster-based
    CNV smoothing and consensus genotyping
  repo: https://github.com/spcdot/CopyscAT
- name: R
  role: Runtime environment for the CopyscAT package (development under R 4.0.0)
- name: NMF
  role: CopyscAT package dependency supporting the CNV inference workflow
- name: fastcluster
  role: CopyscAT package dependency supporting CNV clustering
- name: viridis
  role: CopyscAT package dependency used for plotting CNV distributions
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1126/sciadv.abg6045
    title: 'Copy-scAT: Deconvoluting single-cell chromatin accessibility of genetic
      subclones in cancer'
  source_content_hash: sha256:ddc526861a3a6635fd31455f419c1701f7d36965350aef68e142766f385c693b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/spcdot/CopyscAT
  tools:
  - Copy-scAT
  - NMF
  - R
  - fastcluster
  - viridis
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# Cluster-based CNV smoothing and consensus genotyping

## Summary

A Copy-scAT refinement step in which per-cell CNV calls are smoothed across identified CNV clusters and cells are assigned consensus genotypes, producing coherent arm-level gain/loss and focal amplification profiles per sample. It consolidates noisy single-cell inference into cluster-consistent CNV genotypes for downstream visualization and annotation.

## When to use

Use when scATAC fragment matrices have already been normalized (normalizeMatrixN/collapseChrom3N), CNV clusters identified (identifyCNVClusters/clusterCNV), and per-cluster CNV calls need smoothing into consensus genotypes before plotting or per-cell annotation output.

## When NOT to use

- The malignancy is driven primarily by single nucleotide variants with no or rare CNVs — Copy-scAT explicitly cannot be applied in this setting.
- Input is raw scATAC fragments that have not been filtered and normalized (filterCells/normalizeMatrixN/collapseChrom3N must run first).
- The goal is scRNA-seq CNV calling — Copy-scAT fills the gap that no dedicated tool existed for scATAC data, not transcriptomic inference.

## Inputs

- Normalized, collapsed scATAC fragment count matrix (from normalizeMatrixN and collapseChrom3N)
- CNV results list from identifyCNVClusters
- CNV cluster assignments file from clusterCNV
- Expected pseudodiploid normal profiles for arm-level annotation (annotateCNV4B)

## Outputs

- Cluster-smoothed CNV calls and consensus genotype assignments per cell (cnv_calls_per_cell.tsv)
- Per-arm annotated gains/losses and focal amplification (double minute) calls
- Loss of heterozygosity (LOH) region flags
- Per-sample CNV distribution plots (graphCNVDistribution)

## How to apply

After initializing the environment (initialiseEnvironment) and filtering/normalizing the fragment matrix (filterCells, normalizeMatrixN, collapseChrom3N), run identifyCNVClusters and clusterCNV to group cells sharing CNV patterns. Annotate arm-level gains and losses against expected pseudodiploid normals with annotateCNV4B (saveOutput=TRUE plus an outputSuffix, with parameters such as maxClust2, sdCNV, filterResults, filterRange, and minAlteredCellProp), then detect focal amplifications with getDoubleMinutes and flag LOH regions with getLOHRegions. Apply smoothClusters, passing the input CNV list and the CNV cluster file, to smooth CNV calls across cell clusters so that cells within a cluster share a coherent genotype. Render per-sample CNV distribution plots with graphCNVDistribution and write the per-cell/per-arm annotated calls and consensus genotype assignments to cnv_calls_per_cell.tsv.

## Related tools

- **Copy-scAT** (R package providing identifyCNVClusters, clusterCNV, annotateCNV4B, getDoubleMinutes, getLOHRegions, smoothClusters and graphCNVDistribution used for cluster-based CNV smoothing and consensus genotyping) — https://github.com/spcdot/CopyscAT
- **R** (Runtime environment for the CopyscAT package (development under R 4.0.0))
- **NMF** (CopyscAT package dependency supporting the CNV inference workflow)
- **fastcluster** (CopyscAT package dependency supporting CNV clustering)
- **viridis** (CopyscAT package dependency used for plotting CNV distributions)

## Examples

```
smoothClusters(inputCNVList, cnvClusterFile); graphCNVDistribution(sampName, cnvResults); write.table(cnvCalls, "cnv_calls_per_cell.tsv", sep="\t", quote=FALSE)
```

## Evaluation signals

- cnv_calls_per_cell.tsv exists and contains per-cell, per-arm annotated CNV calls together with consensus genotype assignments.
- Cells sharing a given CNV are concordant within their clusterCNV-assigned cluster after smoothClusters (cluster-consistent genotypes).
- Annotated arm-level calls agree with expected pseudodiploid normals supplied to annotateCNV4B (saveOutput=TRUE with a defined outputSuffix).
- Copy-scAT CNV calls match WGS benchmarking data: the method correctly inferred all or most CNVs called with WGS in validation.
- graphCNVDistribution renders a CNV distribution plot per sample without error, confirming the smoothed CNV list and cluster file were passed correctly.

## Limitations

- Copy-scAT cannot be used with malignancies driven primarily by single nucleotide variants that have no or rare CNVs.
- The optimal way to address CNV effects in downstream scATAC analyses has not yet been identified; further studies are needed.
- Whether CNVs prime cells to acquire specific chromatin accessibility states or stabilize chromatin states of subclones remains undetermined.
- The CopyscAT package is in early beta — documentation and testing are works in progress; it requires an R 4.0.0 environment.
- Clustering based on raw accessibility may be biased; motif-activity-based clustering (e.g., ChromVAR) shows less pronounced effects.

## Evidence

- [methods] smoothClusters rationale: [evidence span withheld — anchor status: not_found]
- [methods] annotateCNV4B parameters: "annotateCNV4B(cnvResults, expectedNormals, saveOutput, maxClust2, outputSuffix, sdCNV, filterResults, filterRange, minAlteredCellProp)"
- [results] benchmarking validation: [evidence span withheld — anchor status: not_found]
- [readme] R environment requirement: "Development is ongoing with R 4.0.0, so please install within an R 4.0.0 environment."
- [abstract] purpose statement: "an R package that uses single-cell epigenomic data to infer copy number variants (CNVs) that define cancer cells"
