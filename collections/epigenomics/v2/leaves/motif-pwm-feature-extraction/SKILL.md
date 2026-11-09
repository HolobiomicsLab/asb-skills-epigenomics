---
name: motif-pwm-feature-extraction
description: Use when enhancer or promoter FASTA sequences must be converted into
  TFBS motif-frequency features (via PWM scanning) as input to a multi-input deep
  learning predictor such as Enhancer-MDLF.
when_to_use_negative:
- Input is already a numerical feature table — re-scanning FASTA against PWMs would
  duplicate the feature step
- The goal is k-mer embedding features rather than TFBS motif frequencies — use dna2vec_code.py
  with the pre-trained dna2vec vectors instead
- The organism/context is not covered by the HOCOMOCO Human v11 PWM collection (e.g.
  non-human, non-mouse species)
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0749
tools:
- name: Enhancer-MDLF
  role: Downstream multi-input deep learning framework whose motif module consumes
    the extracted PWM-scanning features
  repo: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- name: motif_find.py
  role: Script that scans enhancer FASTA sequences against HOCOMOCO Human v11 PWMs
    to extract motif features
  repo: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- name: Python 3.8
  role: Runtime environment (with tensorflow 2.2.0, scikit-learn, numpy) in which
    motif_find.py is executed
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae083
    title: 'Enhancer-MDLF: a novel deep learning framework for identifying cell-specific
      enhancers'
  source_content_hash: sha256:78dc3bc32fec9b31a6f6fdf23521a5a14f0b5344501b93b2e34641331081306a
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
  - Python
  - Python 3.8
  - TensorFlow
  - motif_find.py
  - numpy
  - pip
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# motif-pwm-feature-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts motif-based features for cell-specific enhancer prediction by scanning enhancer FASTA sequences against position weight matrices (PWMs) and p-value threshold scores from the HOCOMOCO Human v11 database, producing frequency-feature files that serve as the motif-module input to Enhancer-MDLF.

## When to use

Use when enhancer or promoter FASTA sequences must be converted into TFBS motif-frequency features (via PWM scanning) as input to a multi-input deep learning predictor such as Enhancer-MDLF.

## When NOT to use

- Input is already a numerical feature table — re-scanning FASTA against PWMs would duplicate the feature step
- The goal is k-mer embedding features rather than TFBS motif frequencies — use dna2vec_code.py with the pre-trained dna2vec vectors instead
- The organism/context is not covered by the HOCOMOCO Human v11 PWM collection (e.g. non-human, non-mouse species)

## Inputs

- Enhancer FASTA sequences (e.g. 'data/train/NHLF.fasta', 'data/test/NHLF.fasta')
- Cell-line label files ('data/train/${cell_line}_y_train.txt', 'data/test/${cell_line}_y_test.txt')
- HOCOMOCO Human v11 PWM files and p-value threshold scores from the 'motif' folder

## Outputs

- Motif-based feature files for training and test sets, saved in the 'feature' directory
- Motif-module input features consumed by Enhancer-MDLF training/prediction and SHAP.py interpretation

## How to apply

Organize sequences in the repository layout ('data/train/${cell_line}.fasta' with '${cell_line}_y_train.txt', 'data/test/${cell_line}.fasta' with '${cell_line}_y_test.txt'; e.g. NHLF) before extraction. Run motif_find.py once per training split and once per independent test split, passing --input_file (the FASTA to scan), --cell_line (the cell-line name, e.g. NHLF), and --set (train|test); the script scans each sequence against the PWMs and p-value threshold scores in the 'motif' folder, which come from the HOCOMOCO Human v11 database. Output feature files are written to the 'feature' directory and, together with dna2vec-based features produced by dna2vec_code.py, form the two input modules of Enhancer-MDLF. The same extraction must be applied identically to train and test sets so that motif-feature dimensions match the trained model; for model interpretation, the same motif-module features are fed to SHAP.py to rank TFBS motifs per cell line.

## Related tools

- **Enhancer-MDLF** (Downstream multi-input deep learning framework whose motif module consumes the extracted PWM-scanning features) — https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- **motif_find.py** (Script that scans enhancer FASTA sequences against HOCOMOCO Human v11 PWMs to extract motif features) — https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- **Python 3.8** (Runtime environment (with tensorflow 2.2.0, scikit-learn, numpy) in which motif_find.py is executed)

## Examples

```
python motif_find.py --input_file data/train/NHLF.fasta --cell_line NHLF --set train
```

## Evaluation signals

- Output feature files exist in the 'feature' directory for both the train and test splits of each processed cell line
- Motif feature dimensions match between training and independent test sets for the same cell line, allowing main.py to train and predict without shape errors
- Running the full pipeline (e.g. on NHLF) yields test results consistent with the paper's reported Enhancer-MDLF performance, which outperforms Enhancer-IF across the eight cell lines
- SHAP.py run on the extracted motif features reproduces reported key motifs (BACH1 highly ranked in HUVEC, GATA1 in K562, SP1 across cell lines)

## Limitations

- Motif features depend entirely on the bundled HOCOMOCO Human v11 PWMs and p-value threshold scores; no alternative motif databases are supported by the described workflow
- Custom datasets must be reformatted into the exact 'data/train|test/${cell_line}.fasta' + '${cell_line}_y_train|test.txt' layout before feature extraction will work
- The feature step is coupled to the repository's environment (Python 3.8, tensorflow 2.2.0); running under incompatible versions may break extraction or downstream model loading

## Evidence
