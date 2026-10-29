---
name: motif-significance-evalue-filtering
description: Use when MEME has discovered candidate motifs from positive vs. negative
  sequence sets and you must retain only statistically significant motifs (E-value
  < 0.05) to build per-sample motif score features for a classifier.
when_to_use_negative:
- Input is already an encoded feature matrix or motif score table — no motif filtering
  or rescoring is needed.
- The analysis goal is de novo motif discovery itself rather than filtering discovered
  motifs into classifier features.
- Sequences are not the fixed 41-nt 6mA-centered windows csDMA expects; the motif
  scoring scheme assumes 41-nt samples.
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_3511
- http://edamontology.org/topic_3474
tools:
- name: MEME
  role: Discovers motifs in differential enrichment mode on positive training samples
    against the negative control set; supplies E-values and position-specific probability
    matrices to be filtered.
- name: feature_selection.py
  role: csDMA encoding script that scores each 41-nt sample against the retained motif
    probability matrices and writes matrix_motif1 and matrix_motif2.
  repo: https://github.com/liuze-nwafu/csDMA
- name: csDMA
  role: Downstream ExtraTrees-based 6mA prediction model that consumes the filtered-motif
    score matrices as Motif features.
  repo: https://github.com/liuze-nwafu/csDMA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-019-49430-4
    title: 'csDMA: an improved bioinformatics tool for identifying DNA 6 mA modifications
      via Chou’s 5-step rule'
  source_content_hash: sha256:4b76045afcf6eef26e490f155579d158a1f243becc237ec96d782ba239222149
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
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_3511
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# motif-significance-evalue-filtering

## Summary

Select statistically significant DNA motifs from MEME differential enrichment output by an E-value < 0.05 cutoff and convert the retained motifs' position-specific probability matrices into per-sample motif score matrices for downstream machine-learning feature encoding in csDMA.

## When to use

Use when MEME has discovered candidate motifs from positive vs. negative sequence sets and you must retain only statistically significant motifs (E-value < 0.05) to build per-sample motif score features for a classifier.

## When NOT to use

- Input is already an encoded feature matrix or motif score table — no motif filtering or rescoring is needed.
- The analysis goal is de novo motif discovery itself rather than filtering discovered motifs into classifier features.
- Sequences are not the fixed 41-nt 6mA-centered windows csDMA expects; the motif scoring scheme assumes 41-nt samples.

## Inputs

- MEME output containing discovered motifs with E-values and position-specific probability matrices (from differential enrichment on 2,768 positive vs. 2,716 negative 41-nt samples)
- 41-nt positive and negative training FASTA sequence datasets

## Outputs

- Per-sample motif score matrices matrix_motif1 and matrix_motif2, one row per 41-nt sample, scored against the retained significant motif probability matrices
- List of retained motifs passing the E-value < 0.05 significance cutoff

## How to apply

Run MEME in differential enrichment mode on the 2,768 positive cross-species 41-nt training samples with the 2,716 negative samples as the control set, limiting discovery to a maximum of 10 motifs. Filter the discovered motifs by E-value < 0.05; in csDMA this retained two significant motifs with E-values of 3.3e-18 and 2.9e-2. Extract the position-specific probability matrices of the retained motifs from the MEME output. Use the feature_selection.py encoding script to score each 41-nt sample against the retained motif probability matrices, writing the per-sample motif score matrices matrix_motif1 and matrix_motif2. These motif score matrices then feed the ExtraTrees-based csDMA model alongside Kmer and Binary feature subsets, which achieved the best combination (AUC 0.871).

## Related tools

- **MEME** (Discovers motifs in differential enrichment mode on positive training samples against the negative control set; supplies E-values and position-specific probability matrices to be filtered.)
- **feature_selection.py** (csDMA encoding script that scores each 41-nt sample against the retained motif probability matrices and writes matrix_motif1 and matrix_motif2.) — https://github.com/liuze-nwafu/csDMA
- **csDMA** (Downstream ExtraTrees-based 6mA prediction model that consumes the filtered-motif score matrices as Motif features.) — https://github.com/liuze-nwafu/csDMA

## Examples

```
python leaveoneout.py training_positive_dataset training_negative_dataset matrix_motif1 matrix_motif2
```

## Evaluation signals

- Only motifs with E-value < 0.05 are retained; in the reference run this yielded exactly two motifs with E-values 3.3e-18 and 2.9e-2.
- The generated motif matrices are consumable by leaveoneout.py (python leaveoneout.py training_positive_dataset training_negative_dataset matrix_motif1 matrix_motif2) and produce a LeaveOneOut score file.
- Motif score matrices contain one score row per 41-nt training sample (2,768 positive / 2,716 negative) with no missing samples.
- Combined Motif + Kmer + Binary feature encoding reproduces the reported best AUC of 0.871 (5-fold CV) / 0.878 on the training dataset with the ExtraTrees classifier.

## Limitations

- csDMA performance on the rice and cross-species datasets was relatively low compared to the M. musculus dataset.
- Only Motif, Kmer, and Binary encoding schemes are implemented; genomic and structural feature encodings were not yet incorporated into csDMA.
- csDMA was implemented in Python 2.7, which may complicate reuse in modern environments.

## Evidence

- [results] Motif E-value significance filter: [evidence span withheld — anchor status: not_found]
- [abstract] Feature encoding scheme: "three feature encoding schemes, Motif, Kmer, and Binary, were used to generate the feature matrix"
- [readme] leaveoneout.py usage: "Usage: python leaveoneout.py training_positive_dataset training_negative_dataset matrix_motif1 matrix_motif2"
- [methods] feature_selection.py role: "#This script implements the feature encoding scheme and can be auto involved in other scripts."
- [intro] Best feature combination: "the model achieved the best AUC value of 0.871 when three feature subsets Motif, Kmer, and Binary feature subsets were selected"
