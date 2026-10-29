---
name: cancer-type-subset-analysis-ucec-brca
description: 'Use when reproducing Figure 5A-B TF-alteration associations: you have
  the CITRUS output ''hid_tmr'' TF activity matrix, the binary SGA matrix (11998 genes),
  and need per-cancer-type (UCEC, BRCA) associations for specific alteration genes.'
when_to_use_negative:
- The goal is training CITRUS from scratch on a new dataset rather than post-hoc TF-alteration
  association testing on precomputed ensemble TF activities
- The cancer type or alteration gene of interest is not among the analyzed set (UCEC/BRCA;
  PIK3CA, PTEN, KRAS, TP53, CTNNB1, GATA3, CDH1) — retuning or a different subset
  strategy is needed
- Input is already a table of TF-alteration associations rather than raw SGA and TF
  activity matrices
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: CITRUS
  role: Trained model used to propagate in silico knocked-out SGA profiles to obtain
    knockout TF activities (hidden layer states)
  repo: https://github.com/osmanbeyoglulab/CITRUS
- name: SciPy
  role: Two-sample t-tests comparing mutant vs wild-type TF activity distributions
- name: Pandas
  role: Loading pickle datasets and compiling tf_alteration_associations.csv
- name: Numpy
  role: Manipulating the binary SGA matrix to set knock-out genes to 0 in carrier
    tumors
- name: Python 3.7
  role: Runtime environment for the analysis scripts
  repo: https://github.com/osmanbeyoglulab/CITRUS
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkac881
    title: Interpretable deep learning for chromatin-informed inference of transcriptional
      programs driven by somatic alterations across cancers
  source_content_hash: sha256:07c6c87cbe3e0831f48a9d437c60939af3aee53929dd3b4cd0dffe90bb33b384
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/osmanbeyoglulab/CITRUS
  tools:
  - CITRUS
  - Numpy
  - Pandas
  - Python
  - Python 3.7
  - SciPy
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# cancer-type-subset-analysis-ucec-brca

## Summary

Cancer-type-restricted analysis of TF–somatic alteration associations in UCEC and BRCA using CITRUS's in silico knock out approach: subset samples and alteration genes per cancer type, knock out each alteration, and test mutant-vs-wildtype TF activity with t-tests and FDR correction.

## When to use

Use when reproducing Figure 5A-B TF-alteration associations: you have the CITRUS output 'hid_tmr' TF activity matrix, the binary SGA matrix (11998 genes), and need per-cancer-type (UCEC, BRCA) associations for specific alteration genes.

## When NOT to use

- The goal is training CITRUS from scratch on a new dataset rather than post-hoc TF-alteration association testing on precomputed ensemble TF activities
- The cancer type or alteration gene of interest is not among the analyzed set (UCEC/BRCA; PIK3CA, PTEN, KRAS, TP53, CTNNB1, GATA3, CDH1) — retuning or a different subset strategy is needed
- Input is already a table of TF-alteration associations rather than raw SGA and TF activity matrices

## Inputs

- dataset_CITRUS.pkl containing the binary SGA alteration matrix (11998 genes) and tumor barcodes
- output_dataset_CITRUS.pkl containing 'hid_tmr' ensemble TF activity matrix, 'tmr' tumor list, and 'can' cancer type labels
- List of alteration genes of interest per cancer type (UCEC: PIK3CA, PTEN, KRAS, TP53, CTNNB1; BRCA: PIK3CA, TP53, GATA3, CDH1)

## Outputs

- tf_alteration_associations.csv with significant (FDR-adjusted) TF-alteration pairs per alteration gene and cancer type
- Per-TF t-test p-values and Benjamini-Hochberg FDR-adjusted q-values for each alteration gene knock out

## How to apply

Load 'dataset_CITRUS.pkl' (binary SGA matrix, 11998 genes) and 'output_dataset_CITRUS.pkl' (ensemble TF activity 'hid_tmr', tumor barcode list 'tmr', cancer type labels 'can') via pandas/pickle. Subset samples to UCEC and BRCA using the cancer type labels, and select alteration genes of interest: PIK3CA, PTEN, KRAS, TP53, CTNNB1 for UCEC and PIK3CA, TP53, GATA3, CDH1 for BRCA. Implement each in silico knock out by setting the alteration gene to 0 (removing mutation/copy number alteration) in all carrier tumors and propagating the perturbed SGA through the trained CITRUS model to obtain knockout TF activities. For each TF, compare mutant (carrier) versus wild-type (non-carrier) TF activity distributions with two-sample t-tests from SciPy, then apply Benjamini-Hochberg FDR correction across TFs (statsmodels or SciPy). Compile significant TF-alteration pairs into tf_alteration_associations.csv and compare against Figure 5A-B.

## Related tools

- **CITRUS** (Trained model used to propagate in silico knocked-out SGA profiles to obtain knockout TF activities (hidden layer states)) — https://github.com/osmanbeyoglulab/CITRUS
- **SciPy** (Two-sample t-tests comparing mutant vs wild-type TF activity distributions)
- **Pandas** (Loading pickle datasets and compiling tf_alteration_associations.csv)
- **Numpy** (Manipulating the binary SGA matrix to set knock-out genes to 0 in carrier tumors)
- **Python 3.7** (Runtime environment for the analysis scripts) — https://github.com/osmanbeyoglulab/CITRUS

## Examples

```
data = pickle.load(open('dataset_CITRUS.pkl','rb')); out = pickle.load(open('output_dataset_CITRUS.pkl','rb')); TF_activity = out['hid_tmr']
```

## Evaluation signals

- The significant (FDR-adjusted) TF-alteration pairs reproduce the reported associations in Figure 5A-B: PTEN-mutation-associated TFs include E2F5, TP63, ELF3, DBP, ZKSCAN3, LHX2, HOXB6, SOX9, MYBL1 and GLIS1 in UCEC
- CTNNB1-mutant status in UCEC is associated with WNT/TGF-beta signaling TFs including TCF7, TCF7L2, TCF7L1, FOXH1, EMX1 and MYBL1
- Subsetting on 'can' labels yields sample counts consistent with UCEC and BRCA tumor barcodes present in the 5803-sample, 17-tumor-type TCGA dataset
- The knock-out SGA matrix differs from the original only in the selected alteration gene column, set to 0 in carrier tumors
- Output tf_alteration_associations.csv contains columns for alteration gene, cancer type, TF, raw p-value, and FDR-adjusted q-value

## Limitations

- TFs of the same family often share similar motifs, so TF motifs may encompass the activities of multiple TFs and cannot disambiguate family members
- Co-binding TF binding patterns (e.g. AP-1−IRF complexes) are not represented in the model
- No directionality (activation vs repression) is represented in TF-target gene priors
- Regulatory network analysis of tumor datasets is complicated by stromal/immune cells within the tumor and heterogeneity of cancer cells

## Evidence

- [readme] SGA is originally a binary matrix containing 11998 genes with 1 as having a distinct mutation or copy number alteration and 0 as none of them.: "SGA is originally a binary matrix containing 11998 genes with 1 as having a distinct mutation or copy number alteration and 0 as none of them."
- [intro] Using a systematic in silico knock out approach, we identified key TFs associated with major somatic alterations.: "Using a systematic in silico knock out approach, we identified key TFs associated with major somatic alterations."
- [discussion] our in silico knock out analysis predicts the causal impact of somatic alterations: "our in silico knock out analysis predicts the causal impact of somatic alterations"
- [discussion] TFs of the same family often share a similar motif and thus are difficult to disambiguate. Therefore, TF motifs may encompass the activities of multiple TFs.: "TFs of the same family often share a similar motif and thus are difficult to disambiguate. Therefore, TF motifs may encompass the activities of multiple TFs."
- [methods] They included 5803 samples with 17 different tumor types from TCGA.: "They included 5803 samples with 17 different tumor types from TCGA."
