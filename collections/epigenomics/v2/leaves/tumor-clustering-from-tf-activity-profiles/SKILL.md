---
name: tumor-clustering-from-tf-activity-profiles
description: Use when you have a CITRUS output pickle with patient-by-TF activity
  matrices ('hid_tmr', 'tmr', 'can') and want to check whether inferred TF activities
  recover known tumor-type distinctions or identify TF–tumor-type associations.
when_to_use_negative:
- Your input is raw somatic alteration or gene expression matrices rather than a trained
  CITRUS output pickle — you must train CITRUS (python run_CITRUS.py) first to obtain
  'hid_tmr'.
- Your goal is disambiguating individual TFs that share similar motifs — CITRUS motifs
  may encompass the activities of multiple TFs of the same family.
- You need directionality (activation vs repression) of TF–target effects — the TF–target
  gene priors used by CITRUS do not represent directionality.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3173
tools:
- name: CITRUS
  role: Produces the patient-specific TF activity matrix ('hid_tmr') and tumor/cancer-type
    lists used for clustering; source code and ensembled TF activities from the paper
  repo: https://github.com/osmanbeyoglulab/CITRUS
- name: scikit-learn
  role: K-means or hierarchical clustering of the patient-by-TF activity matrix across
    17 TCGA tumor types
- name: SciPy
  role: Two-sample t-tests comparing each TF's activity within a cancer type versus
    remaining tumors
- name: pandas
  role: Assembling t-statistics and p-values into the TF-by-tumor-type association
    CSV
- name: PyTorch
  role: Framework for the CITRUS encoder-decoder model whose hidden layer yields TF
    activities; run 10 times for ensembling
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkac881
    title: Interpretable deep learning for chromatin-informed inference of transcriptional
      programs driven by somatic alterations across cancers
  source_content_hash: sha256:ceef798d34e88396b892489a69d2004a8a30080298f9b8b67e3ff03ae5e9a0f5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/osmanbeyoglulab/CITRUS
  tools:
  - CITRUS
  - Numpy
  - Pandas
  - PyTorch
  - Python
  - SciPy
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42105263157894735
n_claims: 19
---

# tumor-clustering-from-tf-activity-profiles

## Summary

Clusters TCGA tumors (17 cancer types, 5803 samples) using patient-specific transcription factor activities inferred by CITRUS ('hid_tmr' layer), recovering known tumor-type groupings such as squamous-morphology and tissue-proximity clusters. Companion t-tests link each TF's activity to individual tumor types.

## When to use

Use when you have a CITRUS output pickle with patient-by-TF activity matrices ('hid_tmr', 'tmr', 'can') and want to check whether inferred TF activities recover known tumor-type distinctions or identify TF–tumor-type associations.

## When NOT to use

- Your input is raw somatic alteration or gene expression matrices rather than a trained CITRUS output pickle — you must train CITRUS (python run_CITRUS.py) first to obtain 'hid_tmr'.
- Your goal is disambiguating individual TFs that share similar motifs — CITRUS motifs may encompass the activities of multiple TFs of the same family.
- You need directionality (activation vs repression) of TF–target effects — the TF–target gene priors used by CITRUS do not represent directionality.

## Inputs

- output_dataset_CITRUS.pkl (CITRUS output pickle with 'hid_tmr' TF activities, 'tmr' tumor barcodes, 'can' cancer type labels)
- dataset_CITRUS.pkl input pickle (SGA index lists, GEP of 5541 filtered genes, TF-target gene matrix) if re-running the model
- Optionally: per-run CITRUS outputs from 10 training runs for ensembling

## Outputs

- tf_activity_cluster_figure.png (cluster/heatmap of tumor groupings: squamous, renal, gastrointestinal, neuroendocrine clusters)
- tf_tumor_type_associations.csv (TF, cancer_type, t_statistic, p_value columns)

## How to apply

Load output_dataset_CITRUS.pkl with pickle and extract the TF activity matrix ('hid_tmr'), tumor barcodes ('tmr'), and cancer type labels ('can'); optionally aggregate TF activities over 10 runs using ensemble_TF.py --runs 10 for stability, since TF activity is a hidden layer before the output gene expression layer. Cluster the patient-by-TF activity matrix across the 17 TCGA tumor types with scikit-learn (k-means or hierarchical clustering) and render a cluster/heatmap figure showing the recovered groupings: squamous-morphology samples (BLCA, CESC, ESCA, HNSC, LUSC), renal (KIRC/KIRP), gastrointestinal (COAD/STAD), and neuroendocrine clusters. Then, for each TF–tumor-type pair, run a SciPy two-sample t-test comparing that TF's activity within a cancer type versus the remaining tumors. Assemble the t-statistics and p-values into a TF-by-tumor-type table with pandas, analogous to Figure 3 / Supplementary Data 1. Note that CITRUS hidden nodes explicitly represent TF states learned from TF binding motifs in open chromatin regions from TCGA ATAC-seq data, which motivates interpreting hid_tmr as patient-specific TF activity.

## Related tools

- **CITRUS** (Produces the patient-specific TF activity matrix ('hid_tmr') and tumor/cancer-type lists used for clustering; source code and ensembled TF activities from the paper) — https://github.com/osmanbeyoglulab/CITRUS
- **scikit-learn** (K-means or hierarchical clustering of the patient-by-TF activity matrix across 17 TCGA tumor types)
- **SciPy** (Two-sample t-tests comparing each TF's activity within a cancer type versus remaining tumors)
- **pandas** (Assembling t-statistics and p-values into the TF-by-tumor-type association CSV)
- **PyTorch** (Framework for the CITRUS encoder-decoder model whose hidden layer yields TF activities; run 10 times for ensembling)

## Examples

```
output_data = pickle.load(open('./data/output_dataset_CITRUS.pkl', 'rb')); TF_activity = output_data['hid_tmr']
```

## Evaluation signals

- tf_tumor_type_associations.csv exists with TF, cancer_type, t_statistic, and p_value columns, one row per TF–tumor-type pair
- tf_activity_cluster_figure.png reproduces the expected groupings: squamous-morphology samples (BLCA, CESC, ESCA, HNSC, LUSC) cluster together and tissue-proximity groups (KIRC/KIRP, COAD/STAD, BRCA/UCEC) co-cluster
- The activity matrix dimensions match the tumor list ('tmr') and cancer type list ('can') extracted from the output pickle
- If ensembling was used, TF activities are aggregated over 10 independent runs via ensemble_TF.py --runs 10
- hid_tmr values are interpretable as TF states consistent with the decoder hidden-layer design tied to TF-target gene motifs

## Limitations

- TFs of the same family often share a similar motif, so TF motifs may encompass the activities of multiple TFs and cannot disambiguate them.
- Co-binding TF binding patterns (e.g. AP-1−IRF complexes) that are biologically meaningful are not represented in the model.
- TF–target gene priors do not represent directionality (whether a gene is activated or repressed by a TF).
- Regulatory network analysis of tumor datasets is complicated by stromal/immune cells within the tumor and heterogeneity of cancer cells themselves.
- Single-run TF activities are unstable; the paper uses an ensemble over 10 runs to stabilize the hidden-layer TF activity.

## Evidence

- [abstract] CITRUS predicts patient-specific TF activities and reveals transcriptional program variations between and within tumor types.: "CITRUS predicts patient-specific TF activities and reveals transcriptional program variations between and within tumor types."
- [abstract] hidden node layer representing TF states learned from motifs: "CITRUS uses a layer of hidden nodes to explicitly represent the state of transcription factors (TFs) to learn the relationships between TFs and their target genes based on TF binding motifs"
- [readme] output dictionary contents including hid_tmr: ""hid_tmr": hid_tmr,       # TF activity"
- [readme] ensemble over 10 runs: "To generate ensembled TF activity, First run the model 10 times"
- [readme] 5803 samples, 17 tumor types: "They included 5803 samples with 17 different tumor types from TCGA."
- [discussion] TF motif ambiguity limitation: "TFs of the same family often share a similar motif and thus are difficult to disambiguate. Therefore, TF motifs may encompass the activities of multiple TFs."
- [discussion] no directionality in priors: "we do not represent directionality in the TF–target gene priors (i.e. whether a gene is activated or repressed by a TF)"
- [discussion] joint modeling revealed patient subgroups: "Joint modeling across different tumor types also revealed patient subgroups associated with somatic alterations."
- [methods] packages used: "some other packages such as PyTorch, Pandas, Numpy, scikit-learn, Scipy are used"
