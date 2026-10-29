---
name: gpu-accelerated-deep-learning-inference
description: Use when a trained deep learning model must be applied to thousands of
  inputs (e.g. ~96,805 genomic elements in a deletion screen) and CPU inference would
  be prohibitively slow.
when_to_use_negative:
- Only a handful of predictions are needed and no GPU environment is set up — the
  article states predictions are feasible without a GPU, just slower
- The task is model training rather than inference — training deepC models essentially
  requires GPU support for hours to days, so use the training-specific workflow instead
- The installed TensorFlow version is untested (the article lists only v1.8, 1.14
  and 2.1 as supported; other versions have not been tested)
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: deepC
  role: Deep neural network whose predictions (via `import deepCregr`) are executed
    for inference on sequence windows
  repo: https://github.com/rschwess/deepC
- name: TensorFlow 2.1
  role: Recommended GPU-enabled deep learning runtime for running deepC predictions;
    deepC was developed under v1.8 and supports 1.8, 1.14 and 2.1
- name: pysam
  role: Extracts megabase-scale reference and deletion sequence windows from the reference
    genome to feed the model
  repo: https://github.com/pysam-developers/pysam
- name: pybedtools
  role: Handles the BED files of CTCF/DNase peaks and shuffled background sites for
    which predictions are generated
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41592-020-0960-3
    title: 'DeepC: predicting 3D genome folding using megabase-scale transfer learning'
  source_content_hash: sha256:0b4ad69eb92458dada275bebcf906f8e96c74b923e66acd734bd75651a49fc0a
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
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# gpu-accelerated-deep-learning-inference

## Summary

Run deep learning model predictions (e.g. deepC chromatin interaction maps from megabase-scale DNA sequence) on GPU-enabled TensorFlow to make large-scale inference like genome-wide deletion screens tractable.

## When to use

Use when a trained deep learning model must be applied to thousands of inputs (e.g. ~96,805 genomic elements in a deletion screen) and CPU inference would be prohibitively slow.

## When NOT to use

- Only a handful of predictions are needed and no GPU environment is set up — the article states predictions are feasible without a GPU, just slower
- The task is model training rather than inference — training deepC models essentially requires GPU support for hours to days, so use the training-specific workflow instead
- The installed TensorFlow version is untested (the article lists only v1.8, 1.14 and 2.1 as supported; other versions have not been tested)

## Inputs

- Trained deepC model checkpoint (e.g. GM12878 model from task_001)
- Megabase-scale DNA sequence windows (reference and deletion-variant) extracted from the reference genome
- BED file of element coordinates (e.g. merged ENCODE GM12878 CTCF n=45,635 and DNase n=47,320 peaks plus 3,850 shuffled background sites)

## Outputs

- Predicted chromatin interaction maps per input sequence window (reference and deletion variants)
- Per-element scores such as mean absolute difference between reference and deletion predictions, written to e.g. deletion_screen_results.csv (~96,805 rows)

## How to apply

Install TensorFlow with GPU support — the article recommends TensorFlow 2.1 (deepC was developed under v1.8 and also supports 1.14), easiest via the TensorFlow docker containers which package the correct CUDA and cuDNN versions. Select the deepC compatibility version matching the installed TensorFlow version, then within a Python script import the model as `import deepCregr` and load the trained checkpoint (e.g. the GM12878 model from task_001). Construct prediction inputs (e.g. reference and deletion megabase-scale sequence windows extracted with pysam) and run batched predictions on the GPU; the article notes that running predictions is feasible without a GPU but significantly faster with one. Collect per-input outputs (predicted chromatin interaction maps) for downstream scoring such as mean absolute differences between reference and deletion predictions.

## Related tools

- **deepC** (Deep neural network whose predictions (via `import deepCregr`) are executed for inference on sequence windows) — https://github.com/rschwess/deepC
- **TensorFlow 2.1** (Recommended GPU-enabled deep learning runtime for running deepC predictions; deepC was developed under v1.8 and supports 1.8, 1.14 and 2.1)
- **pysam** (Extracts megabase-scale reference and deletion sequence windows from the reference genome to feed the model) — https://github.com/pysam-developers/pysam
- **pybedtools** (Handles the BED files of CTCF/DNase peaks and shuffled background sites for which predictions are generated)

## Examples

```
import deepCregr; model = deepCregr.model(paths=["GM12878_checkpoint.h5"]); pred = model.predict(seq_windows)
```

## Evaluation signals

- Confirm the TensorFlow version is one of the tested versions (1.8, 1.14, 2.1) and that GPU support is active (CUDA/cuDNN correct) before running
- Verify the per-element output table (deletion_screen_results.csv) has the expected row count (~96,805) and required fields: coordinates, chromatin class, mean absolute interaction difference
- Check that predicted interaction maps for reference and deletion windows have matching dimensions so mean absolute differences are well-defined
- Confirm the deletion screen output reproduces the reference pattern of Figure 3b: CTCF sites and active promoters/enhancers show the strongest predicted deletion effects
- Shuffled background sites should score lower than CTCF sites and active regulatory elements, serving as a negative control

## Limitations

- Other TensorFlow versions beyond 1.8, 1.14 and 2.1 have not been tested with deepC
- Running predictions without GPU support is significantly slower; GPU support is preferable for predictions and essential for training
- deepC requires specific Python module versions (numpy v1.16.4+, pysam tested with v0.15.2, pybedtools with a compatible bedtools install)

## Evidence
