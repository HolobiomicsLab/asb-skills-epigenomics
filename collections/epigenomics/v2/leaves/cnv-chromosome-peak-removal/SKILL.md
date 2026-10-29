---
name: cnv-chromosome-peak-removal
description: Use when scATAC clusters may be confounded by CNVs and you need to assess
  how cluster structure changes after removing peaks from CNV-harboring chromosomes
  before reclustering.
when_to_use_negative:
- The malignancy is driven primarily by single nucleotide variants with no or rare
  CNVs, since Copy-scAT cannot be applied there
- The clustering is based on motif activity scores (e.g., ChromVAR), where CNV effects
  are less pronounced and peak-removal is not the relevant confounder test
- Input is already a CNV-free peak matrix with no CNV calls available to identify
  which chromosomes to remove
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3673
tools:
- name: CopyscAT
  role: R package supplying the scDataSamp/scDataSampClusters datasets and the CNV-inference/clustering
    workflow used to remove CNV peaks and recluster
  repo: github.com/spcdot/CopyscAT
- name: Copy-scAT
  role: Method used to infer CNVs that define which chromosomes are removed, and whose
    clustering workflow (identifyCNVClusters) is rerun on CNV-free peaks
  repo: github.com/spcdot/CopyscAT
- name: fastcluster
  role: Dependency used for hierarchical clustering of cells on the CNV-free peak
    matrix during reclustering
  repo: github.com/spcdot/CopyscAT
- name: R
  role: Implementation environment (R 4.0.0 per README) in which the removal-and-reclustering
    analysis is executed
  repo: github.com/spcdot/CopyscAT
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1126/sciadv.abg6045
    title: 'Copy-scAT: Deconvoluting single-cell chromatin accessibility of genetic
      subclones in cancer'
  source_content_hash: sha256:8c1091c922c742822e68c1b1b4fdae5fce7c043307f100b4dcf33ed7d945ea49
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Copy-scAT
  - CopyscAT
  - R
  - fastcluster
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# cnv-chromosome-peak-removal

## Summary

A Copy-scAT-based workflow that tests whether inferred copy number variants drive scATAC clustering by removing all peaks on CNV-bearing chromosomes and reclustering cells, quantifying cluster concordance with adjusted mutual information (AMI).

## When to use

Use when scATAC clusters may be confounded by CNVs and you need to assess how cluster structure changes after removing peaks from CNV-harboring chromosomes before reclustering.

## When NOT to use

- The malignancy is driven primarily by single nucleotide variants with no or rare CNVs, since Copy-scAT cannot be applied there
- The clustering is based on motif activity scores (e.g., ChromVAR), where CNV effects are less pronounced and peak-removal is not the relevant confounder test
- Input is already a CNV-free peak matrix with no CNV calls available to identify which chromosomes to remove

## Inputs

- scDataSamp single-cell accessibility dataset (CopyscAT package)
- scDataSampClusters dataset with original cell cluster assignments (CopyscAT package)
- Copy-scAT CNV calls identifying CNV-bearing chromosomes

## Outputs

- ami_cnv_confounder.csv containing the computed AMI value, the reference AMI of 0.634, and cluster labels
- Post-removal cluster assignments for the shared set of cells

## How to apply

Load the scDataSamp accessibility matrix and scDataSampClusters original cluster assignments from the CopyscAT package. From the Copy-scAT CNV calls, determine which chromosomes are predicted to harbor CNVs and drop all peaks mapping to those chromosomes, retaining only CNV-free peaks. Recluster the cells on the CNV-free peak matrix using the Copy-scAT clustering workflow (identifyCNVClusters with default parameters, hierarchical clustering via fastcluster). Compute adjusted mutual information (AMI) between original and post-removal cluster assignments over the shared cells; the paper reports AMI = 0.634 (Fig. 4I), indicating more granular but overall stable clustering. Write the computed AMI, the reference value 0.634, and cluster labels to ami_cnv_confounder.csv, flagging divergence if the reproduced AMI falls far from 0.634.

## Related tools

- **CopyscAT** (R package supplying the scDataSamp/scDataSampClusters datasets and the CNV-inference/clustering workflow used to remove CNV peaks and recluster) — github.com/spcdot/CopyscAT
- **Copy-scAT** (Method used to infer CNVs that define which chromosomes are removed, and whose clustering workflow (identifyCNVClusters) is rerun on CNV-free peaks) — github.com/spcdot/CopyscAT
- **fastcluster** (Dependency used for hierarchical clustering of cells on the CNV-free peak matrix during reclustering) — github.com/spcdot/CopyscAT
- **R** (Implementation environment (R 4.0.0 per README) in which the removal-and-reclustering analysis is executed) — github.com/spcdot/CopyscAT

## Examples

```
library(CopyscAT); data(scDataSamp); data(scDataSampClusters); cnvFree <- scDataSamp[!rownames(scDataSamp) %in% cnvPeaks, ]; reclust <- identifyCNVClusters(cnvFree); ami_cnv_confounder <- aricode::AMI(scDataSampClusters, reclust)
```

## Evaluation signals

- Computed AMI between original and post-removal cluster assignments reproduces the reported reference value of 0.634 (Fig. 4I) within reasonable tolerance
- Post-removal clustering is more granular but overall stable, with moderate cluster concordance relative to the original assignments
- Only peaks on chromosomes predicted to harbor CNVs are removed; all CNV-free peaks are retained in the reclustering matrix
- The shared set of cells is identical between original and post-removal cluster assignments used for the AMI calculation
- ami_cnv_confounder.csv contains the AMI value, the reference value 0.634, and cluster labels; deviation triggers a flag that CNV-peak removal or reclustering parameters diverge from the paper

## Limitations

- Copy-scAT cannot be used with malignancies driven primarily by single nucleotide variants with no or rare CNVs
- The optimal way to address CNV effects in downstream scATAC analyses has not yet been identified; further studies are needed
- CopyscAT is in early beta; documentation and testing are still works in progress and it requires an R 4.0.0 environment

## Evidence

- [intro] Copy-scAT CNV clustering: "cells that share a given CNV tend to cluster together in scATAC experiments, suggesting that genetics may impart information that results in the emergence of specific epigenetic profiles"
- [abstract] Copy-scAT CNV inference: "Here, we describe Copy-scAT, an R package that uses single-cell epigenomic data to infer copy number variants (CNVs) that define cancer cells."
- [methods] fastcluster dependency: "fastcluster (Depends)"
- [methods] sample datasets: "scDataSamp (CopyscAT)"
- [readme] README beta status: "It is currently in early beta - documentation and testing are still works in progress. Development is ongoing with R 4.0.0, so please install within an R 4.0.0 environment."
- [discussion] ChromVAR alternative: "These effects are less pronounced when clustering is based on motif activity scores (e.g., ChromVAR)"
- [discussion] CNV confounder caveat: "Further studies are needed to identify the optimal way to address the effects of CNVs in downstream analyses of scATAC datasets"
