---
name: sgrna-sequence-epigenetic-feature-extraction
description: Use when building or interpreting sgRNA activity prediction models for
  CRISPR-mediated epigenome editing (CRISPRoff/CRISPRi/CRISPRa) and you need epigenetic
  feature matrices or SHAP-based feature importance for sgRNA design.
when_to_use_negative:
- When the analysis goal is pure sequence-only sgRNA efficiency prediction without
  epigenetic context (e.g., DeepHF-style sequence models) — this skill requires cell-type-specific
  epigenetic tracks (ATAC-seq, methylation, expression).
- When the input is already a ranked feature-importance table and no new sgRNA/epigenetic
  feature extraction is needed.
- When working with epigenome editing systems whose key determinants lie outside the
  four feature types considered (nucleosome occupancy, chromatin accessibility, DNA
  methylation, RNA expression), since incomplete epigenetic feature information limited
  EpiCas-DL performance.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: EpiCas-DL
  role: Deep learning framework whose sgRNA activity models provide the trained predictor
    whose features are extracted and interpreted
  repo: https://github.com/yangqianq/EpiCas-DL
- name: XGBoost
  role: Gradient-boosted tree algorithm trained on the sequence+epigenetic feature
    matrix whose SHAP values are computed for feature ranking
- name: TreeExplainer
  role: Interpreter that computes Tree SHAP values and mean |SHAP| feature importances
    on the training dataset
- name: pybedtools 0.9.0
  role: Used in custom Python 3.6 scripts to extract epigenetic feature values at
    sgRNA target coordinates
- name: Python 3.6
  role: Scripting environment for assembling the sgRNA sequence and epigenetic feature
    matrix
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.034
    title: '<i>EpiCas-DL</i> : Predicting sgRNA activity for CRISPR-mediated epigenome
      editing by deep learning'
  source_content_hash: sha256:22f9effb22073c369722252a019c2823512d4a4bc5caac8b992883e8c6b665e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yangqianq/EpiCas-DL
  tools:
  - EpiCas-DL
  - Python 3.6
  - TreeExplainer
  - XGBoost
  - pybedtools 0.9.0
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# sgrna-sequence-epigenetic-feature-extraction

## Summary

Assemble sgRNA sequence encodings together with four epigenetic feature types (nucleosome positioning/distance to TSS, DNA methylation, chromatin accessibility from ATAC-seq, RNA expression) for CRISPR off/i/a datasets, then rank feature contributions to sgRNA activity via Tree SHAP mean |SHAP| values from an XGBoost model.

## When to use

Use when building or interpreting sgRNA activity prediction models for CRISPR-mediated epigenome editing (CRISPRoff/CRISPRi/CRISPRa) and you need epigenetic feature matrices or SHAP-based feature importance for sgRNA design.

## When NOT to use

- When the analysis goal is pure sequence-only sgRNA efficiency prediction without epigenetic context (e.g., DeepHF-style sequence models) — this skill requires cell-type-specific epigenetic tracks (ATAC-seq, methylation, expression).
- When the input is already a ranked feature-importance table and no new sgRNA/epigenetic feature extraction is needed.
- When working with epigenome editing systems whose key determinants lie outside the four feature types considered (nucleosome occupancy, chromatin accessibility, DNA methylation, RNA expression), since incomplete epigenetic feature information limited EpiCas-DL performance.

## Inputs

- sgRNA sequence strings (20 nt spacer plus PAM context) from CRISPRoff/CRISPRi/CRISPRa screening datasets
- epigenetic feature matrix: nucleosome positioning (distance to TSS), chromatin accessibility (ATAC-seq from ATACdb), DNA methylation, RNA expression, per target site
- genomic coordinates (chrom, start, end, PAM coordinate, transcript, gene, cell line) for mapping epigenetic signals to sgRNA targets
- phenotype scores normalized into sgRNA activity scores per gene

## Outputs

- table_s4_shap_feature_importance.csv: feature-name column and mean |SHAP| value column with 30 ranked rows
- one-hot encoded sequence image matrices and min-max normalized epigenetic feature matrices for model input
- ranked list of top epigenetic features favorable for efficient epigenome editing (nucleosome positioning/distance to TSS, RNA expression, chromatin accessibility)

## How to apply

Assemble the sgRNA sequence and epigenetic feature matrix (nucleosome positioning as distance to TSS, chromatin accessibility from ATACdb ATAC-seq data, DNA methylation, RNA expression) using custom Python 3.6 scripts with pybedtools, then one-hot encode sgRNA sequences into an image-like coding scheme and min-max normalize all epigenetic features before model input. Fit an XGBoost gradient-boosted tree model on the 90% training split, then compute Shapley Additive exPlanations (Tree SHAP) values with the TreeExplainer interpreter, calculating mean |SHAP| values (average impact on model output magnitude) on the training dataset. Rank all features by mean |SHAP| and write the top 30 ranked features with their SHAP values to a CSV. The rationale is that identifying the top-ranked epigenetic features (nucleosome positioning/distance to TSS, RNA expression, chromatin accessibility) explains which features drive sgRNA efficacy in epigenome editing, complementing the EpiCas-DL deep learning predictions.

## Related tools

- **EpiCas-DL** (Deep learning framework whose sgRNA activity models provide the trained predictor whose features are extracted and interpreted) — https://github.com/yangqianq/EpiCas-DL
- **XGBoost** (Gradient-boosted tree algorithm trained on the sequence+epigenetic feature matrix whose SHAP values are computed for feature ranking)
- **TreeExplainer** (Interpreter that computes Tree SHAP values and mean |SHAP| feature importances on the training dataset)
- **pybedtools 0.9.0** (Used in custom Python 3.6 scripts to extract epigenetic feature values at sgRNA target coordinates)
- **Python 3.6** (Scripting environment for assembling the sgRNA sequence and epigenetic feature matrix)

## Examples

```
python example.py -sequence GGGCCCAGATCCCTCTATGTGCTCGAAGCAGGTGGACCCC -model CRISPRoff_epi -chrom chr12 -start 100592057 -end 100592096 -pamCoord 100592087 -transcript P1P2 -gene ACTR6 -cell_line Hek293t
```

## Evaluation signals

- Output CSV exists and contains a feature-name column plus a mean |SHAP| value column with exactly 30 ranked rows.
- Top-ranked features correspond to the epigenetic features reported as top favorable features in the paper (nucleosome positioning/distance to TSS, RNA expression, chromatin accessibility).
- Epigenetic features are min-max normalized before model input; sequences are one-hot encoded in the image-like scheme.
- Train/test split is 9:1 with 5-fold cross-validation used for hyperparameter optimization (GPyOpt, 300 acquisitions).
- Feature ranking reproduces the paper's direction of effects, e.g. high chromatin accessibility and high RNA expression associated with higher editing efficiency, DNA methylation associated with higher repression in CRISPRoff datasets.

## Limitations

- Only nucleosome positioning, chromatin accessibility, DNA methylation, and RNA expression were considered; incomplete epigenetic feature information limited EpiCas-DL model performance.
- Small CRISPRoff screening datasets mainly in HEK293T and K562 cell lines increase overfitting risk and limit generalization.
- Differences between CRISPRoff and CRISPRi systems might affect prediction performance in testing datasets; screening datasets may contain noise in measuring knock-down/knock-up efficacy.
- Epigenetic reference tracks for non-model cell lines are not bundled in the repo and must be fetched separately (e.g., from the 'epi_reference' external link).

## Evidence

- [results] Tree SHAP analysis of the CRISPRoff model ranked nucleosome positioning (distance to TSS), RNA expression levels, and chromatin accessibility (ATAC-seq) among the top five favorable features: [evidence span withheld — anchor status: not_found]
- [methods] Mean |SHAP| values, the average impact on model output magnitude, were interpreted on the training dataset by TreeExplainer: "Mean |SHAP| values, the average impact on model output magnitude, were interpreted on the training dataset by TreeExplainer"
- [methods] Feature identification was achieved using a recently developed method, Tree SHAP (Shapley Additive exPlanations): "Feature identification was achieved using a recently developed method, Tree SHAP (Shapley Additive exPlanations)"
- [methods] One-hot encoding strategy was used to formulate the sgRNA sequence into an image-like coding scheme: "One-hot encoding strategy was used to formulate the sgRNA sequence into an image-like coding scheme"
- [readme] The tool combined four types of epigenetic features, including methylation, gene expression, chromatin accessibility and the distance between target site and transcription start site: "The tool combined four types of epigenetic features, including methylation, gene expression, chromatin accessibility and the distance between target site and transcription start site"
