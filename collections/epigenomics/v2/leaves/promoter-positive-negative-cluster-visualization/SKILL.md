---
name: promoter-positive-negative-cluster-visualization
description: Use when you have labeled promoter FASTA-style sequence sets (first half
  positive, second half negative) per human cell line and need a visual check that
  fused sequence-encoding features separate the two classes before model training.
when_to_use_negative:
- Input is already a feature matrix or embedding — this skill starts from raw promoter
  sequences and includes the descriptor-encoding step
- Goal is quantitative promoter prediction performance (accuracy/MCC benchmarks) rather
  than a qualitative visual class-separation check — use the iPro-WAEL main.py model
  workflow instead
- Working with non-human species datasets or enhancer-vs-promoter (EPdata) classification,
  which the source t-SNE finding does not cover
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: iLearnPlus
  role: Extracts the five nucleic-acid descriptors (RCKmer, Mismatch, CKSNAP, TPCP,
    PseTNC) from the promoter sequences
  repo: https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- name: sklearn
  role: Provides t-SNE for the 2D embedding and min-max normalization of the fused
    feature matrix
- name: numpy
  role: Fuses the five descriptor vectors per sequence into a single feature vector
    and supports scaling
- name: iPro-WAEL
  role: Source framework whose 'data' folder supplies the four human cell line promoter
    datasets with first-half/second-half labels
  repo: https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkac824
    title: 'iPro-WAEL: a comprehensive and robust framework for identifying promoters
      in multiple species'
  source_content_hash: sha256:c58a417014885449f701e712ce47b2133c05e574e673ed28108af55c992b7286
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# promoter-positive-negative-cluster-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the iPro-WAEL t-SNE diagnostic that encodes promoter and non-promoter sequences with five fused nucleic-acid descriptors (RCKmer, Mismatch, CKSNAP, TPCP, PseTNC) and projects the normalized fused feature matrix into 2D to visually verify that positives and negatives form two distinct clusters across the four human cell lines.

## When to use

Use when you have labeled promoter FASTA-style sequence sets (first half positive, second half negative) per human cell line and need a visual check that fused sequence-encoding features separate the two classes before model training.

## When NOT to use

- Input is already a feature matrix or embedding — this skill starts from raw promoter sequences and includes the descriptor-encoding step
- Goal is quantitative promoter prediction performance (accuracy/MCC benchmarks) rather than a qualitative visual class-separation check — use the iPro-WAEL main.py model workflow instead
- Working with non-human species datasets or enhancer-vs-promoter (EPdata) classification, which the source t-SNE finding does not cover

## Inputs

- Promoter/non-promoter sequence datasets for four human cell lines from the iPro-WAEL repository 'data' folder (first half of each file labeled 1, second half labeled 0)
- Per-sequence feature vectors computed by iLearnPlus descriptors RCKmer, Mismatch, CKSNAP, TPCP, and PseTNC

## Outputs

- A two-dimensional t-SNE embedding (2D point coordinates) of the normalized fused feature matrix per human cell line
- A matplotlib scatter figure tsne_human_promoter_features.png with positives and negatives colored distinctly

## How to apply

Load each human cell line dataset from the repository's 'data' folder, treating the first half of each file as positive (label 1) and the second half as negative (label 0), consistent with the iPro-WAEL README labeling convention. Encode each sequence with iLearnPlus using the five nucleic-acid descriptors RCKmer, Mismatch, CKSNAP, TPCP, and PseTNC, then fuse the five per-sequence descriptor vectors into a single feature vector and normalize it (e.g., min-max scaling with numpy/scikit-learn) so descriptors of different magnitude ranges are comparable. Apply t-SNE from sklearn to the normalized fused feature matrix of each cell line to obtain a two-dimensional embedding, and plot it as a scatter colored by class (positive vs negative) with matplotlib, saving the figure as tsne_human_promoter_features.png. Rationale: this is the qualitative feature-separation diagnostic underlying the iPro-WAEL paper's claim that its encoding scheme outperforms the relatively simple feature encoding schemes of prior promoter prediction methods.

## Related tools

- **iLearnPlus** (Extracts the five nucleic-acid descriptors (RCKmer, Mismatch, CKSNAP, TPCP, PseTNC) from the promoter sequences) — https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL
- **sklearn** (Provides t-SNE for the 2D embedding and min-max normalization of the fused feature matrix)
- **numpy** (Fuses the five descriptor vectors per sequence into a single feature vector and supports scaling)
- **iPro-WAEL** (Source framework whose 'data' folder supplies the four human cell line promoter datasets with first-half/second-half labels) — https://github.com/HaoWuLab-Bioinformatics/iPro-WAEL

## Examples

```
from sklearn.manifold import TSNE; from sklearn.preprocessing import MinMaxScaler; Z = TSNE(n_components=2).fit_transform(MinMaxScaler().fit_transform(X_fused))
```

## Evaluation signals

- Positive (promoter) and negative (non-promoter) samples in all four human cell lines fall into two visually distinct clusters in the t-SNE scatter
- Each dataset file is split exactly half positive (label 1) / half negative (label 0) before encoding
- The fused feature matrix is normalized (e.g., min-max) before t-SNE, so the five descriptor scales are comparable
- The embedding is exactly two-dimensional and the figure is saved as tsne_human_promoter_features.png
- All five descriptors (RCKmer, Mismatch, CKSNAP, TPCP, PseTNC) contribute to the fused vector — no descriptor omitted

## Limitations

- The feature file for the human dataset is too large to upload to GitHub, so features must be re-extracted locally with iLearnPlus before visualization
- The t-SNE separation is a qualitative visual diagnostic, not a quantitative performance metric; cluster distinctness may vary with t-SNE perplexity and random seed
- Versions for sklearn, numpy, and h5py are unspecified in the paper/README (unlike Python 3.6, keras 2.3.1, tensorflow 2.0.0), so environment reproducibility is partial

## Evidence
