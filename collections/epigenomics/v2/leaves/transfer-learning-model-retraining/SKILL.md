---
name: transfer-learning-model-retraining
description: Use when you have Hi-C contact matrices (e.g. HiC-Pro output from GM12878
  GSE63525 or downsampled valid-contact subsets) and need a sequence-based chromatin
  model retrained on that depth, starting from the chromatin-feature-seeded base model.
when_to_use_negative:
- You only need a Hi-C model for deeply sequenced tissue data already covered by the
  published pre-trained deepC models — retraining is for new depths or new tissue-specific
  Hi-C training sets
- You want a direct deepC-vs-Akita benchmark — the authors explicitly defer comparative
  consolidation between these models to future work and provide no such benchmark
- Your Hi-C data has extremely low depth (on the order of the 10M-contact subset),
  where the model demonstrably fails to learn chromatin structure
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3678
- http://edamontology.org/topic_3474
tools:
- name: deepC
  role: Core tensorflow deep-learning framework; base model is downloaded from its
    ./models directory, retrained per depth, then imported via `import deepCregr`
    for sequence-based predictions
  repo: https://github.com/rschwess/deepC
- name: TensorFlow 2.1
  role: Backend with GPU support for training (essential) and prediction; deepC supports
    v1.8, 1.14 and 2.1 via matching compatibility branches
- name: pysam
  role: Additional python library required by the deepC environment for sequence/file
    handling during model preparation and prediction
  repo: https://github.com/pysam-developers/pysam
- name: pybedtools
  role: Additional python library required by the deepC environment for genomic interval
    handling during model preparation and prediction
- name: HiC-Pro
  role: Used per the Methods to realign and prepare GM12878 contact matrices downsampled
    to 1B/100M/10M valid contacts
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-020-0960-3
    title: 'DeepC: predicting 3D genome folding using megabase-scale transfer learning'
  source_content_hash: sha256:687861d26f1437846e87869a063f9b4f1ab219e4ea8a6c05c3bf76b523fb91da
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rschwess/deepC
  tools:
  - HiC-Pro
  - TensorFlow 2.1
  - deepC
  - pybedtools
  - pysam
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3678
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# transfer-learning-model-retraining

## Summary

Retrain deepC 5 kb-resolution chromatin-interaction models on user-supplied Hi-C data (e.g. downsampled GM12878 contact matrices) by initializing from the pre-trained chromatin-feature-seeded base model in the deepC repository, then evaluate predictions against holdout chromosomes with Pearson correlation. This tests how low sequencing depth can go before sequence-based structure prediction fails.

## When to use

Use when you have Hi-C contact matrices (e.g. HiC-Pro output from GM12878 GSE63525 or downsampled valid-contact subsets) and need a sequence-based chromatin model retrained on that depth, starting from the chromatin-feature-seeded base model.

## When NOT to use

- You only need a Hi-C model for deeply sequenced tissue data already covered by the published pre-trained deepC models — retraining is for new depths or new tissue-specific Hi-C training sets
- You want a direct deepC-vs-Akita benchmark — the authors explicitly defer comparative consolidation between these models to future work and provide no such benchmark
- Your Hi-C data has extremely low depth (on the order of the 10M-contact subset), where the model demonstrably fails to learn chromatin structure

## Inputs

- Hi-C contact matrices prepared with HiC-Pro from GM12878 (GSE63525) downsampled to 1 billion, 100 million and 10 million valid contacts
- Pre-trained chromatin-feature-seeded base model from the ./models directory of the deepC repository
- deepC repository (github.com/rschwess/deepC) with core python code, R scripts and tutorials

## Outputs

- Retrained 5 kb-resolution deepC models, one per contact-depth training set
- Sequence-based predictions over chr16/17 holdout windows
- CSV of per-model mean Pearson correlations on chr16/17 (expected ~0.46 for 1B, ~0.4 for 100M, failure for 10M)

## How to apply

Prepare contact matrices with HiC-Pro from the GSE63525 GM12878 training data at the target depth (e.g. 1B/100M/10M valid contacts). Set up the deepC environment: install TensorFlow with GPU support (tensorflow 2.1 recommended with CUDA 10.1; deepC supports v1.8, 1.14 and 2.1 — pick the deepC compatibility branch matching the installed version), plus pysam and pybedtools, and clone the rschwess/deepC repository. Download the pre-trained chromatin-feature-seeded base model from the ./models directory of the deepC repository — this transfer-learning pre-seeding is what allows deepC to learn from low-depth, low-resolution Hi-C — then retrain 5 kb-resolution models on each depth's contact set (several hours up to days per model, GPU essential). Run predictions over chr16/17 holdout windows via `import deepCregr` (GPU strongly recommended; CPU-only prediction takes ~2h versus ~5 min with a GPU) and compute mean Pearson correlations between predicted and observed interaction maps per model. Accept results if the 1B model reaches ~0.46 and the 100M model ~0.4 on chr16/17, matching Supplementary Figure 8, while the 10M model fails to learn chromatin structure.

## Related tools

- **deepC** (Core tensorflow deep-learning framework; base model is downloaded from its ./models directory, retrained per depth, then imported via `import deepCregr` for sequence-based predictions) — https://github.com/rschwess/deepC
- **TensorFlow 2.1** (Backend with GPU support for training (essential) and prediction; deepC supports v1.8, 1.14 and 2.1 via matching compatibility branches)
- **pysam** (Additional python library required by the deepC environment for sequence/file handling during model preparation and prediction) — https://github.com/pysam-developers/pysam
- **pybedtools** (Additional python library required by the deepC environment for genomic interval handling during model preparation and prediction)
- **HiC-Pro** (Used per the Methods to realign and prepare GM12878 contact matrices downsampled to 1B/100M/10M valid contacts)

## Examples

```
import deepCregr; preds = deepCregr.predict(chr16_windows, model='retrained_1B_model')
```

## Evaluation signals

- The retrained 1B-depth model yields a mean Pearson correlation of ~0.46 with observed chromatin interaction maps on holdout chromosomes 16 and 17 (value_in_range)
- The 100M-depth model yields a mean Pearson correlation of ~0.4 on chr16/17 (value_in_range)
- The 10M-depth model shows failure to learn chromatin structure, consistent with Supplementary Figure 8 (output_matches_reference)
- The deepC compatibility branch used matches the installed tensorflow version (v1.8, 1.14 or 2.1; other versions untested)
- Prediction runs over chr16/17 holdout windows complete in ~5 min on GPU; a ~2h CPU-only run signals GPU support is misconfigured

## Limitations

- Training requires GPU support and takes several hours up to days depending on the dataset and epochs aimed for; predictions without GPU take on the order of 2h versus ~5 min with a GPU
- deepC only supports tensorflow v1.8, 1.14 and 2.1 — other versions have not been tested
- Very low sequencing depth (10M valid contacts) is below the limit at which the model can learn chromatin structure
- No comparative benchmark between deepC and Akita is presented; the authors defer this to future work

## Evidence
