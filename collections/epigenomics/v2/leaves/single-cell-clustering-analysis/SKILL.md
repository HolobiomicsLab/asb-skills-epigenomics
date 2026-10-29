---
name: single-cell-clustering-analysis
description: Use when scATAC clustering may be driven by CNVs rather than genuine
  accessibility states, and you need to test cluster stability by removing CNV-bearing
  chromosome peaks and comparing pre/post-reclustering assignments via AMI.
when_to_use_negative:
- Malignancy is driven primarily by single nucleotide variants with no or rare CNVs
  — Copy-scAT cannot infer CNVs there, so CNV-peak removal is undefined
- Clustering is based on motif activity scores (e.g., ChromVAR), where CNV effects
  are less pronounced and peak removal is not the relevant control
- The input is already a CNV-free feature matrix with no need to test CNV confounding
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: Copy-scAT
  role: CNV inference (identifyCNVClusters) and hierarchical clustering workflow applied
    to the accessibility matrix
  repo: github.com/spcdot/CopyscAT
- name: CopyscAT
  role: R package supplying scDataSamp and scDataSampClusters datasets and clustering
    functions
  repo: github.com/spcdot/CopyscAT
- name: fastcluster
  role: Hierarchical clustering dependency used to cluster cells on the CNV-free peak
    matrix
- name: R
  role: Execution environment (R 4.0.0 per README) for the CopyscAT workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1126/sciadv.abg6045
    title: 'Copy-scAT: Deconvoluting single-cell chromatin accessibility of genetic
      subclones in cancer'
  source_content_hash: sha256:21a55f8b62680d9c71bd6ea96f2678d8504c16e50c62178764c29101b08d5e4c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Copy-scAT
  - CopyscAT
  - R
  - fastcluster
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# single-cell-clustering-analysis

## Summary

Recluster single-cell ATAC-seq data after removing CNV-harboring chromosome peaks to test whether copy number variants confound clustering, quantifying cluster stability with adjusted mutual information (AMI) against a reference value (0.634 in Copy-scAT Fig. 4I).

## When to use

Use when scATAC clustering may be driven by CNVs rather than genuine accessibility states, and you need to test cluster stability by removing CNV-bearing chromosome peaks and comparing pre/post-reclustering assignments via AMI.

## When NOT to use

- Malignancy is driven primarily by single nucleotide variants with no or rare CNVs — Copy-scAT cannot infer CNVs there, so CNV-peak removal is undefined
- Clustering is based on motif activity scores (e.g., ChromVAR), where CNV effects are less pronounced and peak removal is not the relevant control
- The input is already a CNV-free feature matrix with no need to test CNV confounding

## Inputs

- Single-cell accessibility matrix (peaks x cells), e.g. scDataSamp from CopyscAT
- Original cell cluster assignments, e.g. scDataSampClusters from CopyscAT
- Copy-scAT CNV calls identifying CNV-harboring chromosomes

## Outputs

- Adjusted mutual information (AMI) between original and post-CNVI-removal cluster assignments
- Post-reclustering cluster labels per cell
- CSV of AMI value, reference value (0.634), and cluster labels (e.g. ami_cnv_confounder.csv)

## How to apply

Load the scATAC accessibility matrix and original cluster assignments (e.g., scDataSamp and scDataSampClusters from the CopyscAT package). Determine which chromosomes the Copy-scAT CNV calls (identifyCNVClusters) mark as CNV-bearing, and drop all peaks on those chromosomes, retaining only CNV-free peaks. Recluster the cells on the CNV-free peak matrix using the Copy-scAT clustering workflow (identifyCNVClusters with default parameters, hierarchical clustering via fastcluster). Compute adjusted mutual information (AMI) between original and post-removal cluster labels on the shared set of cells; the paper reproduces AMI = 0.634 (Fig. 4I), indicating more granular but overall stable clustering. Rationale: cells sharing a CNV tend to cluster together in scATAC, so removing CNV peaks isolates the genetic contribution to the accessibility clustering structure.

## Related tools

- **Copy-scAT** (CNV inference (identifyCNVClusters) and hierarchical clustering workflow applied to the accessibility matrix) — github.com/spcdot/CopyscAT
- **CopyscAT** (R package supplying scDataSamp and scDataSampClusters datasets and clustering functions) — github.com/spcdot/CopyscAT
- **fastcluster** (Hierarchical clustering dependency used to cluster cells on the CNV-free peak matrix)
- **R** (Execution environment (R 4.0.0 per README) for the CopyscAT workflow)

## Examples

```
Rscript -e 'library(CopyscAT); cnv <- identifyCNVClusters(scDataSamp); keep <- !peaksOnCNVChroms(cnv); reclust <- identifyCNVClusters(scDataSamp[keep,]); ami <- aricode::AMI(scDataSampClusters, reclust)'
```

## Evaluation signals

- Computed AMI should reproduce the reported reference value of 0.634 (Fig. 4I) within reasonable tolerance
- Post-removal clustering should be more granular than the original clustering (more clusters)
- Moderate cluster concordance between original and post-removal assignments (AMI well above 0 but below 1)
- Peaks on all CNV-bearing chromosomes must be fully removed; verify only CNV-free peaks remain in the reclustering matrix
- AMI must be computed on the shared set of cells present in both original and post-removal assignments

## Limitations

- Copy-scAT cannot be used with malignancies driven primarily by single nucleotide variants with no or rare CNVs
- The optimal way to address CNV effects in downstream scATAC analyses is not yet identified; further studies are needed
- CopyscAT is in early beta — documentation and testing are works in progress; requires an R 4.0.0 environment

## Evidence

- [intro] cells sharing a given CNV tend to cluster together: "cells that share a given CNV tend to cluster together in scATAC experiments, suggesting that genetics may impart information"
- [methods] fastcluster dependency: "fastcluster (Depends)"
- [discussion] ChromVAR reduces CNV effects: "These effects are less pronounced when clustering is based on motif activity scores (e.g., ChromVAR)"
- [discussion] optimal CNV handling unresolved: "Further studies are needed to identify the optimal way to address the effects of CNVs in downstream analyses of scATAC datasets"
- [readme] early beta status: "It is currently in early beta - documentation and testing are still works in progress."
