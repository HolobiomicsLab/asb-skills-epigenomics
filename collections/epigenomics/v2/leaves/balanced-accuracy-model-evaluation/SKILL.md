---
name: balanced-accuracy-model-evaluation
description: Use when benchmarking cell-specific enhancer classifiers on the Liu et
  al. eight-cell-line FASTA data, or when measuring cross-cell-line transferability
  (e.g., average BACC of a model trained on one cell line evaluated on the other seven).
when_to_use_negative:
- The input is already a feature table or prediction score set rather than raw enhancer
  FASTA sequences plus labels from the 'data' folder
- The task concerns generic enhancer identification on the Liu et al. mixed nine-cell-line
  dataset or enhancer-promoter data (EPdata) rather than cell-specific eight-cell-line
  transferability evaluation
- A single within-cell-line performance estimate is sufficient and the cross-cell-line
  56-pair transferability matrix is not needed
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Enhancer-MDLF
  role: Multi-input deep learning model (main.py) trained per cell line and evaluated
    via balanced accuracy on cross-cell-line test sets
  repo: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- name: Enhancer-IF
  role: Baseline integrated-ML enhancer predictor whose reported average balanced
    accuracy values are compared against Enhancer-MDLF
- name: dna2vec
  role: Source of pre-trained DNA k-mer vectors used by dna2vec_code.py to build features
    fed to the evaluated model
- name: tensorflow
  role: TensorFlow 2.2.0 framework in which the Enhancer-MDLF network is compiled
    and run before BACC scoring
- name: scikit-learn
  role: Provides the balanced accuracy metric computation over the eight cell lines'
    test sets
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae083
    title: 'Enhancer-MDLF: a novel deep learning framework for identifying cell-specific
      enhancers'
  source_content_hash: sha256:5d9eb8512d692b1ca31437a4c91f0f54309f0814a3f66d819be87e75b58e41b4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
  repo_url: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
  tools:
  - Enhancer-IF
  - Enhancer-MDLF
  - Python
  - dna2vec
  - numpy
  - scikit-learn
  - tensorflow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# balanced-accuracy-model-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates cell-specific enhancer prediction models using balanced accuracy (BACC) across eight human cell lines, including the 56-pair cross-cell-line transferability evaluation of Enhancer-MDLF versus Enhancer-IF.

## When to use

Use when benchmarking cell-specific enhancer classifiers on the Liu et al. eight-cell-line FASTA data, or when measuring cross-cell-line transferability (e.g., average BACC of a model trained on one cell line evaluated on the other seven).

## When NOT to use

- The input is already a feature table or prediction score set rather than raw enhancer FASTA sequences plus labels from the 'data' folder
- The task concerns generic enhancer identification on the Liu et al. mixed nine-cell-line dataset or enhancer-promoter data (EPdata) rather than cell-specific eight-cell-line transferability evaluation
- A single within-cell-line performance estimate is sufficient and the cross-cell-line 56-pair transferability matrix is not needed

## Inputs

- Enhancer training set FASTA sequences per cell line (data/train/${cell line}.fasta)
- Training set labels (data/train/${cell line}_y_train.txt)
- Independent test set FASTA sequences (data/test/${cell line}.fasta)
- Test set labels (data/test/${cell line}_y_test.txt)
- Pre-trained dna2vec DNA vectors
- HOCOMOCO Human v11 PWMs and p-value threshold scores

## Outputs

- Balanced accuracy (BACC) per cell-line source-target pair (56 values)
- Average BACC per source cell line across the other seven cell lines
- Comparison table of Enhancer-MDLF vs Enhancer-IF average BACC (e.g., cross_cell_line_bacc.csv)
- Model prediction results per cell line (e.g., NHLF)

## How to apply

Load the enhancer training and independent test FASTA sequences plus _y_train/_y_test labels for the eight cell lines from the repository's 'data' folder. Extract dna2vec-based features (dna2vec_code.py with the pre-trained dna2vec vectors) and motif features (motif_find.py with HOCOMOCO Human v11 PWMs and p-value threshold scores) for each train/test split, saving outputs to the 'feature' directory. Train Enhancer-MDLF (main.py, a TensorFlow 2.2.0 multi-input deep network) on each cell line training set, then compute balanced accuracy on every source-target cell-line pair (8 models × 7 external test sets = 56 pairs). Average the cross-cell-line BACC per source cell line and compare against Enhancer-IF's reported average BACC values. Validate by checking that Enhancer-MDLF outperforms Enhancer-IF across the eight cell lines and that similar pairs (HUVEC-K562, NHEK-HMEC) show higher cross-prediction performance, as in Supplementary Tables S6/S7.

## Related tools

- **Enhancer-MDLF** (Multi-input deep learning model (main.py) trained per cell line and evaluated via balanced accuracy on cross-cell-line test sets) — https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- **Enhancer-IF** (Baseline integrated-ML enhancer predictor whose reported average balanced accuracy values are compared against Enhancer-MDLF)
- **dna2vec** (Source of pre-trained DNA k-mer vectors used by dna2vec_code.py to build features fed to the evaluated model)
- **tensorflow** (TensorFlow 2.2.0 framework in which the Enhancer-MDLF network is compiled and run before BACC scoring)
- **scikit-learn** (Provides the balanced accuracy metric computation over the eight cell lines' test sets)

## Examples

```
python main.py --cell_line NHLF
```

## Evaluation signals

- All 8 source cell lines × 7 target test sets are evaluated, yielding 56 balanced accuracy values with no missing pairs
- Enhancer-MDLF average BACC exceeds Enhancer-IF across all eight cell lines (e.g., HEK293 model reaches 86.18% average BACC predicting other cell lines)
- Cross-prediction BACC ordering preserves mutual similarity patterns: HUVEC-K562 and NHEK-HMEC pairs show higher cross-cell-line performance
- Result table (cross_cell_line_bacc.csv) reproduces the relative ordering reported in Supplementary Tables S6/S7
- Feature files for every train and test split exist in the 'feature' directory before main.py is invoked

## Limitations

- Environment is pinned to Python 3.8, pip 23.3.1, tensorflow 2.2.0; other versions may break the network or metrics
- Evaluation depends on the repository's pre-trained dna2vec vectors and HOCOMOCO Human v11 PWMs/p-value threshold scores; swapping them changes results
- Only eight human cell lines are covered; conclusions about transferability do not extend to other cell types or species without new data organized in the documented data/train and data/test format
- In the EPdata folder, labels are positional (first half of each file labeled 0, second half labeled 1), which is error-prone if files are reordered

## Evidence

- [abstract] Enhancer-MDLF outperforms the previous method, Enhancer-IF, across eight distinct human cell lines: "Enhancer-MDLF outperforms the previous method, Enhancer-IF, across eight distinct human cell lines"
- [readme] The folder '**data**' contains the data of the enhancers, containing the sequences of the independent test sets and training sets on eight cell lines.: "The folder "**data**" contains the data of the enhancers, containing the sequences of the independent test sets and training sets on eight cell lines."
- [readme] Users can run the script as follows to compile and run Enhancer-MDLF: "Users can run the script as follows to compile and run Enhancer-MDLF"
- [readme] The folder '**dna2vec**' contains the pre-trained DNA vectors provided in dna2vec[2].: "The folder "**dna2vec**" contains the pre-trained DNA vectors provided in dna2vec[2]."
