---
name: rephine-response-signature-computation
description: Use when the goal is to organize anti-cancer drugs (e.g., CCLE compounds)
  into mechanism-of-action groups from TR-drug response associations, or to reproduce
  the RePhine drug-cluster dendrogram of Figure 3A.
when_to_use_negative:
- ChIP-seq data for a TR is missing or contains too few targets — the article warns
  this would mislead TR identification, so the signature values are unreliable
- The goal is inferring acquired/secondary resistance from post-treatment dynamics
  — CCLE lacks post-treatment data (e.g., erlotinib resistance via acquired STAT3
  activation cannot be detected)
- Input is already a curated drug-feature table not derived from TR response associations
  — clustering would not reflect RePhine's mechanism-of-action signature
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: RePhine
  role: R package that regresses TR ChIP-seq targets against drug response to produce
    uniP statistics used to build the -Log10(uniP) signature matrix
  repo: https://github.com/coexps/RePhine
- name: R
  role: Language in which RePhine is implemented and in which hclust (ward.D) hierarchical
    clustering of drugs is performed
- name: Python
  role: Runs the modified RP score calculation script (RP_calculation_modified.py)
    producing regulatory potential scores used as RePhine input
  repo: https://github.com/coexps/RePhine
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2019.09.008
    title: '<i>RePhine</i>: An Integrative Method for Identification of Drug Response-Related
      Transcriptional Regulators'
  source_content_hash: sha256:1ca03d9ffad761b90ff497503f25a7c2843db956283d4466ecbda8b6b099cfab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/coexps/RePhine
  tools:
  - Python
  - R
  - RePhine
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# rephine-response-signature-computation

## Summary

Compute RePhine transcriptional-regulator response signatures (-Log10 uniP values) for each drug in the CCLE pharmacogenomic data and use them to hierarchically cluster drugs by mechanism of action, reproducing the Figure 3A organization.

## When to use

Use when the goal is to organize anti-cancer drugs (e.g., CCLE compounds) into mechanism-of-action groups from TR-drug response associations, or to reproduce the RePhine drug-cluster dendrogram of Figure 3A.

## When NOT to use

- ChIP-seq data for a TR is missing or contains too few targets — the article warns this would mislead TR identification, so the signature values are unreliable
- The goal is inferring acquired/secondary resistance from post-treatment dynamics — CCLE lacks post-treatment data (e.g., erlotinib resistance via acquired STAT3 activation cannot be detected)
- Input is already a curated drug-feature table not derived from TR response associations — clustering would not reflect RePhine's mechanism-of-action signature

## Inputs

- Precomputed regulatory potential (RP) scores across >1000 ENCODE ChIP-seq data sets (Google Drive resource from the RePhine repo)
- ENCODE ChIP-seq target sets for 160 transcriptional regulators
- CCLE pharmacological profiles (drug response across cancer cell lines)
- CCLE gene expression and copy number data
- CCLE mutation data

## Outputs

- 160-TR by N-drug matrix of -Log10(uniP) RePhine response signatures
- Hierarchical clustering dendrogram of CCLE drugs
- Drug-to-cluster mapping table (drug_clusters.csv)

## How to apply

Load the precomputed regulatory potential (RP) scores for the CCLE drugs and the ENCODE ChIP-seq target sets for 160 transcriptional regulators from the RePhine repository resources. Run the RePhine regression (R package) for each TR-drug pair to obtain uniP statistics, and transform each into -Log10(uniP) to build a 160-TR by N-drug signature matrix; the regression itself adjusts for copy number effects, mutation-related confounding factors (via adaptive lasso and likelihood ratio test), and computes partial correlations between targets and drug response. Then apply hierarchical clustering on the drugs from this matrix in R's hclust using the ward.D method, cut the dendrogram, and export the drug-to-cluster mapping to drug_clusters.csv. Validate against Figure 3A: chemotherapy drugs plus panobinostat should co-cluster (cluster 1), EGFR inhibitors (erlotinib, AZD0530, lapatinib, ZD-6474) with 17-AAG (cluster 3), and MEK/RAF/MET inhibitors PD-0325901, AZD6244, PLX4720, RAF265, PHA-665752 together (cluster 2); if assignments mismatch, revisit clustering parameters or the signature matrix.

## Related tools

- **RePhine** (R package that regresses TR ChIP-seq targets against drug response to produce uniP statistics used to build the -Log10(uniP) signature matrix) — https://github.com/coexps/RePhine
- **R** (Language in which RePhine is implemented and in which hclust (ward.D) hierarchical clustering of drugs is performed)
- **Python** (Runs the modified RP score calculation script (RP_calculation_modified.py) producing regulatory potential scores used as RePhine input) — https://github.com/coexps/RePhine

## Examples

```
hc <- hclust(dist(sig_matrix), method="ward.D"); drug_clusters <- cutree(hc, k=3); write.csv(data.frame(drug=names(drug_clusters), cluster=drug_clusters), "drug_clusters.csv")
```

## Evaluation signals

- The named drug-cluster assignments match Figure 3A: MEK inhibitors PD-0325901 and AZD6244, RAF inhibitors PLX4720 and RAF265, and MET inhibitor PHA-665752 all co-cluster (cluster 2, a MAPK-related group)
- Chemotherapy drugs plus panobinostat co-cluster (cluster 1) and EGFR inhibitors (erlotinib, AZD0530, lapatinib, ZD-6474) group with 17-AAG (cluster 3)
- The signature matrix has dimensions 160 TRs by N drugs, one -Log10(uniP) value per TR-drug pair
- RePhine dependencies resolve: R packages glmnet, MASS, lmtest, ppcor, psych, ROCR installed and the pipeline runs on CCLE pharmacology + ENCODE RP-score inputs
- drug_clusters.csv is produced and each CCLE drug in the validation set receives exactly one cluster label

## Limitations

- RePhine-derived associations may not be causal: upstream regulators or kinases may influence drug response while simultaneously regulating downstream TRs; TR knockout/activation experiments are needed to validate causality
- Lack of ChIP-seq data or ChIP-seq data with too few targets would mislead TR identification
- Detection of acquired resistance is limited due to lack of post-treatment data in CCLE, so drug-perturbation effects on genes cannot be integrated
- The method is restricted to cases where reliable target information is available

## Evidence

- [results] RePhine response-signature clustering placed the MEK inhibitors PD-0325901 and AZD6244, RAF inhibitors PLX4720 and RAF265, and the MET inhibitor PHA-665752 together in cluster 2.2, a MAPK-related group.: "we observed that RePhine-derived TR signatures could effectively cluster drugs with different mechanisms of action"
- [discussion] RePhine is available as an R package with accompanying guide and repository.: "RePhine, which is implemented as an R package and accompanied by a user guide, is available at https://github.com/coexps/RePhine."
- [readme] uniP is computed via likelihood ratio test over concordantly correlated targets.: "Use likelihood ratio test to evaluate significance (uniP) of the TRs whose targets have concordant positive or negative partial correlations with drug response"
- [readme] Partial correlations between targets and drug response are computed controlling confounding factors.: "Compute the partial correlation between a specific gene and a drug by controlling the confounding factors."
- [readme] ENCODE RP scores used in the analysis are distributed via the repository's Google Drive resource.: [evidence span withheld — anchor status: external_doc]
