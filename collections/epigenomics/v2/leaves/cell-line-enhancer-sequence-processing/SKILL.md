---
name: cell-line-enhancer-sequence-processing
description: Use when enhancer training/test FASTA sequences for one of the eight
  human cell lines must be converted into dna2vec and motif-based features before
  running Enhancer-MDLF (main.py) training or prediction.
when_to_use_negative:
- The features are already extracted in the 'feature' directory and the goal is only
  to train/predict — run 'python main.py --cell_line ${cell_line}' directly instead
- The input is a different data type (e.g. enhancer-promoter EPdata labels or the
  generic Liu et al. dataset) whose naming/format conventions differ from the eight-cell-line
  enhancer layout described in the README
- The goal is interpreting which TFBS motifs are important in a trained model — use
  SHAP.py ('SHAP.py' is the code for exploring important motifs) rather than re-extracting
  sequence features
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0654
tools:
- name: Enhancer-MDLF
  role: Target multi-input deep learning framework that consumes the extracted dna2vec
    and motif frequency features
  repo: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- name: motif_find.py
  role: Script that extracts HOCOMOCO v11 PWM-based motif frequency features from
    the enhancer FASTA sequences
  repo: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- name: dna2vec_code.py
  role: Script that extracts dna2vec-based features from the same enhancer FASTA sequences
  repo: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- name: dna2vec
  role: Source of the pre-trained DNA vectors used by dna2vec_code.py for k-mer embedding
- name: HOCOMOCO Human v11
  role: Database supplying the motif PWMs and p-value threshold scores used to count
    TFBS per motif
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae083
    title: 'Enhancer-MDLF: a novel deep learning framework for identifying cell-specific
      enhancers'
  source_content_hash: sha256:76a88cbd61d6c642c76589b9c0f2e625281a4b483370d15f48709b24453f57a4
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
  - Enhancer-MDLF
  - HOCOMOCO Human v11
  - HOCOMOCO Human v11 database
  - Python
  - dna2vec
  - dna2vec_code.py
  - motif_find.py
  - numpy
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# cell-line-enhancer-sequence-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encodes cell-specific enhancer FASTA sequences (e.g. data/train/NHLF.fasta) into dna2vec word-vector features and 401-dimensional HOCOMOCO Human v11 motif frequency features for input to the Enhancer-MDLF multi-input deep learning framework.

## When to use

Use when enhancer training/test FASTA sequences for one of the eight human cell lines must be converted into dna2vec and motif-based features before running Enhancer-MDLF (main.py) training or prediction.

## When NOT to use

- The features are already extracted in the 'feature' directory and the goal is only to train/predict — run 'python main.py --cell_line ${cell_line}' directly instead
- The input is a different data type (e.g. enhancer-promoter EPdata labels or the generic Liu et al. dataset) whose naming/format conventions differ from the eight-cell-line enhancer layout described in the README
- The goal is interpreting which TFBS motifs are important in a trained model — use SHAP.py ('SHAP.py' is the code for exploring important motifs) rather than re-extracting sequence features

## Inputs

- Enhancer training set FASTA: data/train/${cell_line}.fasta (e.g. data/train/NHLF.fasta)
- Enhancer test set FASTA: data/test/${cell_line}.fasta (e.g. data/test/NHLF.fasta)
- Label files: data/train/${cell_line}_y_train.txt and data/test/${cell_line}_y_test.txt
- HOCOMOCO Human v11 PWM files and p-value threshold scores from the 'motif' folder
- Pre-trained dna2vec DNA vectors from the 'dna2vec' folder

## Outputs

- dna2vec-based feature files written to the 'feature' directory (one per cell_line/set)
- Motif frequency feature files: a fixed-length 401-dimension vector (one per HOCOMOCO motif) per input sequence, counts normalized by sequence length

## How to apply

Organize the data exactly as the README expects: training/test sequences in 'data/train/${cell line name}.fasta' and 'data/test/${cell line name}.fasta' with matching label files (${cell line name}_y_train.txt / _y_test.txt). Create a clean Python 3.8 environment (e.g. 'conda create -n test python=3.8') and install dependencies with 'pip install -r requirements.txt' (tensorflow 2.2.0, scikit-learn, numpy) to avoid package conflicts. For each cell line and set, run 'dna2vec_code.py' and 'motif_find.py' with --input_file, --cell_line, and --set arguments; motif_find.py loads the PWMs and p-value threshold scores from the 'motif' folder (HOCOMOCO Human v11), slides each PWM along every sequence to score windows, retains windows passing the motif's p-value threshold, and assembles per-motif TFBS counts into a fixed 401-dimension vector that is converted to motif frequencies by dividing each count by the sequence length. Verify the 'feature' directory contains one motif frequency vector of 401-motif dimensionality per input sequence before proceeding to 'python main.py --cell_line ${cell_line}'.

## Related tools

- **Enhancer-MDLF** (Target multi-input deep learning framework that consumes the extracted dna2vec and motif frequency features) — https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- **motif_find.py** (Script that extracts HOCOMOCO v11 PWM-based motif frequency features from the enhancer FASTA sequences) — https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- **dna2vec_code.py** (Script that extracts dna2vec-based features from the same enhancer FASTA sequences) — https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- **dna2vec** (Source of the pre-trained DNA vectors used by dna2vec_code.py for k-mer embedding)
- **HOCOMOCO Human v11** (Database supplying the motif PWMs and p-value threshold scores used to count TFBS per motif)

## Examples

```
python motif_find.py --input_file data/train/NHLF.fasta --cell_line NHLF --set train
```

## Evaluation signals

- motif_find.py and dna2vec_code.py complete without error for both the train and test sets of the target cell line
- The 'feature' directory contains a motif feature file for each requested cell_line/set combination
- Each sequence is represented by a motif frequency vector of the expected 401-motif dimensionality
- Motif vector values are normalized to the sequence length (per-motif TFBS counts divided by sequence length)
- Downstream 'python main.py --cell_line ${cell_line}' runs without missing-feature errors, confirming the feature files match the training/test labels

## Limitations

- The pipeline is built around the eight human cell lines shipped in the 'data' folder; custom datasets must be reorganized into the exact ${cell_line}.fasta / ${cell_line}_y_train.txt format described in the README
- Environment is version-sensitive (Python 3.8, pip 23.3.1, tensorflow 2.2.0); the README recommends a fresh conda virtual environment to avoid conflicts with the user's commonly used packages
- The motif feature dimensionality is fixed at 401 HOCOMOCO v11 motifs, so the encoding is bound to that motif set and its p-value thresholds
- No changelog found in the repository, so behavioral changes between commits cannot be tracked

## Evidence

- [methods] the position weight matrix (PWM) of motifs and the p-value threshold score from the HOCOMOCO Human v11 database[3]: "the position weight matrix (PWM) of motifs and the p-value threshold score from the HOCOMOCO Human v11 database[3]"
- [methods] The file "**motif_find.py**" is the code used to extract motif features.: "The file "**motif_find.py**" is the code used to extract motif features."
- [readme] The folder "**data**" contains the data of the enhancers, containing the sequences of the independent test sets and training sets on eight cell lines.: "The folder "**data**" contains the data of the enhancers, containing the sequences of the independent test sets and training sets on eight cell lines."
- [readme] The output feature files will be saved in the 'feature' directory: "The output feature files will be saved in the 'feature' directory"
