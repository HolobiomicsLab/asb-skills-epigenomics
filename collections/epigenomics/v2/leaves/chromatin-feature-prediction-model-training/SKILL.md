---
name: chromatin-feature-prediction-model-training
description: Use when the goal is to pre-train a sequence-based convolutional network
  on chromatin feature datasets (e.g. ENCODE/Roadmap marks such as CTCF, DNase, ChIP-seq)
  as the first phase of deepC transfer learning for chromatin interaction prediction.
when_to_use_negative:
- Your target is already a trained chromatin interaction model — this skill covers
  only the chromatin feature pre-training phase, not the second-phase fine-tuning
  on 5 kb Hi-C skeleton targets from Rao et al. Hi-C.
- Your TensorFlow version is not v1.8, 1.14, or 2.1 — the article states other versions
  have not been tested and the deepC compatibility branch must match the installed
  release.
- You only need to run predictions with an already-trained deepC model — predictions
  do not require this training phase, though a GPU still speeds them up.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: deepC
  role: Framework providing the DeepSEA-adapted chromatin feature network and training
    code; imported in Python as `import deepCregr`
  repo: https://github.com/rschwess/deepC
- name: TensorFlow 2.1
  role: Deep learning backend used with GPU support (via docker containers with packaged
    CUDA/cuDNN) to train the chromatin feature network
- name: pysam
  role: Python module installed as a deepC dependency for handling sequence/alignment
    data during training preparation
  repo: https://github.com/pysam-developers/pysam
- name: pybedtools
  role: Python library installed as a deepC dependency for genomic interval operations
    during training data preparation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-020-0960-3
    title: 'DeepC: predicting 3D genome folding using megabase-scale transfer learning'
  source_content_hash: sha256:86048fd856d671525af527078bec2ae2ff1ddd5935e3d4ba2c444b1dad74284e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rschwess/deepC
  tools:
  - TensorFlow 2.1
  - deepC
  - pybedtools
  - pysam
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# chromatin-feature-prediction-model-training

## Summary

Trains a DeepSEA-adapted convolutional network to predict a compendium of ~936 ENCODE/Roadmap chromatin feature datasets, producing the pre-trained hidden-layer weights that are later transferred into the deepC chromatin interaction model via two-step transfer learning.

## When to use

Use when the goal is to pre-train a sequence-based convolutional network on chromatin feature datasets (e.g. ENCODE/Roadmap marks such as CTCF, DNase, ChIP-seq) as the first phase of deepC transfer learning for chromatin interaction prediction.

## When NOT to use

- Your target is already a trained chromatin interaction model — this skill covers only the chromatin feature pre-training phase, not the second-phase fine-tuning on 5 kb Hi-C skeleton targets from Rao et al. Hi-C.
- Your TensorFlow version is not v1.8, 1.14, or 2.1 — the article states other versions have not been tested and the deepC compatibility branch must match the installed release.
- You only need to run predictions with an already-trained deepC model — predictions do not require this training phase, though a GPU still speeds them up.

## Inputs

- ENCODE/Roadmap compendium of ~936 chromatin feature datasets (DNase, ChIP-seq, etc.)
- DNA sequence in megabase-scale windows processed via pysam/pybedtools-compatible genome resources
- GPU-enabled environment with TensorFlow (v1.8, 1.14, or 2.1) and matching CUDA/cuDNN

## Outputs

- Trained DeepSEA-adapted chromatin feature network weights (checkpoints in HDF5 format)
- Training logs recording epochs and loss
- Pre-trained convolutional module weights suitable for seeding the deepC chromatin interaction network

## How to apply

Set up TensorFlow with GPU support using the official TensorFlow docker containers (TensorFlow 2.1 recommended, with matching CUDA and cuDNN packaged), and install pysam and pybedtools via pip or bioconda. Clone the deepC repository and select the compatibility branch matching the installed TensorFlow release (e.g. the tensorflow2.1plus_compatibility_version branch for TensorFlow 2.1+). Train the DeepSEA-adapted chromatin feature network on the ENCODE/Roadmap compendium of ~936 chromatin feature datasets, running with GPU support for several hours up to days depending on dataset size and epochs. Save trained checkpoints in HDF5 format alongside training logs recording epochs and loss. The rationale is that hidden layers optimized to predict chromatin features learn the grammar of regulatory elements (e.g. CTCF-bound domain boundaries), which can then be seeded into the ~60M-parameter interaction network that trains the dilated gated 1D convolutions (adapted from wavenet) and fully connected layers from scratch on 5 kb Hi-C skeleton targets.

## Related tools

- **deepC** (Framework providing the DeepSEA-adapted chromatin feature network and training code; imported in Python as `import deepCregr`) — https://github.com/rschwess/deepC
- **TensorFlow 2.1** (Deep learning backend used with GPU support (via docker containers with packaged CUDA/cuDNN) to train the chromatin feature network)
- **pysam** (Python module installed as a deepC dependency for handling sequence/alignment data during training preparation) — https://github.com/pysam-developers/pysam
- **pybedtools** (Python library installed as a deepC dependency for genomic interval operations during training data preparation)

## Examples

```
pip install pybedtools; pip install pysam
```

## Evaluation signals

- Verify the saved checkpoint files exist and load as valid HDF5 model files
- Confirm the training script runs end to end on GPU without error
- Check that the resulting deepC network (conv module + ten dilated gated 1D conv layers) has a parameter count within tolerance of the expected ~60M parameters
- Confirm the installed TensorFlow version is one of the tested releases (v1.8, 1.14, 2.1) and the deepC compatibility branch matches it
- Confirm GPU is used during training, since GPU support is essential for training and runs can take hours up to days

## Limitations

- deepC was developed under TensorFlow v1.8 and only supports v1.8, 1.14 and 2.1; other versions have not been tested, so a matching compatibility branch must be chosen
- Training requires GPU support for several hours up to days depending on the dataset and epochs aimed for
- pybedtools additionally requires a compatible version of bedtools installed

## Evidence

- [methods] The tensorflow docker containers are the easiest way to set up tensorflow with GPU and come with the correct CUDA and cuDNN versions packaged.: "The tensorflow docker containers are the easiest way to set up tensorflow with GPU and come with the correct CUDA and cuDNN versions packaged."
- [discussion] We found deepC models to yield substantially better predictions when we pre-seeded the model with hidden layers optimized to predict a compendium of chromatin features.: "We found deepC models to yield substantially better predictions when we pre-seeded the model with hidden layers optimized to predict a compendium of chromatin features."
- [methods] Training deepC models requires running with GPU support for several hours (up to days depending on the dataset and epochs aimed for): "Training deepC models requires running with GPU support for several hours (up to days depending on the dataset and epochs aimed for)"
- [methods] deepC was developed under v1.8 and supports (v1.8, 1.14 and 2.1 other versions have not been tested): "deepC was developed under v1.8 and supports (v1.8, 1.14 and 2.1 other versions have not been tested)"
- [readme] GPU support is preferable for predictions and essential for training: "GPU support is preferable for predictions and essential for training"
