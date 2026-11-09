---
name: in-silico-gene-knockout-perturbation
description: Use when you have a trained CITRUS model output (hid_tmr TF activities),
  a binary SGA alteration matrix, and want to predict which TFs are associated with
  specific somatic alterations per cancer type.
when_to_use_negative:
- You lack a trained CITRUS model or its output pickle files (dataset_CITRUS.pkl /
  output_dataset_CITRUS.pkl); the knock out requires propagating perturbed SGA through
  this specific model
- Your goal is differential expression between mutant and wild-type tumors from raw
  data rather than model-inferred causal TF activity associations
- TFs of interest belong to the same motif family that CITRUS cannot disambiguate,
  or you need TF activation/repression directionality or co-binding (e.g. AP-1-IRF
  complexes) represented
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CITRUS
  role: Trained chromatin-informed neural network whose SGA input is perturbed (gene
    set to 0) to generate knockout TF activities
  repo: https://github.com/osmanbeyoglulab/CITRUS
- name: SciPy
  role: Two-sample t-tests comparing carrier versus non-carrier TF activity distributions
- name: statsmodels
  role: Benjamini-Hochberg FDR correction of TF-level p-values across the multiple-testing
    burden
- name: Pandas
  role: Loading pickle files and compiling significant TF-alteration associations
    into tf_alteration_associations.csv
- name: PyTorch
  role: Framework the trained CITRUS model runs under when propagating perturbed SGA
    inputs
  repo: https://github.com/osmanbeyoglulab/CITRUS
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkac881
    title: Interpretable deep learning for chromatin-informed inference of transcriptional
      programs driven by somatic alterations across cancers
  source_content_hash: sha256:8f1a5b48dbe8385fc3499d3d1875444a180fd03977a8f35db90c0d4ad7143578
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
  - statsmodels
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# in-silico-gene-knockout-perturbation

## Summary

Systematic in silico knock out analysis that zeroes a somatic alteration (SGA) gene in carrier tumors and propagates the perturbed input through the trained CITRUS model to obtain knockout TF activities, then tests mutant versus wild-type TF activity differences to identify TFs causally associated with alterations such as PIK3CA, PTEN, KRAS, TP53 and CTNNB1.

## When to use

Use when you have a trained CITRUS model output (hid_tmr TF activities), a binary SGA alteration matrix, and want to predict which TFs are associated with specific somatic alterations per cancer type.

## When NOT to use

- You lack a trained CITRUS model or its output pickle files (dataset_CITRUS.pkl / output_dataset_CITRUS.pkl); the knock out requires propagating perturbed SGA through this specific model
- Your goal is differential expression between mutant and wild-type tumors from raw data rather than model-inferred causal TF activity associations
- TFs of interest belong to the same motif family that CITRUS cannot disambiguate, or you need TF activation/repression directionality or co-binding (e.g. AP-1-IRF complexes) represented

## Inputs

- output_dataset_CITRUS.pkl containing the ensemble TF activity matrix ('hid_tmr'), tumor list ('tmr'), and cancer type labels ('can')
- dataset_CITRUS.pkl containing the binary SGA alteration matrix (11998 genes; 1 = mutation or copy number alteration, 0 = none)
- List of alteration genes of interest per cancer type (UCEC: PIK3CA, PTEN, KRAS, TP53, CTNNB1; BRCA: PIK3CA, TP53, GATA3, CDH1)
- Trained CITRUS model (PyTorch) capable of generating knockout TF activities from perturbed SGA input

## Outputs

- tf_alteration_associations.csv with significant TF-alteration pairs per alteration gene and cancer type (FDR-adjusted)
- Per-TF t-test p-values and Benjamini-Hochberg FDR-adjusted p-values for carrier vs non-carrier comparisons
- Knockout TF activity matrices obtained by propagating perturbed SGA through CITRUS

## How to apply

Load the ensemble TF activity matrix ('hid_tmr'), tumor barcode list, and cancer type labels from output_dataset_CITRUS.pkl, and the binary SGA matrix (11998 genes) from dataset_CITRUS.pkl with pandas/pickle. Subset samples to the cancer type of interest (e.g. UCEC and BRCA) and select the alteration genes to knock out (PIK3CA, PTEN, KRAS, TP53, CTNNB1 for UCEC; PIK3CA, TP53, GATA3, CDH1 for BRCA). Implement the knock out by setting the alteration gene to 0 (removing the mutation or copy number alteration) in all carrier tumors and propagating the perturbed SGA through the trained CITRUS model to obtain knockout TF activities. For each TF, compare carrier versus non-carrier ('wild type') TF activity distributions with two-sample t-tests from SciPy, then correct p-values across TFs using Benjamini-Hochberg FDR. Compile significant TF-alteration associations per alteration gene and cancer type into tf_alteration_associations.csv and compare against the paper's Figure 5A-B.

## Related tools

- **CITRUS** (Trained chromatin-informed neural network whose SGA input is perturbed (gene set to 0) to generate knockout TF activities) — https://github.com/osmanbeyoglulab/CITRUS
- **SciPy** (Two-sample t-tests comparing carrier versus non-carrier TF activity distributions)
- **statsmodels** (Benjamini-Hochberg FDR correction of TF-level p-values across the multiple-testing burden)
- **Pandas** (Loading pickle files and compiling significant TF-alteration associations into tf_alteration_associations.csv)
- **PyTorch** (Framework the trained CITRUS model runs under when propagating perturbed SGA inputs) — https://github.com/osmanbeyoglulab/CITRUS

## Examples

```
output_data = pickle.load(open('./data/output_dataset_CITRUS.pkl', 'rb')); TF_activity = output_data['hid_tmr']; tumor_list = output_data['tmr']
```

## Evaluation signals

- Significant (FDR-adjusted) TF-alteration pairs reproduce the reported associations in Figure 5A-B: PTEN mutations associate with cell cycle/differentiation TFs including E2F5, TP63, ELF3, DBP, ZKSCAN3, LHX2, HOXB6, SOX9 and MYBL1; CTNNB1-mutant status associates with WNT/TGF-beta signaling TFs TCF7, TCF7L2, TCF7L1, FOXH1, EMX1 and MYBL1 in UCEC
- BRCA associations reproduce for PIK3CA, TP53, GATA3 and CDH1 as shown in Figure 5B
- t_alteration_associations.csv compiles results per alteration gene and cancer type with FDR-adjusted significance flags
- TF activity values come from the ensemble of 10 CITRUS runs (hid_tmr), not a single training run
- Knockout input check: carrier tumors for gene g show g set to 0 in the perturbed SGA matrix before propagation

## Limitations

- TFs of the same family share similar motifs and are difficult to disambiguate, so TF motifs may encompass the activities of multiple TFs
- Co-binding TF binding patterns (e.g. AP-1-IRF complexes) are biologically meaningful but not represented in the model
- No directionality (activation vs repression) is represented in the TF-target gene priors
- Tumor regulatory network analysis is confounded by stromal/immune cells and heterogeneity of the cancer cells themselves

## Evidence

- [intro] Using a systematic in silico knock out approach, we identified key TFs associated with major somatic alterations.: "Using a systematic in silico knock out approach, we identified key TFs associated with major somatic alterations."
- [discussion] our in silico knock out analysis predicts the causal impact of somatic alterations: "our in silico knock out analysis predicts the causal impact of somatic alterations"
- [readme] SGA is originally a binary matrix containing 11998 genes with 1 as having a distinct mutation or copy number alteration and 0 as none of them.: "SGA is originally a binary matrix containing 11998 genes with 1 as having a distinct mutation or copy number alteration and 0 as none of them."
- [readme] To generate ensembled TF activity, First run the model 10 times: "To generate ensembled TF activity, First run the model 10 times"
- [discussion] TFs of the same family often share a similar motif and thus are difficult to disambiguate.: "TFs of the same family often share a similar motif and thus are difficult to disambiguate. Therefore, TF motifs may encompass the activities of multiple TFs."
