---
name: ablation-study-performance-comparison
description: Use when a multi-input deep learning model combines two feature extractors
  (e.g. dna2vec and motif features) and you must quantify whether fusion outperforms
  each module trained alone across multiple cell lines.
when_to_use_negative:
- Input is already an extracted feature table — this skill covers the full extraction
  (dna2vec_code.py, motif_find.py), training, and evaluation pipeline
- The goal is cross-cell-line prediction via transfer learning or motif interpretation
  via SHAP.py rather than module-ablation performance comparison
- Only a single feature type is available (e.g. no HOCOMOCO PWMs or pre-trained dna2vec
  vectors), so the dna2vec-only vs motif-only vs combined contrast cannot be constructed
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Enhancer-MDLF
  role: The combined multi-input deep learning framework whose dna2vec-only and motif-only
    ablations are compared
  repo: https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- name: dna2vec
  role: Source of the pre-trained DNA vectors used by dna2vec_code.py to build one
    ablation feature arm
- name: tensorflow
  role: TensorFlow 2.2.0 framework used to train all three model variants (network.py
    architecture via main.py)
- name: scikit-learn
  role: Used to compute BACC and the six additional evaluation metrics for each variant
    on the independent test sets
- name: numpy
  role: Numerical dependency for feature handling and metric computation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae083
    title: 'Enhancer-MDLF: a novel deep learning framework for identifying cell-specific
      enhancers'
  source_content_hash: sha256:33a21fb80c57bcd2ce9d9b996654d65bf746e65e712793330694bd76968615d9
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
  - TensorFlow
  - dna2vec
  - numpy
  - scikit-learn
  - tensorflow
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# ablation-study-performance-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce a module-combination ablation for Enhancer-MDLF: train dna2vec-only, motif-only, and combined multi-input variants per cell line and compare all reported metrics on independent test sets to quantify the benefit of feature fusion for cell-specific enhancer prediction.

## When to use

Use when a multi-input deep learning model combines two feature extractors (e.g. dna2vec and motif features) and you must quantify whether fusion outperforms each module trained alone across multiple cell lines.

## When NOT to use

- Input is already an extracted feature table — this skill covers the full extraction (dna2vec_code.py, motif_find.py), training, and evaluation pipeline
- The goal is cross-cell-line prediction via transfer learning or motif interpretation via SHAP.py rather than module-ablation performance comparison
- Only a single feature type is available (e.g. no HOCOMOCO PWMs or pre-trained dna2vec vectors), so the dna2vec-only vs motif-only vs combined contrast cannot be constructed

## Inputs

- Cell-line enhancer training FASTA files and label files ('data/train/${cell line}.fasta', 'data/train/${cell line}_y_train.txt')
- Independent test FASTA files and label files ('data/test/${cell line}.fasta', 'data/test/${cell line}_y_test.txt')
- Pre-trained DNA vectors from the dna2vec folder
- HOCOMOCO Human v11 PWMs and p-value threshold scores from the motif folder
- Trained model variants (dna2vec-only, motif-only, combined Enhancer-MDLF) from the model folder

## Outputs

- Per-variant, per-cell-line metric table (ablation_results.csv) with BACC and six additional performance metrics
- Comparison showing whether the combined multi-input model outperforms each single-feature module on all eight cell lines

## How to apply

Load the enhancer training and independent test FASTA sets with their label files (e.g. 'data/train/NHLF.fasta', 'data/train/NHLF_y_train.txt') for all eight cell lines. Extract dna2vec-based features with dna2vec_code.py (using the pre-trained DNA vectors in the dna2vec folder) and motif-based features with motif_find.py, which scans sequences against the PWMs and p-value threshold scores from the HOCOMOCO Human v11 database; both write outputs to the 'feature' directory. Train three variants per cell line — dna2vec-only, motif-only, and the combined multi-input Enhancer-MDLF — using main.py with the network.py TensorFlow 2.2.0 architecture. Evaluate each variant on its cell line's independent test set with scikit-learn, computing balanced accuracy (BACC) plus six additional metrics, then compile per-variant, per-cell-line results into ablation_results.csv. The ablation is confirmed correct only if the combined model outperforms both single-feature models across the metrics, reproducing the pattern of Figure 3 and Supplementary Tables S2/S3.

## Related tools

- **Enhancer-MDLF** (The combined multi-input deep learning framework whose dna2vec-only and motif-only ablations are compared) — https://github.com/HaoWuLab-Bioinformatics/Enhancer-MDLF
- **dna2vec** (Source of the pre-trained DNA vectors used by dna2vec_code.py to build one ablation feature arm)
- **tensorflow** (TensorFlow 2.2.0 framework used to train all three model variants (network.py architecture via main.py))
- **scikit-learn** (Used to compute BACC and the six additional evaluation metrics for each variant on the independent test sets)
- **numpy** (Numerical dependency for feature handling and metric computation)

## Examples

```
python dna2vec_code.py --input_file data/train/NHLF.fasta --cell_line NHLF --set train; python motif_find.py --input_file data/train/NHLF.fasta --cell_line NHLF --set train; python main.py --cell_line NHLF
```

## Evaluation signals

- The combined multi-input Enhancer-MDLF outperforms both the dna2vec-only and motif-only variants on every cell line tested (eight cell lines)
- All six reported metrics, including balanced accuracy (BACC), show improvement for the combined model relative to each individually trained module
- ablation_results.csv contains a complete per-variant, per-cell-line metric table with no missing cell lines
- The resulting ranking reproduces the ablation pattern of Figure 3 and Supplementary Tables S2/S3
- Feature extraction step 2 completes without error and writes dna2vec and motif feature files for both train and test splits to the 'feature' directory

## Limitations

- Results are specific to the eight provided cell lines and the Liu et al.-style train/test split; users with their own data must reorganize files into the 'data/train' and 'data/test' FASTA-plus-label format before the ablation is valid
- Feature extraction depends on external resources (pre-trained dna2vec vectors, HOCOMOCO Human v11 PWMs and p-value thresholds); missing or mismatched versions invalidate the comparison
- The environment is pinned (Python 3.8, tensorflow 2.2.0, pip 23.3.1); running under other versions may prevent reproduction of the ablation results

## Evidence

- [abstract] Enhancer-MDLF outperforms the previous method, Enhancer-IF, across eight distinct human cell lines: "Enhancer-MDLF outperforms the previous method, Enhancer-IF, across eight distinct human cell lines"
- [readme] data folder description: "The folder "**data**" contains the data of the enhancers, containing the sequences of the independent test sets and training sets on eight cell lines."
- [methods] motif feature source: "the position weight matrix (PWM) of motifs and the p-value threshold score from the HOCOMOCO Human v11 database"
- [readme] run command: "`python main.py --cell_line ${cell_line}`    
e.g.`python main.py --cell_line NHLF`"
- [readme] dependency pinning: "Python 3.8  
pip 23.3.1  
tensorflow 2.2.0  
scikit-learn  
numpy"
