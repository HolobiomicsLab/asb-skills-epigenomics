---
name: tcga-pancancer-dataset-loading
description: Use when a task requires training or re-tuning CITRUS, reproducing the
  paper's TCGA experiments, or inspecting the pre-packaged pan-cancer SGA/GEP/TF-target
  matrices delivered as a pickle file.
when_to_use_negative:
- Your input is a raw, unprocessed TCGA download (Firehose/GDC files) rather than
  the pre-processed pickle — preprocessing/filtering steps (top-variant gene selection,
  whitelisting) must be applied first
- You need patient-level clinical or imaging data; the pickle contains only SGA, GEP,
  TF-target matrix, cancer types, and tumor barcodes
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2640
tools:
- name: CITRUS
  role: Provides run_CITRUS.py and the packaged dataset_CITRUS.pkl consumed by this
    loading skill
  repo: https://github.com/osmanbeyoglulab/CITRUS
- name: Python 3.7
  role: Runtime for unpickling dataset_CITRUS.pkl and inspecting its dictionary structure
- name: Pandas
  role: Supports handling tabular structures when exploring the loaded matrices
- name: Numpy
  role: Handles array representations of the loaded SGA/GEP matrices
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkac881
    title: Interpretable deep learning for chromatin-informed inference of transcriptional
      programs driven by somatic alterations across cancers
  source_content_hash: sha256:9570d893df1fa8a39f21b279640d3103734bc4756d683dffd1788bd2855ed4f1
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
  - Python 3.7
  - SciPy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# tcga-pancancer-dataset-loading

## Summary

Load the CITRUS-packaged TCGA pan-cancer dataset (dataset_CITRUS.pkl, 5803 samples across 17 tumor types) from the ./data directory and inspect its contents (SGA index lists, GEP of 5541 filtered genes, TF-target matrix, cancer types, tumor barcodes) before model training or hyperparameter sweeps.

## When to use

Use when a task requires training or re-tuning CITRUS, reproducing the paper's TCGA experiments, or inspecting the pre-packaged pan-cancer SGA/GEP/TF-target matrices delivered as a pickle file.

## When NOT to use

- Your input is a raw, unprocessed TCGA download (Firehose/GDC files) rather than the pre-processed pickle — preprocessing/filtering steps (top-variant gene selection, whitelisting) must be applied first
- You need patient-level clinical or imaging data; the pickle contains only SGA, GEP, TF-target matrix, cancer types, and tumor barcodes

## Inputs

- dataset_CITRUS.pkl (pickled dataset with SGA, GEP, TF-target gene matrix, cancer types, tumor barcodes)
- TCGA pan-cancer sample set (5803 samples, 17 tumor types, whitelisted by TCGA Pan-Cancer Analysis Working Group)

## Outputs

- Loaded Python dictionary with keys for SGA index lists, GEP (5541 genes), TF-target matrix, cancer type list, and tumor list
- Validated dataset ready for run_CITRUS.py training and hyperparameter tuning

## How to apply

Download dataset_CITRUS.pkl from https://sites.pitt.edu/~xim33/CITRUS and place it in the ./data directory of the cloned osmanbeyoglulab/CITRUS repository. Open the pickle with Python's pickle module (open with 'rb') and inspect data.keys() to confirm the expected structures: the somatic alteration gene matrix stored as index lists for gene embedding, the gene expression matrix filtered to 5541 top-variant genes, the TF-target gene matrix, cancer type labels, and tumor barcodes. Verify the sample count is 5803 across 17 tumor types and that samples carry parallel RNA-seq, somatic mutation, and GISTIC copy number data with TCGA Pan-Cancer Analysis Working Group whitelisting. Once validated, pass the dataset via run_CITRUS.py arguments (--input_dir ./data, --dataset_name) so training or hyperparameter sweeps (e.g., dropout_rate 0.1-0.5, attention_size/attention_head grids) consume the correct inputs.

## Related tools

- **CITRUS** (Provides run_CITRUS.py and the packaged dataset_CITRUS.pkl consumed by this loading skill) — https://github.com/osmanbeyoglulab/CITRUS
- **Python 3.7** (Runtime for unpickling dataset_CITRUS.pkl and inspecting its dictionary structure)
- **Pandas** (Supports handling tabular structures when exploring the loaded matrices)
- **Numpy** (Handles array representations of the loaded SGA/GEP matrices)

## Examples

```
python run_CITRUS.py --input_dir ./data --dataset_name dataset_CITRUS.pkl
```

## Evaluation signals

- data.keys() returns the expected dictionary structure (SGA, GEP, TF-target matrix, cancer types, tumor barcodes) after pickle.load
- Sample count matches 5803 TCGA samples spanning 17 tumor types
- GEP matrix contains the filtered 5541 genes (top variant genes per cancer type present in the TF-target gene matrix)
- run_CITRUS.py executes with --input_dir ./data and the default dataset name without file-not-found errors
- Samples reflect TCGA whitelisting and parallel RNA-seq, somatic mutation, and GISTIC copy number availability

## Limitations

- The pickle is specific to the paper's preprocessed pan-cancer cohort; training on other datasets requires re-tuning hyperparameters per the README
- SGA is delivered as sparse binary index lists (transformed from a 11998-gene binary matrix), not a dense 0/1 matrix, so naive row/column indexing will misalign
- The dataset packages only processed inputs; raw TCGA Firehose and GDC ATAC-seq sources must be fetched separately

## Evidence

- [readme] The integrated dataset of the pickle file can be downloaded from https://sites.pitt.edu/~xim33/CITRUS: "The integrated dataset of the pickle file can be downloaded from https://sites.pitt.edu/~xim33/CITRUS"
- [readme] 5803 samples with 17 different tumor types from TCGA: "They included 5803 samples with 17 different tumor types from TCGA."
- [readme] GEP filtered to 5541 genes: "GEP contains continuous values of gene expression with filtered 5541 genes by considering top variant genes in each cancer type as well as gene availability in TF-target gene matrix."
- [supplementary] only used samples with parallel RNA-seq, somatic mutation, and GISTIC copy number data: "For our analysis, we only used samples with parallel RNA-seq, somatic mutation, and GISTIC copy number data."
- [readme] Download input dataset and put it into ./data directory: "Download input dataset "dataset_CITRUS.pkl" from https://sites.pitt.edu/~xim33/CITRUS and put it into ./data directory."
