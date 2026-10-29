---
name: pancancer-tumor-type-structure-visualization
description: Use when you have CITRUS output (output_dataset_CITRUS.pkl with 'hid_tmr',
  'tmr', 'can') for 17 TCGA tumor types and want to cluster tumors, render tumor-grouping
  heatmaps, and test which TFs define each cancer type.
when_to_use_negative:
- You only have raw SGA/GEP inputs (dataset_CITRUS.pkl) and no trained CITRUS model
  output — TF activities ('hid_tmr') do not exist yet; train/run run_CITRUS.py first.
- You need de novo TF activity inference from expression data alone without CITRUS's
  chromatin-informed TF-target priors and somatic alteration context.
- You have a single tumor type only — the clustering/association structure this skill
  visualizes depends on joint modeling across the 17 TCGA tumor types.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: CITRUS
  role: Produces the patient-specific TF activity matrix ('hid_tmr') whose clustering
    structure is visualized
  repo: https://github.com/osmanbeyoglulab/CITRUS
- name: ensemble_TF.py
  role: Aggregates TF activities from 10 CITRUS runs for stable clustering input
  repo: https://github.com/osmanbeyoglulab/CITRUS
- name: scikit-learn
  role: K-means / hierarchical clustering of the patient-by-TF activity matrix across
    17 tumor types
- name: SciPy
  role: Two-sample t-tests per TF comparing activity within each cancer type vs the
    remaining tumors
- name: Pandas
  role: Assembles TF-by-tumor-type t-statistics and p-values into tf_tumor_type_associations.csv
- name: Numpy
  role: Array handling for the TF activity matrix and test statistics
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkac881
    title: Interpretable deep learning for chromatin-informed inference of transcriptional
      programs driven by somatic alterations across cancers
  source_content_hash: sha256:75902b52f954895b8da9473ab5bea027d27235eb212e1424c253c259c109eab4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/osmanbeyoglulab/CITRUS
  tools:
  - CITRUS
  - Numpy
  - Pandas
  - Python
  - SciPy
  - ensemble_TF.py
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# pancancer-tumor-type-structure-visualization

## Summary

Clusters TCGA tumors across 17 cancer types using CITRUS-inferred, patient-specific transcription factor activities ('hid_tmr' hidden-layer values) to visualize whether pan-cancer TF activity structure recovers known tumor-type groupings, and quantifies TF–tumor-type associations with per-TF two-sample t-tests.

## When to use

Use when you have CITRUS output (output_dataset_CITRUS.pkl with 'hid_tmr', 'tmr', 'can') for 17 TCGA tumor types and want to cluster tumors, render tumor-grouping heatmaps, and test which TFs define each cancer type.

## When NOT to use

- You only have raw SGA/GEP inputs (dataset_CITRUS.pkl) and no trained CITRUS model output — TF activities ('hid_tmr') do not exist yet; train/run run_CITRUS.py first.
- You need de novo TF activity inference from expression data alone without CITRUS's chromatin-informed TF-target priors and somatic alteration context.
- You have a single tumor type only — the clustering/association structure this skill visualizes depends on joint modeling across the 17 TCGA tumor types.

## Inputs

- output_dataset_CITRUS.pkl (pickle dictionary containing 'hid_tmr' TF activities, 'tmr' tumor barcodes, 'can' cancer type labels)
- ensemble TF activities aggregated from 10 CITRUS runs (ensemble_TF.py --runs 10)
- 17 TCGA tumor types: 5803 whitelisted Pan-Cancer samples with parallel RNA-seq, somatic mutation, and GISTIC copy number data

## Outputs

- tf_activity_cluster_figure.png (cluster/heatmap of tumor groupings: squamous-morphology, renal, gastrointestinal, neuroendocrine clusters)
- tf_tumor_type_associations.csv (TF, cancer_type, t_statistic, p_value columns)

## How to apply

Load the trained-model pickle (output_dataset_CITRUS.pkl) with Python's pickle and extract the patient-by-TF activity matrix from 'hid_tmr' together with tumor barcodes ('tmr') and cancer type labels ('can'); optionally first run CITRUS 10 times and aggregate with `python ensemble_TF.py --runs 10` to stabilize TF activities, which are the hidden layer before the output gene expression layer. Cluster the 5803-sample, 17-tumor-type activity matrix with scikit-learn (k-means or hierarchical clustering), then render a cluster/heatmap figure (tf_activity_cluster_figure.png) checking that squamous-morphology samples (BLCA, CESC, ESCA, HNSC, LUSC) group together and that tissue-proximity pairs (KIRC/KIRP, COAD/STAD, BRCA/UCEC) co-cluster. For each TF, use SciPy two-sample t-tests comparing its activity within each cancer type versus all remaining tumors, assembling t-statistics and p-values into a TF-by-tumor-type pandas table saved as tf_tumor_type_associations.csv, analogous to Figure 3 / Supplementary Data 1. Use the whitelisted TCGA Pan-Cancer samples (5803 samples, 17 types) with parallel RNA-seq, somatic mutation, and GISTIC copy number data as in the paper.

## Related tools

- **CITRUS** (Produces the patient-specific TF activity matrix ('hid_tmr') whose clustering structure is visualized) — https://github.com/osmanbeyoglulab/CITRUS
- **ensemble_TF.py** (Aggregates TF activities from 10 CITRUS runs for stable clustering input) — https://github.com/osmanbeyoglulab/CITRUS
- **scikit-learn** (K-means / hierarchical clustering of the patient-by-TF activity matrix across 17 tumor types)
- **SciPy** (Two-sample t-tests per TF comparing activity within each cancer type vs the remaining tumors)
- **Pandas** (Assembles TF-by-tumor-type t-statistics and p-values into tf_tumor_type_associations.csv)
- **Numpy** (Array handling for the TF activity matrix and test statistics)

## Examples

```
python ensemble_TF.py --runs 10
```

## Evaluation signals

- tf_tumor_type_associations.csv exists with TF, cancer_type, t_statistic, and p_value columns, one row per TF–tumor-type pair
- tf_activity_cluster_figure.png reproduces expected groupings: squamous-morphology samples (BLCA, CESC, ESCA, HNSC, LUSC) cluster together
- Tissue-proximity pairs co-cluster: KIRC/KIRP, COAD/STAD, and BRCA/UCEC
- Row/column counts match the input: tumor barcodes in 'tmr' align 1:1 with rows of 'hid_tmr' and labels in 'can' (17 types)
- If using ensemble mode, exactly 10 run outputs were aggregated via ensemble_TF.py --runs 10 before clustering

## Limitations

- TFs of the same family often share a similar motif, so inferred 'TF activities' may encompass the activities of multiple TFs and cannot disambiguate family members.
- Co-binding TF binding patterns (e.g. AP-1–IRF complexes) are not represented in the model, so clusters may reflect composite activities.
- No directionality (activation vs repression) is represented in the TF–target gene priors used to define 'hid_tmr'.
- Tumor regulatory signals are confounded by stromal/immune cells within the tumor and heterogeneity of cancer cells, affecting cluster interpretation.
- Clustering relies on TF activities from a hidden layer, which the authors stabilize only via a 10-run ensemble; single-run activities may be less stable.

## Evidence

- [abstract] CITRUS predicts patient-specific TF activities and reveals transcriptional program variations between and within tumor types.: "CITRUS predicts patient-specific TF activities and reveals transcriptional program variations between and within tumor types."
- [readme] Ensemble TF activity as hidden layer: "We use the ensemble method to further stabilize TF activity which is a hidden layer before the output gene expression layer."
- [methods] Run model 10 times for ensemble: "To generate ensembled TF activity, First run the model 10 times"
- [readme] Output dictionary keys: ""hid_tmr": hid_tmr,       # TF activity"
- [discussion] Joint modeling patient subgroups: "Joint modeling across different tumor types also revealed patient subgroups associated with somatic alterations."
- [discussion] TF motif family ambiguity: "TFs of the same family often share a similar motif and thus are difficult to disambiguate."
