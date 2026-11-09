---
name: random-forest-dna-affinity-regression
description: Use when predicting TF binding affinities from uPBM (35-mer oligos),
  gcPBM, or HT-SELEX datasets with a random forest over MD-derived DNA shape/flexibility
  features rather than sequence-only CNNs like DeepBind.
when_to_use_negative:
- Input is raw sequences without experimental affinity labels — the regressor requires
  a `*_training.txt` file pairing k-mers with measured affinities.
- HT-SELEX data displays inconsistency between enrichments at different cycles — apply
  the SVM filtering step first instead of training on suspicious cases.
- Goal is in vivo TFBS prediction without chromatin context — the paper warns that
  factors beyond intrinsic TF affinity operate in vivo, so predictions must be complemented
  with nucleosome positioning maps.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0160
tools:
- name: DNAffinity
  role: Supplies the random forest regressor and the MD-derived geometry/flexibility
    feature database; entry points upbm_regressor.py, selex_regressor.py, gcpbm_regressor.py
    take the protein name as 1st argument
  repo: https://github.com/Jalbiti/DNAffinity
- name: MEME suite
  role: Motif discovery used to trim uPBM 35-mer sequences to the motif region, reducing
    sequence variability before training
- name: DNAShapeR
  role: Shape-based ML regressor baseline used to benchmark DNAffinity R2 on the same
    TFs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkac708
    title: 'DNAffinity: a machine-learning approach to predict DNA binding affinities
      of transcription factors'
  source_content_hash: sha256:eff23fba0d4abf1549022e287652e68d3b816182b89ad3f146656cb8ca6f18ff
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
  - DNAShapeR
  - DNAffinity
  - MEME suite
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# random-forest-dna-affinity-regression

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train a random forest regressor on MD-derived DNA equilibrium geometry and flexibility parameters (base-pair step resolution) to reproduce in vitro transcription factor binding affinities from uPBM, gcPBM, or HT-SELEX data, evaluating the determination coefficient (R2) on a held-out 20% of each dataset.

## When to use

Use when predicting TF binding affinities from uPBM (35-mer oligos), gcPBM, or HT-SELEX datasets with a random forest over MD-derived DNA shape/flexibility features rather than sequence-only CNNs like DeepBind.

## When NOT to use

- Input is raw sequences without experimental affinity labels — the regressor requires a `*_training.txt` file pairing k-mers with measured affinities.
- HT-SELEX data displays inconsistency between enrichments at different cycles — apply the SVM filtering step first instead of training on suspicious cases.
- Goal is in vivo TFBS prediction without chromatin context — the paper warns that factors beyond intrinsic TF affinity operate in vivo, so predictions must be complemented with nucleosome positioning maps.

## Inputs

- uPBM per-TF training file `proteins/{protein}/{protein}_training.txt` listing 35-mer oligos with experimental binding affinities (e.g. DREAM5, 66 mouse TFs)
- HT-SELEX per-cycle files `proteins/{protein}/{cycle}.txt`
- gcPBM files `proteins/{protein}/{protein}_{concentration}.txt` plus `proteins/{protein}/_freq_matrix_6.txt`
- MD-derived DNA equilibrium geometry and flexibility parameters at base-pair step resolution from the DNAffinity feature database
- MEME suite motif result for uPBM sequence trimming; PWM-derived high-affinity motif for sequence weighting

## Outputs

- Trained per-TF random forest regressor predicting binding affinities for held-out k-mers
- Determination coefficient (R2) between predicted and experimental affinities, per TF and averaged (e.g. 0.69 ± 0.17 on uPBM, 0.93 ± 0.02 on gcPBM)

## How to apply

Represent every k-mer with features describing DNA conformation at base-pair step resolution — equilibrium geometry plus flexibility (Olson definition) taken from the MD-derived parameter database shipped with DNAffinity on GitHub. Preprocess per technique: for uPBM, trim the 35-mer sequences to the motif region using the MEME suite result to reduce sequence variability, then balance the training set by undersampling low-affinity k-mers and assigning 10x weight to sequences matching the PWM-derived high-affinity motif; for HT-SELEX, first use SVM to remove cases with inconsistent enrichments across cycles. Train the random forest (e.g. run `upbm_regressor.py {protein}` with `proteins/{protein}/{protein}_training.txt`, protein name as 1st argument) on 80% of the trimmed, weighted sequences. Compute the determination coefficient (R2) between predictions and experimental values on the remaining 20%; expected benchmarks are average R2 = 0.69 ± 0.17 across 66 mouse TFs on uPBM and 0.93 ± 0.02 on gcPBM. For in vivo extension, train on gcPBM (e.g. CBF1, R2 = 0.80) and score PB-exo/ChIP-exo peaks along the yeast genome after combining predictions with G1-phase nucleosome maps.

## Related tools

- **DNAffinity** (Supplies the random forest regressor and the MD-derived geometry/flexibility feature database; entry points upbm_regressor.py, selex_regressor.py, gcpbm_regressor.py take the protein name as 1st argument) — https://github.com/Jalbiti/DNAffinity
- **MEME suite** (Motif discovery used to trim uPBM 35-mer sequences to the motif region, reducing sequence variability before training)
- **DNAShapeR** (Shape-based ML regressor baseline used to benchmark DNAffinity R2 on the same TFs)

## Examples

```
python upbm_regressor.py Gata4
```

## Evaluation signals

- Held-out R2 per TF on the 20% test split: the average across 66 mouse uPBM TFs should land near 0.69 ± 0.17 (gcPBM near 0.93 ± 0.02).
- Split invariant: regressor trained on exactly 80% of the trimmed, weighted sequences; R2 computed only on the remaining 20%.
- Preprocessing audit: uPBM sequences trimmed to the MEME-derived motif region; training set balanced by undersampling low-affinity k-mers with 10x weight on PWM-matching sequences.
- File-format check: `proteins/{protein}/{protein}_training.txt` present and formatted like the test_data templates (e.g. Gata4_training.txt) before running upbm_regressor.py.
- For HT-SELEX runs, verify SVM removed cases with inconsistent cross-cycle enrichments before training.

## Limitations

- uPBM reproducibility is markedly lower than gcPBM (average R2 0.69 ± 0.17 vs 0.93 ± 0.02), reflecting the noisier uPBM technique.
- The concept of a transcription factor binding site is not uniquely defined — it depends on the intrinsically noisy, low-resolution experimental technique used to detect it, preventing a universal predictor.
- Even though DNAffinity predicts many potential binding sites, no experimental evidence of in vivo binding exists for many of them; in vivo extrapolation requires chromatin structure (nucleosome maps) beyond intrinsic affinity.

## Evidence
