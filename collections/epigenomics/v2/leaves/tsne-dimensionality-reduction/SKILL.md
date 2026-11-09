---
name: tsne-dimensionality-reduction
description: Use when promoter/non-promoter sequences (e.g. the iPro-WAEL 'data' folder
  human cell-line files) must be encoded with fused sequence descriptors and visually
  checked for class separation via t-SNE.
when_to_use_negative:
- Input is already a feature table or embedding — this skill starts from raw promoter
  sequences that must be encoded with iLearnPlus descriptors
- The goal is promoter classification performance (use the iPro-WAEL RF/CNN ensemble
  via main.py, not t-SNE visualization)
- Data are not the labeled promoter/non-promoter sequence files (first half label
  1, second half label 0) for which class-separation visualization is meaningful
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: iLearnPlus
  role: Extracts the five nucleic-acid feature descriptors (RCKmer, Mismatch, CKSNAP,
    TPCP, PseTNC) from promoter sequences
  repo: https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- name: sklearn
  role: Provides t-SNE for the 2D embedding of the normalized fused feature matrix
- name: numpy
  role: Used for normalizing/fusing the five descriptor vectors into a comparable
    feature matrix
- name: iPro-WAEL
  role: Source repository providing the human cell-line promoter datasets and the
    feature-extraction workflow
  repo: https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkac824
    title: 'iPro-WAEL: a comprehensive and robust framework for identifying promoters
      in multiple species'
  source_content_hash: sha256:0a78c292ee4a078f722c541ee536876d01f09c387593ad010fbbe7a90ea296bd
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
  repo_url: https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
  tools:
  - Python
  - iLearnPlus
  - iPro-WAEL
  - numpy
  - sklearn
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# tsne-dimensionality-reduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce the t-SNE visualization that shows promoters and non-promoters separating into two distinct clusters when encoded with the five fused nucleic-acid descriptors (RCKmer, Mismatch, CKSNAP, TPCP, PseTNC) used by iPro-WAEL on the four human cell-line datasets.

## When to use

Use when promoter/non-promoter sequences (e.g. the iPro-WAEL 'data' folder human cell-line files) must be encoded with fused sequence descriptors and visually checked for class separation via t-SNE.

## When NOT to use

- Input is already a feature table or embedding — this skill starts from raw promoter sequences that must be encoded with iLearnPlus descriptors
- The goal is promoter classification performance (use the iPro-WAEL RF/CNN ensemble via main.py, not t-SNE visualization)
- Data are not the labeled promoter/non-promoter sequence files (first half label 1, second half label 0) for which class-separation visualization is meaningful

## Inputs

- Promoter sequence files from the iPro-WAEL 'data' folder (sequences of the 12 datasets for seven species; first half positive, second half negative)
- Per-sequence feature vectors from iLearnPlus for the descriptors RCKmer, Mismatch, CKSNAP, TPCP, and PseTNC

## Outputs

- Normalized fused feature matrix per human cell line
- 2D t-SNE embedding (sklearn) of the fused features per cell line
- Scatter plot figure, e.g. tsne_human_promoter_features.png, with promoters and non-promoters colored distinctly

## How to apply

Load each human cell-line promoter dataset from the repository's 'data' folder, labeling the first half of each file as positive (label 1) and the second half as negative (label 0). Encode each sequence with iLearnPlus using the five nucleic-acid descriptors RCKmer, Mismatch, CKSNAP, TPCP, and PseTNC, then fuse the five descriptor vectors per sequence into a single feature vector. Normalize the fused features (e.g. min-max scaling with numpy/scikit-learn) so descriptors with different magnitude ranges are comparable before embedding. Apply scikit-learn's t-SNE to the normalized fused feature matrix to obtain a two-dimensional embedding per cell line, and scatter-plot it with matplotlib coloring positive and negative samples distinctly. Validation: positive and negative samples should fall into two visually distinct clusters, demonstrating that the fused features effectively distinguish promoters from non-promoters.

## Related tools

- **iLearnPlus** (Extracts the five nucleic-acid feature descriptors (RCKmer, Mismatch, CKSNAP, TPCP, PseTNC) from promoter sequences) — https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- **sklearn** (Provides t-SNE for the 2D embedding of the normalized fused feature matrix)
- **numpy** (Used for normalizing/fusing the five descriptor vectors into a comparable feature matrix)
- **iPro-WAEL** (Source repository providing the human cell-line promoter datasets and the feature-extraction workflow) — https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL

## Examples

```
from sklearn.manifold import TSNE; emb = TSNE(n_components=2).fit_transform(X_fused_normalized)
```

## Evaluation signals

- Positive (promoter) and negative (non-promoter) samples in the t-SNE plot form two visually distinct clusters for each of the four human cell lines
- Fused vectors are min-max normalized before t-SNE, so no single descriptor (e.g. PseTNC vs CKSNAP) dominates the embedding due to scale
- Each labeled file contributes samples from both halves (first half label 1, second half label 0) to the plot
- The embedding is 2-dimensional (sklearn TSNE with n_components=2) and is saved as a scatter figure such as tsne_human_promoter_features.png

## Limitations

- The feature file for the human dataset is too large to upload to GitHub, so features must be regenerated with feature_code.py and iLearnPlus before visualization
- Versions of sklearn, numpy, and h5py are unspecified in the repository (unlike Python 3.6, keras 2.3.1, tensorflow 2.0.0), so exact t-SNE results may vary with library versions
- The t-SNE cluster separation is qualitative/visual validation of feature separability, not a quantitative promoter-prediction performance measure

## Evidence
