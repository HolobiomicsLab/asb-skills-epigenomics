---
name: mechanism-of-action-cluster-validation
description: Use when you need to reproduce or validate drug clustering by mechanism
  of action from RePhine -Log10(uniP) TR signature matrices built on CCLE pharmacogenomic
  data and ENCODE ChIP-seq regulatory potential scores.
when_to_use_negative:
- The goal is independent TR identification from raw data rather than clustering/validating
  existing RePhine uniP signatures (run the full RePhine pipeline steps 1-7 of the
  README instead).
- Reliable ChIP-seq target information is unavailable or the ChIP-seq data has too
  few targets, which would mislead TR identification and corrupt the signature matrix.
- The question concerns acquired/secondary drug resistance in post-treatment settings
  — the article notes CCLE lacks post-treatment data, so acquired resistance (e.g.,
  erlotinib via acquired STAT3 activation) cannot be detected.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_2640
tools:
- name: RePhine
  role: R package that regresses TR targets against CCLE drug response to yield uniP
    statistics used to build the -Log10(uniP) signature matrix
  repo: https://github.com/coexps/RePhine
- name: R
  role: Environment for RePhine and for hierarchical clustering (hclust, ward.D /
    complete linkage) and dendrogram cutting
  repo: https://github.com/coexps/RePhine
- name: Python
  role: Runs RP_calculation_modified.py to compute regulatory potential scores from
    ChIP-seq peak bed files used as input features
  repo: https://github.com/coexps/RePhine
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2019.09.008
    title: '<i>RePhine</i>: An Integrative Method for Identification of Drug Response-Related
      Transcriptional Regulators'
  source_content_hash: sha256:91a5ec2acc86fd538d253b1298b38420bcebe897759a8c3ef655109b231eba0a
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# mechanism-of-action-cluster-validation

## Summary

Hierarchically cluster CCLE drugs using RePhine-derived −Log10(uniP) transcriptional regulator (TR) response signatures and validate that drugs with similar mechanisms of action (e.g., MEK/RAF/MET MAPK-pathway inhibitors) co-cluster, reproducing the paper's Figure 3A drug organization.

## When to use

Use when you need to reproduce or validate drug clustering by mechanism of action from RePhine -Log10(uniP) TR signature matrices built on CCLE pharmacogenomic data and ENCODE ChIP-seq regulatory potential scores.

## When NOT to use

- The goal is independent TR identification from raw data rather than clustering/validating existing RePhine uniP signatures (run the full RePhine pipeline steps 1-7 of the README instead).
- Reliable ChIP-seq target information is unavailable or the ChIP-seq data has too few targets, which would mislead TR identification and corrupt the signature matrix.
- The question concerns acquired/secondary drug resistance in post-treatment settings — the article notes CCLE lacks post-treatment data, so acquired resistance (e.g., erlotinib via acquired STAT3 activation) cannot be detected.

## Inputs

- Precomputed RP scores for CCLE drugs and >1000 ENCODE ChIP-seq datasets (regulatory potential scores, Google Drive resources from the RePhine repo)
- CCLE pharmacogenomic profiles (gene expression, copy number, mutation status, cancer type, pharmacological drug response)
- ENCODE ChIP-seq target sets for the 160 transcriptional regulators
- RePhine R package (Rephine_1.1.0) and drug_code_R.R

## Outputs

- 160-TR by N-drug signature matrix of -Log10(uniP) response values
- Drug dendrogram from hclust (ward.D / complete linkage)
- drug_clusters.csv mapping each drug to its cluster assignment

## How to apply

Load the precomputed RP scores for CCLE drugs and the ENCODE ChIP-seq target sets for the 160 transcriptional regulators from the RePhine repository materials (Google Drive RP-score resources). Run the RePhine regression (R package with glmnet, MASS, lmtest, ppcor, psych, ROCR) for each TR-drug pair to obtain uniP statistics via the likelihood ratio test, then transform them into -Log10(uniP) values to form a 160-TR by N-drug signature matrix. Apply hierarchical clustering on the drugs with R's hclust using the ward.D method with complete linkage, then cut the dendrogram and export the drug-to-cluster mapping to drug_clusters.csv. Validate the result against Figure 3A: chemotherapy drugs plus panobinostat in cluster 1, EGFR inhibitors (erlotinib, AZD0530, lapatinib, ZD-6474) with 17-AAG in cluster 3, and MEK inhibitors (PD-0325901, AZD6244), RAF inhibitors (PLX4720, RAF265), and the MET inhibitor PHA-665752 together in cluster 2.2 as a MAPK-related group. Mismatches indicate the clustering parameters or signature matrix must be revisited.

## Related tools

- **RePhine** (R package that regresses TR targets against CCLE drug response to yield uniP statistics used to build the -Log10(uniP) signature matrix) — https://github.com/coexps/RePhine
- **R** (Environment for RePhine and for hierarchical clustering (hclust, ward.D / complete linkage) and dendrogram cutting) — https://github.com/coexps/RePhine
- **Python** (Runs RP_calculation_modified.py to compute regulatory potential scores from ChIP-seq peak bed files used as input features) — https://github.com/coexps/RePhine

## Examples

```
Rscript -e 'hc <- hclust(dist(t(sig_matrix), method="complete"), method="ward.D"); clusters <- cutree(hc, k=3); write.csv(data.frame(drug=names(clusters), cluster=clusters), "drug_clusters.csv")'
```

## Evaluation signals

- MEK inhibitors PD-0325901 and AZD6244, RAF inhibitors PLX4720 and RAF265, and MET inhibitor PHA-665752 all land in cluster 2.2 (the MAPK-related group), matching Figure 3A.
- Chemotherapy drugs plus panobinostat co-cluster in cluster 1, and EGFR inhibitors (erlotinib, AZD0530, lapatinib, ZD-6474) group with 17-AAG in cluster 3.
- The signature matrix has dimensions 160 TRs by N CCLE drugs with finite -Log10(uniP) values for every TR-drug pair.
- drug_clusters.csv contains a complete drug-to-cluster mapping covering all drugs in the dendrogram with no missing assignments.
- Re-running with the same ward.D / complete linkage parameters reproduces an identical dendrogram structure.

## Limitations

- Lack of ChIP-seq data or ChIP-seq data with too few targets would mislead TR identification, degrading the signature matrix.
- RePhine is restricted to cases where reliable target information is available, and detected TR-drug relationships may not be causal (upstream regulators/kinases may confound).
- Detection of acquired resistance is limited due to lack of post-treatment data in CCLE; secondary resistance to erlotinib via acquired STAT3 activation could not be detected.

## Evidence

- [abstract] Utilizing RePhine and Cancer Cell Line Encyclopedia data, we observed that RePhine-derived TR signatures could effectively cluster drugs with different mechanisms of action.: "Utilizing RePhine and Cancer Cell Line Encyclopedia data, we observed that RePhine-derived TR signatures could effectively cluster drugs with different mechanisms of action."
- [discussion] RePhine, which is implemented as an R package and accompanied by a user guide, is available at https://github.com/coexps/RePhine.: "RePhine, which is implemented as an R package and accompanied by a user guide, is available at https://github.com/coexps/RePhine."
- [readme] Regulatory potential scores across more than 1000 ChIP-seq data from Encode that are used in RePhine analysis are available on google drive: [evidence span withheld — anchor status: not_found]
- [readme] Use likelihood ratio test to evaluate significance (uniP) of the TRs whose targets have concordant positive or negative partial correlations with drug response: "Use likelihood ratio test to evaluate significance (uniP) of the TRs whose targets have concordant positive or negative partial correlations with drug response"
- [discussion] Lack of ChIP-seq data or ChIP-seq data with too few targets would mislead the identification of TRs: "Lack of ChIP-seq data or ChIP-seq data with too few targets would mislead the identification of TRs"
