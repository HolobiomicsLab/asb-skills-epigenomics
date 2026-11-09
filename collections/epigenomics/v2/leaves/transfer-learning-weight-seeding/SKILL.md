---
name: transfer-learning-weight-seeding
description: Use when training a sequence-based deep learning model on scarce, low-depth
  Hi-C targets and pre-trained chromatin feature weights are available to seed hidden
  layers before fine-tuning on interaction prediction.
when_to_use_negative:
- You need to train the chromatin feature predictor itself from labels — this skill
  covers the second-phase interaction training, not DeepSEA-style feature pre-training
- You lack the pre-seeded chromatin feature weights; deepC models yield substantially
  better predictions when pre-seeded, and training from scratch forfeits this advantage
- Your TensorFlow version is outside the tested set (v1.8, 1.14, 2.1) with no matching
  deepC compatibility branch
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3474
tools:
- name: deepC
  role: Framework providing the chromatin interaction network whose hidden layers
    are seeded with pre-trained chromatin feature weights; import as deepCregr in
    Python scripts
  repo: github.com/rschwess/deepC
- name: TensorFlow 2.1
  role: Deep learning backend used for both pre-training the chromatin feature network
    and fine-tuning the seeded interaction model; requires GPU support for training
- name: ibab/tensorflow-wavenet
  role: Source of the dilated convolution implementation used in the ten dilated gated
    1D conv layers of the interaction network trained from scratch during seeding
  repo: github.com/ibab/tensorflow-wavenet
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-020-0960-3
    title: 'DeepC: predicting 3D genome folding using megabase-scale transfer learning'
  source_content_hash: sha256:d2ec86075c6139908fc79524158151e6e81768ede4de7d12f9edba5df26a9d6e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - TensorFlow 2.1
  - deepC
  - ibab/tensorflow-wavenet
  - pybedtools
  - pysam
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# transfer-learning-weight-seeding

## Summary

Two-phase transfer learning in which deepC's convolutional module is initialized with weights pre-trained to predict ~936 ENCODE/Roadmap chromatin features, then fine-tuned alongside from-scratch dilated gated conv layers to predict Hi-C chromatin interactions from megabase-scale DNA sequence.

## When to use

Use when training a sequence-based deep learning model on scarce, low-depth Hi-C targets and pre-trained chromatin feature weights are available to seed hidden layers before fine-tuning on interaction prediction.

## When NOT to use

- You need to train the chromatin feature predictor itself from labels — this skill covers the second-phase interaction training, not DeepSEA-style feature pre-training
- You lack the pre-seeded chromatin feature weights; deepC models yield substantially better predictions when pre-seeded, and training from scratch forfeits this advantage
- Your TensorFlow version is outside the tested set (v1.8, 1.14, 2.1) with no matching deepC compatibility branch

## Inputs

- Pre-trained chromatin feature network weights (DeepSEA-adapted conv module trained on ~936 ENCODE/Roadmap chromatin feature datasets)
- 5 kb Hi-C skeleton interaction targets derived from Rao et al. Hi-C (GEO GSE63525) for GM12878 and K562
- Megabase-scale DNA sequence windows

## Outputs

- Trained chromatin interaction model checkpoints saved in HDF5 format (~60M parameters)
- Training logs recording epochs and loss

## How to apply

First train (or obtain) a DeepSEA-adapted chromatin feature network on the ENCODE/Roadmap compendium of ~936 chromatin feature datasets using TensorFlow with GPU support. Build the chromatin interaction network — a conv module plus ten dilated gated 1D conv layers (~60M parameters, dilated convolution adapted from tensorflow-wavenet) — and seed its hidden layers with the trained chromatin feature network weights, effectively transferring learned sequence patterns while the dilated module and fully connected interpretation layer train from scratch. Fine-tune the seeded network on 5 kb Hi-C skeleton targets derived from Rao et al. Hi-C (GEO GSE63525) for GM12878 and K562, running for several hours up to days on GPU depending on dataset size and epochs. Match the deepC compatibility branch (e.g. tensorflow2.1plus_compatibility_version) to your installed TensorFlow release (v1.8, 1.14 or 2.1), and save checkpoints in HDF5 format with training logs recording epochs and loss.

## Related tools

- **deepC** (Framework providing the chromatin interaction network whose hidden layers are seeded with pre-trained chromatin feature weights; import as deepCregr in Python scripts) — github.com/rschwess/deepC
- **TensorFlow 2.1** (Deep learning backend used for both pre-training the chromatin feature network and fine-tuning the seeded interaction model; requires GPU support for training)
- **ibab/tensorflow-wavenet** (Source of the dilated convolution implementation used in the ten dilated gated 1D conv layers of the interaction network trained from scratch during seeding) — github.com/ibab/tensorflow-wavenet

## Examples

```
pip install pysam pybedtools; git clone https://github.com/rschwess/deepC && git checkout tensorflow2.1plus_compatibility_version; import deepCregr
```

## Evaluation signals

- Checkpoint files exist and load as valid HDF5 model files
- Training script runs end to end on GPU without errors
- Saved network's parameter count is within tolerance of the expected ~60M parameters
- Seeded model yields substantially better interaction predictions than an unseeded baseline (per the paper's pre-seeding finding)
- Training logs record epochs and loss progressing over hours-to-days of GPU training

## Limitations

- Training requires GPU support and takes several hours up to days depending on dataset size and epochs
- Only TensorFlow v1.8, 1.14 and 2.1 have been tested; other versions are untested and require selecting the matching deepC compatibility branch
- Comparative benchmarking against Akita was deferred to future work, so no direct head-to-head performance baseline is provided

## Evidence
