---
name: kmer-undersampling-for-affinity-training-sets
description: 'Use when preparing uPBM training data for DNAffinity''s random-forest
  regressor: 35-mer affinities are dominated by weak binders and the PWM-defined high-affinity
  motif must be emphasized (10x weight) after MEME-based trimming.'
when_to_use_negative:
- 'Input is gcPBM or HT-SELEX data: those techniques use different preprocessing (gcPBM
  `proteins/{protein}/{protein}_{concentration}.txt` plus `_freq_matrix_6.txt`; HT-SELEX
  cycle files with SVM removal of inconsistent enrichments between cycles), not uPBM
  MEME-trimming plus undersampling/weighting.'
- The `{protein}_training.txt` file is already the preprocessed template consumed
  by `upbm_regressor.py` (e.g. test_data `Gata4_training.txt`) — launch the regressor
  directly instead of re-balancing.
- The goal is in vivo TFBS discovery in yeast (DNAffinity predictions combined with
  yeast G1 nucleosome maps) rather than rebuilding an in vitro uPBM training set.
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: DNAffinity
  role: Provides the random-forest regressor (upbm_regressor.py) that consumes the
    undersampled, 10x-weighted {protein}_training.txt and trains on MD-derived DNA
    equilibrium geometry and flexibility parameters
  repo: https://github.com/Jalbiti/DNAffinity
- name: MEME suite
  role: Motif discovery whose result defines the motif region for trimming uPBM 35-mers
    and the PWM-derived high-affinity motif receiving 10x weight
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkac708
    title: 'DNAffinity: a machine-learning approach to predict DNA binding affinities
      of transcription factors'
  source_content_hash: sha256:16a956d62e5f822c24000d32a5dce3e25e3e296ed2f55203e4366621888a83d9
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
  - DNAffinity
  - MEME suite
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# kmer-undersampling-for-affinity-training-sets

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Balances uPBM 35-mer affinity training sets before DNAffinity random-forest training by undersampling low-affinity k-mers and giving 10x weight to sequences matching the PWM-derived high-affinity motif, after trimming to the MEME-derived motif region. This keeps the regression focused on the informative high-affinity regime and reproduces the paper's benchmark R2 = 0.69 ± 0.17 across 66 mouse TFs.

## When to use

Use when preparing uPBM training data for DNAffinity's random-forest regressor: 35-mer affinities are dominated by weak binders and the PWM-defined high-affinity motif must be emphasized (10x weight) after MEME-based trimming.

## When NOT to use

- Input is gcPBM or HT-SELEX data: those techniques use different preprocessing (gcPBM `proteins/{protein}/{protein}_{concentration}.txt` plus `_freq_matrix_6.txt`; HT-SELEX cycle files with SVM removal of inconsistent enrichments between cycles), not uPBM MEME-trimming plus undersampling/weighting.
- The `{protein}_training.txt` file is already the preprocessed template consumed by `upbm_regressor.py` (e.g. test_data `Gata4_training.txt`) — launch the regressor directly instead of re-balancing.
- The goal is in vivo TFBS discovery in yeast (DNAffinity predictions combined with yeast G1 nucleosome maps) rather than rebuilding an in vitro uPBM training set.

## Inputs

- Per-TF uPBM training file `proteins/{protein}/{protein}_training.txt` containing 35-mer sequences with experimental binding affinities (DREAM5, 66 mouse TFs)
- MEME suite motif result for the TF (defines the motif region used to trim the 35-mers)
- PWM-derived high-affinity motif used to flag sequences for 10x weighting

## Outputs

- Trimmed, undersampled, 10x-weighted per-TF training set of 35-mers with MD-derived DNA equilibrium geometry and flexibility features
- Trained DNAffinity random-forest regressor per TF
- Determination coefficient (R2) on the held-out 20% per TF and the average R2 across TFs (benchmark 0.69 ± 0.17)

## How to apply

Load the per-TF uPBM training file `proteins/{protein}/{protein}_training.txt` (35-mer sequences with experimental binding affinities; DREAM5 covers 66 mouse TFs) and trim each sequence to the motif region using the MEME suite result, which diminishes sequence variability. Balance the set by undersampling low-affinity k-mers so weak binders do not dominate, and assign 10x weight to sequences matching the PWM-derived high-affinity motif. Train the DNAffinity random-forest regressor — features are MD-derived DNA equilibrium geometry and flexibility parameters at base-pair step resolution — on 80% of the trimmed, weighted sequences. Predict affinities for the held-out 20% and compute the determination coefficient (R2) between predictions and experimental values, repeating per TF and averaging (benchmark: R2 = 0.69 ± 0.17). Rationale: uPBM libraries are dominated by weak binders, so weighting motif-matching sequences 10x and trimming to the MEME motif keeps training concentrated on the high-affinity regime that defines the PWM.

## Related tools

- **DNAffinity** (Provides the random-forest regressor (upbm_regressor.py) that consumes the undersampled, 10x-weighted {protein}_training.txt and trains on MD-derived DNA equilibrium geometry and flexibility parameters) — https://github.com/Jalbiti/DNAffinity
- **MEME suite** (Motif discovery whose result defines the motif region for trimming uPBM 35-mers and the PWM-derived high-affinity motif receiving 10x weight)

## Examples

```
python upbm_regressor.py Gata4
```

## Evaluation signals

- Trimmed sequences should align to the MEME-derived motif region, with reduced variability relative to the original 35-mers.
- PWM-matching sequences carry 10x weight and low-affinity k-mers are undersampled — inspect the weighted training distribution before fitting.
- Data split is 80% training / 20% testing; the determination coefficient (R2) must be computed only on the held-out 20%.
- Average R2 across the 66 DREAM5 mouse TFs should land near the reported 0.69 ± 0.17.
- Input files must match the `{protein}_training.txt` template format expected by upbm_regressor.py (compare with the test_data templates).

## Limitations

- Reported average R2 = 0.69 ± 0.17 hides large per-TF variability; individual TFs can fit far worse than the average.
- The transcription factor binding site concept is not uniquely defined and depends on the intrinsically noisy, low-resolution experimental technique, so uPBM-tuned undersampling/weights may not transfer to gcPBM or HT-SELEX.
- The README requires all required files to be present with template-matching formatting; malformed `{protein}_training.txt` files will break the regressor run.

## Evidence

- [other] Undersampling and 10x PWM weighting of the training set: [evidence span withheld — anchor status: not_found]
