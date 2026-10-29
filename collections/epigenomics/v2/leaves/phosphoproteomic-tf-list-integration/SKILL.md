---
name: phosphoproteomic-tf-list-integration
description: Use when CITRUS ensemble TF activities (hid_tmr) must be validated against
  independent TF lists, e.g. MCF10A isogenic PIK3CA knock-in phosphoproteomics or
  AKT1 kinase assay substrate TFs.
when_to_use_negative:
- Input is raw CITRUS per-run output rather than the ensembled TF activities — first
  run the model 10 times and aggregate with ensemble_TF.py --runs 10
- The external reference set is not a TF list (e.g., gene-level or phosphosite-level
  data), since overlap is defined at the transcription factor level
- The somatic alteration of interest is not represented in the SGA matrix used to
  train the model, so no inferred TF activities can be associated with it
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3512
tools:
- name: CITRUS
  role: Source of the ensembled TF activity matrix (hid_tmr) being externally validated
  repo: https://github.com/osmanbeyoglulab/CITRUS
- name: Python
  role: Execution environment for the association and overlap analysis (python 3.7
    and above)
- name: Pandas
  role: Loading and manipulating the pickled CITRUS output and supplementary TF lists
- name: SciPy
  role: Statistical association testing of TF activity vs PIK3CA mutation status and
    Fisher's exact test for list overlap
- name: Numpy
  role: Array operations on the TF activity matrix and mutation vector
- name: scikit-learn
  role: Supporting package for the analysis environment as listed in the paper methods
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkac881
    title: Interpretable deep learning for chromatin-informed inference of transcriptional
      programs driven by somatic alterations across cancers
  source_content_hash: sha256:96bc2db2b7c97e1ede76312acfe8af550d72287fd44b0a90dc55195db32536ef
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
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# phosphoproteomic-tf-list-integration

## Summary

Externally validates model-inferred transcription factor activities by testing their association with a somatic alteration (e.g., PIK3CA mutation) across TCGA samples and quantifying overlap with independent TF lists from phosphoproteomic and kinase-assay experiments.

## When to use

Use when CITRUS ensemble TF activities (hid_tmr) must be validated against independent TF lists, e.g. MCF10A isogenic PIK3CA knock-in phosphoproteomics or AKT1 kinase assay substrate TFs.

## When NOT to use

- Input is raw CITRUS per-run output rather than the ensembled TF activities — first run the model 10 times and aggregate with ensemble_TF.py --runs 10
- The external reference set is not a TF list (e.g., gene-level or phosphosite-level data), since overlap is defined at the transcription factor level
- The somatic alteration of interest is not represented in the SGA matrix used to train the model, so no inferred TF activities can be associated with it

## Inputs

- output_dataset_CITRUS.pkl: pickled dictionary of the 10-run ensemble CITRUS output containing 'hid_tmr' (ensembled TF activities) and 'tmr' (tumor list)
- PIK3CA somatic mutation status per sample for the 5803 TCGA samples (from TCGA Firehose somatic mutation data)
- Published MCF10A isogenic PIK3CA E545K/H1047R knock-in phosphoproteomic TF list (20 TFs) from supplementary tables
- AKT1 kinase assay substrate TF list (7 TFs) from supplementary tables

## Outputs

- List of CITRUS TFs significantly associated with PIK3CA mutation (FDR-adjusted P < 0.05)
- overlap_statistics.csv with overlap counts (16/20 phosphoproteomic TFs; 6/7 AKT-substrate TFs) and Fisher's exact test enrichment statistics

## How to apply

Load the ensembled TF activity matrix ('hid_tmr') from output_dataset_CITRUS.pkl via pickle, which corresponds to the 5803-sample, 17-tumor-type TCGA cohort. For each TF, test association of its inferred activity with PIK3CA somatic mutation status using a SciPy statistical test, apply Benjamini-Hochberg FDR correction, and retain TFs at FDR-adjusted P < 0.05. Load the external reference lists: the published MCF10A isogenic PIK3CA E545K/H1047R knock-in phosphoproteomic TF list (20 TFs) and the AKT1 kinase assay substrate TF list (7 TFs) from the supplementary tables. Compute the overlap of FDR-significant PIK3CA-associated CITRUS TFs against each external list and quantify enrichment with Fisher's exact test (SciPy), writing counts and statistics to overlap_statistics.csv. Correct application is confirmed when reproduced overlap counts match the published 16/20 phosphoproteomic TFs and 6/7 AKT-substrate TFs.

## Related tools

- **CITRUS** (Source of the ensembled TF activity matrix (hid_tmr) being externally validated) — https://github.com/osmanbeyoglulab/CITRUS
- **Python** (Execution environment for the association and overlap analysis (python 3.7 and above))
- **Pandas** (Loading and manipulating the pickled CITRUS output and supplementary TF lists)
- **SciPy** (Statistical association testing of TF activity vs PIK3CA mutation status and Fisher's exact test for list overlap)
- **Numpy** (Array operations on the TF activity matrix and mutation vector)
- **scikit-learn** (Supporting package for the analysis environment as listed in the paper methods)

## Examples

```
output_data = pickle.load(open("./data/output_dataset_CITRUS.pkl", "rb")); TF_activity = output_data["hid_tmr"]
```

## Evaluation signals

- Reproduced overlap of FDR-significant PIK3CA-associated CITRUS TFs with the phosphoproteomic list equals 16 of 20 TFs
- Reproduced overlap with the AKT1 kinase assay substrate list equals 6 of 7 TFs
- All retained TFs have Benjamini-Hochberg FDR-adjusted P-value < 0.05
- hid_tmr extracted from the pickle has one row per TCGA tumor sample (5803 samples, 17 tumor types)
- overlap_statistics.csv contains both overlap counts and Fisher's exact test enrichment statistics for both comparisons

## Limitations

- TFs of the same family often share a similar motif and thus are difficult to disambiguate, so TF motifs may encompass the activities of multiple TFs
- TF-target gene priors do not represent directionality (activation vs repression)
- Regulatory network analysis of tumor datasets is complicated by stromal/immune cells within the tumor and heterogeneity of cancer cells
- Overlap validation depends on TFs being represented in the external phosphoproteomic/kinase lists; TFs absent from those lists cannot be validated by this approach

## Evidence

- [other] 16 of 20 TFs represented in the phosphoproteomic data associated with mutant PIK3CA were associated with PIK3CA mutation, and 6 of 7 AKT-substrate TFs were associated with PIK3CA mutation (FDR-adjusted P-value < 0.05): [evidence span withheld — anchor status: not_found]
- [methods] They included 5803 samples with 17 different tumor types from TCGA.: "They included 5803 samples with 17 different tumor types from TCGA."
- [methods] some other packages such as PyTorch, Pandas, Numpy, scikit-learn, Scipy are used: "some other packages such as PyTorch, Pandas, Numpy, scikit-learn, Scipy are used"
- [readme] TF activity is available from the output pickle and can be extracted as shown in the README.: "output_data = pickle.load(open("./data/output_dataset_CITRUS.pkl", "rb"))"
- [readme] To generate ensembled TF activity, First run the model 10 times: "To generate ensembled TF activity, First run the model 10 times"
