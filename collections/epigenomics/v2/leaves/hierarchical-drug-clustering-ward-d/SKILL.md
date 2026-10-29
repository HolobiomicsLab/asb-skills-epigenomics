---
name: hierarchical-drug-clustering-ward-d
description: Use when you need to organize drugs by similarity of their RePhine TR
  response signatures and validate that drugs with the same mechanism of action (e.g.,
  MEK, RAF, EGFR inhibitors) co-cluster.
when_to_use_negative:
- TR-drug associations are needed as independent biomarkers rather than a global drug-similarity
  view (use RePhine's uniP/multiP candidate ranking instead)
- Reliable ChIP-seq target information is unavailable or has too few targets, which
  would mislead TR identification downstream of RePhine
- Post-treatment/acquired-resistance dynamics are the target of the analysis — CCLE
  lacks post-treatment data, so secondary resistance (e.g., acquired STAT3 activation
  with erlotinib) cannot be captured
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: RePhine
  role: R package that regulates TR-drug pairs to produce uniP statistics transformed
    into the -Log10(uniP) signature matrix used as clustering input
  repo: https://github.com/coexps/RePhine
- name: R
  role: Environment providing hclust (ward.D, complete linkage) and cutree for the
    hierarchical clustering and dendrogram cutting; requires glmnet, MASS, lmtest,
    ppcor, psych, ROCR
  repo: https://github.com/coexps/RePhine
- name: Python
  role: Modified RP score calculation script (RP_calculation_modified.py) that computes
    regulatory potential scores from ChIP-seq peak bed files, feeding the signature
    matrix
  repo: https://github.com/coexps/RePhine
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2019.09.008
    title: '<i>RePhine</i>: An Integrative Method for Identification of Drug Response-Related
      Transcriptional Regulators'
  source_content_hash: sha256:1e072acf475eaa112643ecb2e7ab2c33e14d82c98f0a0de69dbecd0ae74925ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/coexps/RePhine
  tools:
  - Python
  - R
  - RePhine
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# hierarchical-drug-clustering-ward-d

## Summary

Clusters CCLE drugs by mechanism of action using RePhine-derived TR response signatures (-Log10 uniP with direction) via hierarchical clustering in R's hclust, reproducing the Figure 3A grouping where MEK/RAF/MET inhibitors co-cluster.

## When to use

Use when you need to organize drugs by similarity of their RePhine TR response signatures and validate that drugs with the same mechanism of action (e.g., MEK, RAF, EGFR inhibitors) co-cluster.

## When NOT to use

- TR-drug associations are needed as independent biomarkers rather than a global drug-similarity view (use RePhine's uniP/multiP candidate ranking instead)
- Reliable ChIP-seq target information is unavailable or has too few targets, which would mislead TR identification downstream of RePhine
- Post-treatment/acquired-resistance dynamics are the target of the analysis — CCLE lacks post-treatment data, so secondary resistance (e.g., acquired STAT3 activation with erlotinib) cannot be captured

## Inputs

- Precomputed RP (regulatory potential) scores for CCLE drugs across >1000 ENCODE ChIP-seq TR target sets
- ENCODE ChIP-seq target sets for the 160 transcriptional regulators
- CCLE pharmacogenomic data (gene expression, copy number, mutation status, cancer type, pharmacological profiles)
- RePhine regression uniP statistics per TR-drug pair

## Outputs

- 160-TR by N-drug matrix of -Log10(uniP) RePhine response signature values
- Dendrogram from hclust (ward.D, complete linkage) over drugs
- drug_clusters.csv mapping each drug to its cluster assignment

## How to apply

Load the precomputed RP scores for CCLE drugs and the ENCODE ChIP-seq target sets for the 160 transcriptional regulators from the RePhine repository (Google Drive resource of regulatory potential scores across >1000 ENCODE ChIP-seq datasets). Run the RePhine regression R package for each TR-drug pair to obtain uniP statistics and transform them into -Log10(uniP) with direction, yielding a 160-TR by N-drug signature matrix. Apply hierarchical clustering on the drugs using this matrix with the ward.D method and complete linkage in R's hclust, then cut the dendrogram (cutree) to assign drug clusters and write the drug-to-cluster mapping to drug_clusters.csv. The rationale is that RePhine signatures capture TR-level impact on drug response, so drugs sharing a mechanism of action acquire similar signature vectors and co-cluster.

## Related tools

- **RePhine** (R package that regulates TR-drug pairs to produce uniP statistics transformed into the -Log10(uniP) signature matrix used as clustering input) — https://github.com/coexps/RePhine
- **R** (Environment providing hclust (ward.D, complete linkage) and cutree for the hierarchical clustering and dendrogram cutting; requires glmnet, MASS, lmtest, ppcor, psych, ROCR) — https://github.com/coexps/RePhine
- **Python** (Modified RP score calculation script (RP_calculation_modified.py) that computes regulatory potential scores from ChIP-seq peak bed files, feeding the signature matrix) — https://github.com/coexps/RePhine

## Examples

```
hc <- hclust(dist(signature_matrix), method="ward.D"); drug_clusters <- cutree(hc, k=3); write.csv(data.frame(drug=rownames(signature_matrix), cluster=drug_clusters), "drug_clusters.csv", row.names=FALSE)
```

## Evaluation signals

- MEK inhibitors PD-0325901 and AZD6244, RAF inhibitors PLX4720 and RAF265, and MET inhibitor PHA-665752 all fall in the MAPK-related cluster 2.2 as in Figure 3A
- Chemotherapy drugs plus panobinostat co-cluster in cluster 1, matching Figure 3A
- EGFR inhibitors (erlotinib, AZD0530, lapatinib, ZD-6474) group with 17-AAG in cluster 3
- Signature matrix dimensions are 160 TRs by the full CCLE drug set
- drug_clusters.csv exists and contains a cluster assignment for every drug in the signature matrix

## Limitations

- RePhine is restricted to cases where reliable target information is available; lack of ChIP-seq data or ChIP-seq with too few targets would mislead TR identification
- TR-drug relationships identified through targets may not be causal; upstream regulators or kinases may confound
- Detection of acquired resistance is limited due to lack of post-treatment data in CCLE
- Cluster assignment depends on the exact clustering parameters (ward.D with complete linkage) and signature matrix; mismatch with Figure 3A requires revisiting parameters or the matrix

## Evidence

- [abstract] we observed that RePhine-derived TR signatures could effectively cluster drugs with different mechanisms of action.: "we observed that RePhine-derived TR signatures could effectively cluster drugs with different mechanisms of action."
- [other] Apply hierarchical clustering on the drugs using the -Log10(uniP) TR signature matrix with the ward.D method and complete linkage in R's hclust.: [evidence span withheld — anchor status: not_found]
- [readme] Regulatory potential scores across more than 1000 ChIP-seq data from Encode that are used in RePhine analysis are available on google drive.: [evidence span withheld — anchor status: not_found]
- [readme] Use likelihood ratio test to evaluate significance (uniP) of the TRs whose targets have concordant positive or negative partial correlations with drug response.: "Use likelihood ratio test to evaluate significance (uniP) of the TRs whose targets have concordant positive or negative partial correlations with drug response"
- [discussion] Lack of ChIP-seq data or ChIP-seq data with too few targets would mislead the identification of TRs.: "Lack of ChIP-seq data or ChIP-seq data with too few targets would mislead the identification of TRs"
