---
name: motif-discovery-differential-enrichment
description: Use when you have labeled positive and negative short sequence datasets
  (e.g., 6 mA sites vs. non-sites) and need enriched motifs as encoded features for
  a downstream machine-learning classifier.
when_to_use_negative:
- When the input is already an encoded feature table (e.g., k-mer frequency, KSNPF,
  or binary features) and no motif discovery is needed
- When samples are not short fixed-length sequences centered on the modified base
  (csDMA uses 41-nt samples); motif scoring against other sequence contexts is not
  supported here
- When a predictor for a single species with its own published encoding (e.g., iDNA6mA-PseKNC
  for M. musculus or i6mA-Pred for rice) is required instead of cross-species motif
  features
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_3511
- http://edamontology.org/topic_3474
tools:
- name: csDMA
  role: Overall prediction pipeline that consumes the motif score matrices as the
    Motif feature subset
  repo: https://github.com/liuze-nwafu/csDMA
- name: MEME
  role: Differential enrichment motif discovery on positive samples with negatives
    as control; produces motif probability matrices
- name: feature_selection.py
  role: Implements the feature encoding scheme, scoring each 41-nt sample against
    retained motif probability matrices to write matrix_motif1 and matrix_motif2
  repo: https://github.com/liuze-nwafu/csDMA
- name: scikit-learn ExtraTrees
  role: Classifier trained on the encoded Motif + Kmer + Binary features; selected
    among five algorithms by 5-fold cross-validation
  repo: https://github.com/liuze-nwafu/csDMA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-019-49430-4
    title: 'csDMA: an improved bioinformatics tool for identifying DNA 6 mA modifications
      via Chou’s 5-step rule'
  source_content_hash: sha256:d3d122d534d6ccbda5dd2ab83d466a17cf9fd69b08c579cccd647d6f24a7801c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liuze-nwafu/csDMA
  tools:
  - MEME
  - Python
  - csDMA
  - feature_selection.py
  - scikit-learn ExtraTrees
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_3511
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# motif-discovery-differential-enrichment

## Summary

Discover sequence motifs enriched around 6 mA sites by running MEME in differential enrichment mode on positive vs. negative 41-nt training samples, then convert significant motif probability matrices into per-sample motif score features (matrix_motif1/matrix_motif2) for the csDMA ExtraTrees model.

## When to use

Use when you have labeled positive and negative short sequence datasets (e.g., 6 mA sites vs. non-sites) and need enriched motifs as encoded features for a downstream machine-learning classifier.

## When NOT to use

- When the input is already an encoded feature table (e.g., k-mer frequency, KSNPF, or binary features) and no motif discovery is needed
- When samples are not short fixed-length sequences centered on the modified base (csDMA uses 41-nt samples); motif scoring against other sequence contexts is not supported here
- When a predictor for a single species with its own published encoding (e.g., iDNA6mA-PseKNC for M. musculus or i6mA-Pred for rice) is required instead of cross-species motif features

## Inputs

- 2,768 positive cross-species 41-nt 6 mA training samples
- 2,716 negative cross-species 41-nt control samples
- MEME differential enrichment output containing position-specific probability matrices for discovered motifs

## Outputs

- Filtered set of significant motifs (E-value < 0.05) with their position-specific probability matrices
- Per-sample motif score matrices matrix_motif1 and matrix_motif2 consumable by csDMA training/evaluation scripts

## How to apply

Run MEME in differential enrichment mode on the 2,768 positive cross-species 41-nt training samples using the 2,716 negative samples as the control set, limiting discovery to a maximum of 10 motifs. Filter the discovered motifs by E-value < 0.05 (in csDMA this retained motifs with E-values 3.3e-18 and 2.9e-2). Extract the position-specific probability matrices of the retained motifs from the MEME output. Use the feature_selection.py encoding script to score each 41-nt sample against the retained motif probability matrices and write the per-sample motif score matrices matrix_motif1 and matrix_motif2. These motif features are combined with Kmer and Binary subsets (best combination AUC 0.871) and consumed by scripts such as classifier_selection.py and leaveoneout.py for ExtraTrees model building and evaluation.

## Related tools

- **csDMA** (Overall prediction pipeline that consumes the motif score matrices as the Motif feature subset) — https://github.com/liuze-nwafu/csDMA
- **MEME** (Differential enrichment motif discovery on positive samples with negatives as control; produces motif probability matrices)
- **feature_selection.py** (Implements the feature encoding scheme, scoring each 41-nt sample against retained motif probability matrices to write matrix_motif1 and matrix_motif2) — https://github.com/liuze-nwafu/csDMA
- **scikit-learn ExtraTrees** (Classifier trained on the encoded Motif + Kmer + Binary features; selected among five algorithms by 5-fold cross-validation) — https://github.com/liuze-nwafu/csDMA

## Examples

```
python leaveoneout.py training_positive_dataset training_negative_dataset matrix_motif1 matrix_motif2
```

## Evaluation signals

- All retained motifs must have E-value < 0.05; in the csDMA run, E-values 3.3e-18 and 2.9e-2 passed the filter
- matrix_motif1 and matrix_motif2 files must be directly consumable by leaveoneout.py and classifier_selection.py (Usage: python leaveoneout.py training_positive_dataset training_negative_dataset matrix_motif1 matrix_motif2)
- The Motif + Kmer + Binary feature combination should reproduce the reported best AUC of 0.871 under 5-fold cross-validation and AUC 0.878 on the training dataset
- Number of discovered motifs must not exceed the maximum of 10 set for MEME discovery
- leaveoneout.py must successfully emit a LeaveOneOut score file for the ExtraTrees-based model

## Limitations

- csDMA performance was relatively low on the rice and cross-species datasets, so motif-derived features may transfer imperfectly across species
- Only computational sequence features (Motif, Kmer, Binary) were used; genomic and structural feature encoding schemes were not incorporated in this version of csDMA
- csDMA is implemented in Python 2.7, which may complicate reuse in modern environments

## Evidence

- [abstract] Motif probability matrix generation: "three feature encoding schemes, Motif, Kmer, and Binary, were used to generate the feature matrix"
- [intro] Best feature combination: "the model achieved the best AUC value of 0.871 when three feature subsets Motif, Kmer, and Binary feature subsets were selected"
- [readme] feature_selection.py role: "#This script implements the feature encoding scheme and can be auto involved in other scripts."
- [readme] leaveoneout.py usage: "Usage: python leaveoneout.py training_positive_dataset training_negative_dataset matrix_motif1 matrix_motif2"
