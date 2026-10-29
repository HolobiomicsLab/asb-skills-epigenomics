---
name: upbm-sequence-motif-trimming-with-meme
description: Use when preprocessing uPBM training data (long oligos, e.g. 35-mers
  in proteins/{protein}/{protein}_training.txt) prior to affinity regression, to cut
  sequences down to the MEME-derived motif region.
when_to_use_negative:
- Input is HT-SELEX per-cycle enrichment data (proteins/{protein}/{cycle}.txt) — that
  pipeline uses SVM filtering of inconsistent enrichments, not MEME-based trimming
- Input is gcPBM data (proteins/{protein}/{protein}_{concentration}.txt with _freq_matrix_6.txt),
  which follows the gcpbm_regressor.py workflow without this trimming step
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_0160
tools:
- name: MEME suite
  role: Motif discovery on uPBM oligos; its result defines the motif region used to
    trim sequences and reduce variability
- name: DNAffinity
  role: Downstream consumer of the trimmed uPBM sequences; its upbm_regressor.py trains
    the random-forest affinity model
  repo: https://github.com/Jalbiti/DNAffinity
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkac708
    title: 'DNAffinity: a machine-learning approach to predict DNA binding affinities
      of transcription factors'
  source_content_hash: sha256:3490a33d7843a1d157ad320a77c8bf5abb039b334ea1828a679ccf775ae62c66
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/Jalbiti/DNAffinity
  repo_url: https://github.com/Jalbiti/DNAffinity
  tools:
  - BEESEM
  - BindSpace
  - DNAffinity
  - DeepBind
  - MEME suite
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# upbm-sequence-motif-trimming-with-meme

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A uPBM-specific preprocessing step that trims 35-mer oligos to the motif region using the MEME suite motif result, reducing sequence variability before training the DNAffinity random-forest regressor.

## When to use

Use when preprocessing uPBM training data (long oligos, e.g. 35-mers in proteins/{protein}/{protein}_training.txt) prior to affinity regression, to cut sequences down to the MEME-derived motif region.

## When NOT to use

- Input is HT-SELEX per-cycle enrichment data (proteins/{protein}/{cycle}.txt) — that pipeline uses SVM filtering of inconsistent enrichments, not MEME-based trimming
- Input is gcPBM data (proteins/{protein}/{protein}_{concentration}.txt with _freq_matrix_6.txt), which follows the gcpbm_regressor.py workflow without this trimming step

## Inputs

- uPBM per-TF training file: proteins/{protein}/{protein}_training.txt with 35-mer sequences and experimental binding affinities (66 mouse TFs in the DREAM5 benchmark)
- MEME suite motif discovery output for the same TF's oligo set

## Outputs

- Trimmed uPBM sequences restricted to the MEME-derived motif region, with affinities retained, ready for DNAffinity regressor training

## How to apply

Load the per-TF uPBM training file `proteins/{protein}/{protein}_training.txt` containing 35-mer sequences with experimental binding affinities. Run the MEME suite on the sequences to identify the enriched motif, then trim each oligo to the motif region so that the variability of the sequence diminishes — this concentrates the feature space on positions the TF actually recognizes. The trimmed sequences then feed into the DNAffinity pipeline (with undersampling and 10x weighting of PWM-matching sequences) for random-forest training on 80% of data and R2 evaluation on the held-out 20%. Note this trimming is uPBM-specific; HT-SELEX data instead uses SVM filtering of inconsistent per-cycle enrichments rather than MEME-based trimming.

## Related tools

- **MEME suite** (Motif discovery on uPBM oligos; its result defines the motif region used to trim sequences and reduce variability)
- **DNAffinity** (Downstream consumer of the trimmed uPBM sequences; its upbm_regressor.py trains the random-forest affinity model) — https://github.com/Jalbiti/DNAffinity

## Examples

```
meme proteins/Gata4/Gata4_training.txt -dna -oc meme_out; python upbm_regressor.py Gata4
```

## Evaluation signals

- Trimmed sequences should be shorter than the original 35-mers and aligned to the MEME-derived motif region, with experimental affinities preserved per sequence
- Downstream uPBM regression should reproduce the reported average R2 = 0.69 ± 0.17 across the 66 mouse TFs (trained on 80%, tested on 20%)
- Cross-dataset transfer (HT-SELEX-trained model applied to trimmed uPBM data) should yield an average Pearson correlation near 0.47, above DeepBind (0.35), BEESEM (0.36) and BindSpace (0.20)
- File layout check per README: the trimmed training data must remain compatible with the `proteins/{protein}/{protein}_training.txt` template consumed by `upbm_regressor.py`

## Limitations

- The concept of a transcription factor binding site is not uniquely defined — it depends on the intrinsically noisy, low-resolution experimental technique — so trimming windows may not generalize across platforms
- Trimming is tied to MEME's motif estimate; a poor motif call on a given TF will propagate into reduced prediction quality (uPBM R2 shows a large spread, ± 0.17)

## Evidence
