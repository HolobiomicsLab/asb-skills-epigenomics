---
name: chipseq-target-set-integration
description: Use when pharmacogenomic drug sensitivity data (e.g. CCLE) must be linked
  to ChIP-seq-defined TR target sets to infer drug-response-associated transcriptional
  regulators or to organize drugs into mechanism-of-action clusters.
when_to_use_negative:
- ChIP-seq data are missing or target sets contain too few targets, which would mislead
  TR identification
- Post-treatment data are needed (e.g. detection of acquired resistance such as erlotinib/STAT3
  activation) — CCLE lacks post-treatment data so these associations cannot be detected
- Reliable TR target information is unavailable; RePhine identifies response-related
  TRs through targets and is restricted to cases where such information exists
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3369
- http://edamontology.org/topic_3308
tools:
- name: RePhine
  role: R package whose regression over ChIP-seq RP-score target sets produces per-TR-drug
    uniP statistics used to build the drug-clustering signature matrix
  repo: https://github.com/coexps/RePhine
- name: R
  role: Runtime for the Rephine package and for hclust (ward.D) hierarchical clustering
    of the −Log10(uniP) drug signature matrix
  repo: https://github.com/coexps/RePhine
- name: Python
  role: Runs RP_calculation_modified.py, which computes RP scores from ChIP-seq peak
    BED files considering both distance and peak strength
  repo: https://github.com/coexps/RePhine
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2019.09.008
    title: '<i>RePhine</i>: An Integrative Method for Identification of Drug Response-Related
      Transcriptional Regulators'
  source_content_hash: sha256:c549fae7506fa53bfe3fc8aadf03deaebe550c28cd9cf1b0b2762a2884837ac2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/coexps/RePhine
  tools:
  - Python
  - R
  - RePhine
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3369
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# chipseq-target-set-integration

## Summary

Integrate ENCODE ChIP-seq regulatory potential (RP) scores for 160 transcriptional regulators with CCLE pharmacogenomic drug response data using the RePhine regression, then transform uniP statistics into −Log10(uniP) TR response signatures to cluster drugs by mechanism of action.

## When to use

Use when pharmacogenomic drug sensitivity data (e.g. CCLE) must be linked to ChIP-seq-defined TR target sets to infer drug-response-associated transcriptional regulators or to organize drugs into mechanism-of-action clusters.

## When NOT to use

- ChIP-seq data are missing or target sets contain too few targets, which would mislead TR identification
- Post-treatment data are needed (e.g. detection of acquired resistance such as erlotinib/STAT3 activation) — CCLE lacks post-treatment data so these associations cannot be detected
- Reliable TR target information is unavailable; RePhine identifies response-related TRs through targets and is restricted to cases where such information exists

## Inputs

- Precomputed regulatory potential (RP) scores across >1000 ENCODE ChIP-seq datasets (from the RePhine Google Drive resource)
- CCLE pharmacogenomic data: gene expression, copy number, mutation status, cancer type annotations, and pharmacological profiles
- RePhine User Guide (RePhine User Guide.pdf) and drug_code_R.R pipeline script

## Outputs

- 160-TR by N-drug −Log10(uniP) response signature matrix
- Hierarchical clustering dendrogram of CCLE drugs (ward.D) and cut dendrogram cluster assignments
- Drug-to-cluster mapping table (e.g. drug_clusters.csv) matching the Figure 3A mechanism-of-action organization

## How to apply

Load the precomputed RP scores for the CCLE drugs and the ENCODE ChIP-seq target sets for the 160 transcriptional regulators from the RePhine repository (Google Drive resource). Run the RePhine regression (R package Rephine_1.1.0, with R packages glmnet, MASS, lmtest, ppcor, psych, ROCR) for each TR-drug pair to obtain the likelihood-ratio-test uniP statistic, and transform to −Log10(uniP) to build a 160-TR by N-drug signature matrix; RePhine internally removes copy-number effects by linear regression, identifies drug-related mutations via adaptive lasso, computes partial correlations controlling confounding factors, and evaluates TRs whose targets show concordant positive or negative partial correlations. Apply hierarchical clustering on the drugs using this −Log10(uniP) matrix with ward.D in R's hclust, cut the dendrogram, and export the drug-to-cluster mapping (e.g. drug_clusters.csv). Validate against the published Figure 3A organization: chemotherapy drugs plus panobinostat in cluster 1, EGFR inhibitors with 17-AAG in cluster 3, and the MEK inhibitors PD-0325901/AZD6244, RAF inhibitors PLX4720/RAF265, and MET inhibitor PHA-665752 co-clustered in the MAPK-related cluster 2.2; revisit clustering parameters or the signature matrix if assignments diverge.

## Related tools

- **RePhine** (R package whose regression over ChIP-seq RP-score target sets produces per-TR-drug uniP statistics used to build the drug-clustering signature matrix) — https://github.com/coexps/RePhine
- **R** (Runtime for the Rephine package and for hclust (ward.D) hierarchical clustering of the −Log10(uniP) drug signature matrix) — https://github.com/coexps/RePhine
- **Python** (Runs RP_calculation_modified.py, which computes RP scores from ChIP-seq peak BED files considering both distance and peak strength) — https://github.com/coexps/RePhine

## Examples

```
hc <- hclust(dist(sig_matrix), method="ward.D"); drug_clusters <- cutree(hc, k=3); write.csv(data.frame(drug=names(drug_clusters), cluster=drug_clusters), "drug_clusters.csv")
```

## Evaluation signals

- Named-drug cluster assignments match Figure 3A: chemotherapy + panobinostat co-cluster (cluster 1); erlotinib, AZD0530, lapatinib, ZD-6474 group with 17-AAG (cluster 3); PD-0325901, AZD6244, PLX4720, RAF265, PHA-665752 co-cluster (cluster 2, subcluster 2.2)
- Signature matrix dimensions equal 160 transcriptional regulators × N CCLE drugs, with values being −Log10(uniP) transforms
- Each TR-drug uniP value derives from a likelihood-ratio test over targets with concordant positive or negative partial correlations (RePhine step 6)
- Input RP scores correspond to the >1000 ENCODE ChIP-seq regulatory potential scores distributed with the RePhine repository
- R session includes all prerequisites: glmnet, MASS, lmtest, ppcor, psych, ROCR

## Limitations

- Lack of ChIP-seq data or ChIP-seq data with too few targets would mislead the identification of TRs
- Detected TR-drug relationships may not be causal; upstream regulators or kinases may influence drug response while simultaneously regulating downstream TRs, requiring TR knockout/activation validation
- RePhine is restricted to cases where reliable target information is available
- Detection of acquired resistance is limited due to lack of post-treatment data in CCLE (e.g. secondary resistance to erlotinib via acquired STAT3 activation could not be detected)

## Evidence

- [abstract] we developed a regression-based pharmacogenomic and ChIP-seq data integration method (RePhine) to infer the impact of TRs on drug response: "we developed a regression-based pharmacogenomic and ChIP-seq data integration method (RePhine) to infer the impact of TRs on drug response"
- [abstract] RePhine-derived TR signatures could effectively cluster drugs with different mechanisms of action: "we observed that RePhine-derived TR signatures could effectively cluster drugs with different mechanisms of action"
- [readme] Use likelihood ratio test to evaluate significance (uniP) of the TRs whose targets have concordant positive or negative partial correlations with drug response: "Use likelihood ratio test to evaluate significance (uniP) of the TRs whose targets have concordant positive or negative partial correlations with drug response"
- [readme] Regulatory potential scores across more than 1000 ChIP-seq data from Encode that are used in RePhine analysis are available on google drive: [evidence span withheld — anchor status: external_doc]
- [discussion] Lack of ChIP-seq data or ChIP-seq data with too few targets would mislead the identification of TRs: "Lack of ChIP-seq data or ChIP-seq data with too few targets would mislead the identification of TRs"
