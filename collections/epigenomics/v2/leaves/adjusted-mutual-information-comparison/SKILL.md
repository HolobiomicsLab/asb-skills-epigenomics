---
name: adjusted-mutual-information-comparison
description: Use when two cluster label vectors cover the same set of cells and you
  need a chance-corrected score of clustering stability, e.g. before vs. after removing
  CNV-harboring chromosome peaks from a scATAC accessibility matrix.
when_to_use_negative:
- The two cluster assignments cover different sets of cells — AMI must be computed
  on the shared cell set only
- You need to compare partitions of different cardinality or with unpaired labels
  rather than per-cell matched assignments
- The goal is assessing CNV inference quality itself rather than stability of downstream
  cell clustering
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3611
tools:
- name: CopyscAT
  role: Source of scDataSamp/scDataSampClusters example data and the identifyCNVClusters
    workflow that generates the cluster labels being compared
  repo: github.com/spcdot/CopyscAT
- name: Copy-scAT
  role: Provides CNV calls used to identify CNV-bearing chromosomes whose peaks are
    removed before reclustering
- name: fastcluster
  role: Hierarchical clustering backend used in the Copy-scAT reclustering step whose
    output labels feed the AMI computation
- name: R
  role: Implementation environment for the CopyscAT workflow and AMI computation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1126/sciadv.abg6045
    title: 'Copy-scAT: Deconvoluting single-cell chromatin accessibility of genetic
      subclones in cancer'
  source_content_hash: sha256:32f21e6564c1215a646b38b4c31e5ef852f1020a00ceed1cc92e3e4b82897adc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Copy-scAT
  - CopyscAT
  - R
  - fastcluster
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3611
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# adjusted-mutual-information-comparison

## Summary

Quantifies concordance between two cluster assignments of the same cells using adjusted mutual information (AMI), as used in Copy-scAT to test whether removing CNV-bearing peaks changes scATAC clustering structure (reported AMI = 0.634, Fig. 4I).

## When to use

Use when two cluster label vectors cover the same set of cells and you need a chance-corrected score of clustering stability, e.g. before vs. after removing CNV-harboring chromosome peaks from a scATAC accessibility matrix.

## When NOT to use

- The two cluster assignments cover different sets of cells — AMI must be computed on the shared cell set only
- You need to compare partitions of different cardinality or with unpaired labels rather than per-cell matched assignments
- The goal is assessing CNV inference quality itself rather than stability of downstream cell clustering

## Inputs

- Original cell cluster assignments (e.g. scDataSampClusters from the CopyscAT package)
- Post-reclustering cluster assignments from the CNV-free peak accessibility matrix
- Set of shared cell identifiers present in both clusterings

## Outputs

- Adjusted mutual information (AMI) value quantifying cluster concordance
- Validation report comparing computed AMI to the expected reference value of 0.634
- CSV file (e.g. ami_cnv_confounder.csv) containing the AMI, reference value, and cluster labels

## How to apply

Load the original cluster assignments (e.g. scDataSampClusters from CopyscAT) alongside the post-reclustering labels produced after filtering peaks on CNV-bearing chromosomes and reclustering with identifyCNVClusters/fastcluster. Restrict both label vectors to the shared set of cells so the comparison is on identical identities, then compute adjusted mutual information (AMI), which corrects for agreement expected by chance. Compare the computed AMI to the paper's reference of 0.634 (Fig. 4I) within reasonable tolerance; a large deviation indicates the CNV-peak removal or reclustering parameters diverge from the published workflow. Write the AMI value, the reference value, and the cluster labels to an output file (ami_cnv_confounder.csv) for reproducibility and validation.

## Related tools

- **CopyscAT** (Source of scDataSamp/scDataSampClusters example data and the identifyCNVClusters workflow that generates the cluster labels being compared) — github.com/spcdot/CopyscAT
- **Copy-scAT** (Provides CNV calls used to identify CNV-bearing chromosomes whose peaks are removed before reclustering)
- **fastcluster** (Hierarchical clustering backend used in the Copy-scAT reclustering step whose output labels feed the AMI computation)
- **R** (Implementation environment for the CopyscAT workflow and AMI computation)

## Examples

```
ami <- arules::? no — use sklearn: from sklearn.metrics import adjusted_mutual_info_score; ami = adjusted_mutual_info_score(orig_clusters, reclust_clusters)
```

## Evaluation signals

- Computed AMI reproduces the reported value of 0.634 (Fig. 4I) within reasonable tolerance
- AMI is computed on the same shared set of cells present in both the original and post-removal cluster assignments
- Post-removal clustering is more granular but overall stable, consistent with the reported moderate concordance
- Output file (e.g. ami_cnv_confounder.csv) contains the AMI value, the reference value 0.634, and cluster labels
- Large deviation from 0.634 flags divergence in CNV-peak removal or identifyCNVClusters/fastcluster reclustering parameters

## Limitations

- AMI reflects overall partition agreement and does not capture which specific clusters merged or split after CNV-peak removal
- Interpretation depends on matching the paper's exact reclustering parameters (identifyCNVClusters defaults, fastcluster hierarchical clustering); parameter drift changes the expected value
- Further studies are needed to identify the optimal way to address the effects of CNVs in downstream analyses of scATAC datasets, so 0.634 is a paper-specific benchmark rather than a universal threshold

## Evidence

- [results] Clustering after CNV removal is more granular but overall stable, with moderate cluster concordance: [evidence span withheld — anchor status: not_found]
- [discussion] These effects are less pronounced when clustering is based on motif activity scores: "These effects are less pronounced when clustering is based on motif activity scores (e.g., ChromVAR)"
- [readme] CopyscAT is designed to identify large-scale and local alterations in chromosomes without need for a control file: "CopyscAT is designed to identify large-scale and local alterations in chromosomes without need for a control file."
