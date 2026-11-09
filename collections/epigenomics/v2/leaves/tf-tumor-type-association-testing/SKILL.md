---
name: tf-tumor-type-association-testing
description: Use when you have patient-by-TF activity values from a CITRUS model output
  (e.g. 'hid_tmr' from output_dataset_CITRUS.pkl) plus tumor barcodes and cancer-type
  labels, and you need to identify which TFs are significantly activated or repressed
  in each of the 17 TCGA tumor types.
when_to_use_negative:
- You only have raw gene expression (GEP) or somatic alteration (SGA) matrices but
  no CITRUS-inferred TF activities — the t-tests require the patient-specific 'hid_tmr'
  TF activity layer, not raw inputs.
- You want TF–somatic-alteration causal links rather than TF–tumor-type associations
  — that requires the in silico knock out analysis, not per-tumor-type t-tests.
- Your tumor labels are missing or misaligned with tumor barcodes in 'tmr' — the within-type
  vs. remainder contrast requires correctly paired 'can' labels per sample.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: CITRUS
  role: Produces the patient-specific TF activity matrix ('hid_tmr') that the per-TF
    t-tests are run on
  repo: https://github.com/osmanbeyoglulab/CITRUS
- name: SciPy
  role: Runs the two-sample t-tests comparing each TF's activity within each cancer
    type versus the remaining tumors
- name: Pandas
  role: Assembles t-statistics and p-values into the TF-by-tumor-type association
    table and writes tf_tumor_type_associations.csv
- name: scikit-learn
  role: Clusters the patient-by-TF activity matrix (k-means or hierarchical) to render
    the tumor-type cluster/heatmap figure
- name: Numpy
  role: Handles the TF activity matrix arrays extracted from the output pickle
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkac881
    title: Interpretable deep learning for chromatin-informed inference of transcriptional
      programs driven by somatic alterations across cancers
  source_content_hash: sha256:8ba30f8f68fc3fd039310d76feffc66e3611a7fdc801e36ec7a07af23eba0a21
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
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# tf-tumor-type-association-testing

## Summary

Systematically associates CITRUS-inferred, patient-specific transcription factor (TF) activities with tumor types by running one SciPy two-sample t-test per TF–cancer-type pair, comparing each TF's activity within a cancer type against the remaining tumors. This converts the 'hid_tmr' TF-activity matrix into a TF-by-tumor-type association table analogous to Figure 3 / Supplementary Data 1 of the CITRUS paper.

## When to use

Use when you have patient-by-TF activity values from a CITRUS model output (e.g. 'hid_tmr' from output_dataset_CITRUS.pkl) plus tumor barcodes and cancer-type labels, and you need to identify which TFs are significantly activated or repressed in each of the 17 TCGA tumor types.

## When NOT to use

- You only have raw gene expression (GEP) or somatic alteration (SGA) matrices but no CITRUS-inferred TF activities — the t-tests require the patient-specific 'hid_tmr' TF activity layer, not raw inputs.
- You want TF–somatic-alteration causal links rather than TF–tumor-type associations — that requires the in silico knock out analysis, not per-tumor-type t-tests.
- Your tumor labels are missing or misaligned with tumor barcodes in 'tmr' — the within-type vs. remainder contrast requires correctly paired 'can' labels per sample.

## Inputs

- Trained CITRUS model output pickle (output_dataset_CITRUS.pkl) containing 'hid_tmr' (TF activity), 'tmr' (tumor barcodes), and 'can' (cancer type list)
- Ensembled TF activities from 10 CITRUS runs (optional, via ensemble_TF.py)
- Patient-by-TF activity matrix across 5803 samples / 17 TCGA tumor types

## Outputs

- tf_tumor_type_associations.csv — TF-by-tumor-type association table with TF, cancer_type, t_statistic, and p_value columns
- tf_activity_cluster_figure.png — cluster/heatmap figure of recovered tumor groupings (squamous-morphology, renal, gastrointestinal, and neuroendocrine clusters) (optional companion output)

## How to apply

Load the trained-model output pickle (output_dataset_CITRUS.pkl) with pickle and extract the TF activity matrix ('hid_tmr') together with the tumor barcode list ('tmr') and cancer type labels ('can'); for stability, aggregate TF activities over 10 model runs using 'python ensemble_TF.py --runs 10'. For each TF–tumor-type pair, use SciPy two-sample t-tests to compare that TF's activity in tumors of the given cancer type versus all remaining tumors, yielding one t-statistic and p-value per pair. Assemble the t-statistics and p-values into a TF-by-tumor-type association table with pandas and save it as tf_tumor_type_associations.csv. Optionally precede testing with clustering of the patient-by-TF activity matrix (scikit-learn k-means or hierarchical clustering) to confirm known tumor-type groupings such as squamous-morphology samples (BLCA, CESC, ESCA, HNSC, LUSC) clustering together.

## Related tools

- **CITRUS** (Produces the patient-specific TF activity matrix ('hid_tmr') that the per-TF t-tests are run on) — https://github.com/osmanbeyoglulab/CITRUS
- **SciPy** (Runs the two-sample t-tests comparing each TF's activity within each cancer type versus the remaining tumors)
- **Pandas** (Assembles t-statistics and p-values into the TF-by-tumor-type association table and writes tf_tumor_type_associations.csv)
- **scikit-learn** (Clusters the patient-by-TF activity matrix (k-means or hierarchical) to render the tumor-type cluster/heatmap figure)
- **Numpy** (Handles the TF activity matrix arrays extracted from the output pickle)

## Examples

```
python ensemble_TF.py --runs 10
```

## Evaluation signals

- tf_tumor_type_associations.csv exists and has exactly the columns TF, cancer_type, t_statistic, and p_value.
- The number of rows in the association table equals (number of TFs) × (number of tumor types), i.e. one test per TF–tumor-type pair across the 17 TCGA cancer types.
- The clustering figure (tf_activity_cluster_figure.png) reproduces the expected CITRUS tumor-type groupings: squamous-morphology samples (BLCA, CESC, ESCA, HNSC, LUSC) cluster together, and tissue-proximity groups such as KIRC/KIRP, COAD/STAD, and BRCA/UCEC co-cluster.
- If ensemble aggregation is used, TF activities come from exactly 10 runs (ensemble_TF.py --runs 10) for stabilized activity values.
- Every 'hid_tmr' row can be matched to a tumor barcode in 'tmr' and a cancer type in 'can' with no length mismatch.

## Limitations

- TF motifs cannot disambiguate TFs of the same family (shared similar motifs), so TF activities may encompass the activities of multiple TFs.
- No directionality (activation vs. repression) is represented in the TF–target gene priors, limiting causal interpretation of association results.
- Tumor regulatory network analysis is confounded by stromal/immune cells within the tumor and heterogeneity of the cancer cells themselves.
- Co-binding TF binding patterns (e.g. AP-1−IRF complexes) are biologically meaningful for gene expression but are not currently represented in the CITRUS model.
- The model was trained on 5803 TCGA samples with 17 tumor types; associations on other cohorts or cancer types are untested here.

## Evidence

- [abstract] CITRUS predicts patient-specific TF activities and reveals transcriptional program variations between and within tumor types.: "CITRUS predicts patient-specific TF activities and reveals transcriptional program variations between and within tumor types."
- [methods] For each TF, use SciPy two-sample t-tests to compare its activity within each cancer type versus the remaining tumors, one test per TF–tumor-type pair.: [evidence span withheld — anchor status: not_found]
- [readme] Ensemble TF activity over 10 runs for stability.: "python ensemble_TF.py --runs 10"
- [readme] The output pickle contains hid_tmr (TF activity), tmr (tumor list), and can (cancer type list).: ""hid_tmr": hid_tmr,       # TF activity"
- [methods] They included 5803 samples with 17 different tumor types from TCGA.: "They included 5803 samples with 17 different tumor types from TCGA."
