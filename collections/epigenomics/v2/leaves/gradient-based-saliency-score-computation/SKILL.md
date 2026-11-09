---
name: gradient-based-saliency-score-computation
description: Use when you have a trained deepC checkpoint (e.g. GM12878) and hg19
  sequence, and need a non-negative per-base track of sequence importance underlying
  predicted chromatin interactions.
when_to_use_negative:
- You need comparative benchmarking of prediction accuracy between models (e.g. deepC
  vs Akita) rather than per-base importance — the paper defers such comparison to
  future work
- You need chromatin interaction predictions themselves rather than base-resolution
  sequence determinants (use standard deepC prediction instead)
- Your model is not a deepC/TensorFlow sequence model — the saliency procedure assumes
  gradients are computable with respect to the one-hot sequence input
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: deepC
  role: Sequence-based deep learning model whose output gradients with respect to
    the one-hot input yield the per-base saliency scores; imported in python as deepCregr
  repo: github.com/rschwess/deepC
- name: TensorFlow 2.1
  role: Provides automatic differentiation (GradientTape) to compute the gradient
    of the deepC output with respect to the sequence input
- name: pysam
  role: Retrieves hg19 genomic sequence and writes the per-base score bedGraph track
  repo: pysam-developers/pysam
- name: pybedtools
  role: Genomic interval/sequence handling for the 1 Mb + bin windows and bedGraph
    output
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-020-0960-3
    title: 'DeepC: predicting 3D genome folding using megabase-scale transfer learning'
  source_content_hash: sha256:369e117cded984b072a21d923c4e6a4099ed6d8c54dc5230a2453796f002c427
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - TensorFlow 2.1
  - deepC
  - pybedtools
  - pysam
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# gradient-based-saliency-score-computation

## Summary

Compute per-base saliency scores with deepC by taking the gradient of the model's chromatin interaction prediction with respect to the one-hot encoded megabase-scale DNA sequence input, quantifying how much each individual base pair contributes to predicting 3D genome folding.

## When to use

Use when you have a trained deepC checkpoint (e.g. GM12878) and hg19 sequence, and need a non-negative per-base track of sequence importance underlying predicted chromatin interactions.

## When NOT to use

- You need comparative benchmarking of prediction accuracy between models (e.g. deepC vs Akita) rather than per-base importance — the paper defers such comparison to future work
- You need chromatin interaction predictions themselves rather than base-resolution sequence determinants (use standard deepC prediction instead)
- Your model is not a deepC/TensorFlow sequence model — the saliency procedure assumes gradients are computable with respect to the one-hot sequence input

## Inputs

- Trained deepC model checkpoint (e.g. tissue-specific GM12878 model from task_001)
- hg19 genomic sequence for 1 Mb + bin windows (via pysam/pybedtools)
- One-hot encoded A/C/G/T sequence tensors of shape (window_length, 4)

## Outputs

- Per-base non-negative saliency score track averaged over overlapping sliding windows
- bedGraph file (chrom, start, end, score columns)
- bigWig file converted from the bedGraph

## How to apply

Load the trained deepC model from within a python script via `import deepCregr`, using a TensorFlow-compatible deepC version (deepC supports TensorFlow v1.8, 1.14 and 2.1) with GPU support for speed. Retrieve hg19 genomic sequence for successive 1 Mb + bin windows with pysam/pybedtools and one-hot encode the A/C/G/T bases. Run a forward pass and, using TensorFlow automatic differentiation, compute the gradient of the model output with respect to the one-hot sequence input — this gradient estimates how much the interaction prediction depends on each single base pair. Take the element-wise dot product of gradient and input, average over the overlapping sliding (1 Mb + bin) windows, and take the absolute value so all per-base scores are non-negative. Write the result as a bedGraph track (chrom, start, end, score) with pysam/pybedtools and convert it to bigWig.

## Related tools

- **deepC** (Sequence-based deep learning model whose output gradients with respect to the one-hot input yield the per-base saliency scores; imported in python as deepCregr) — github.com/rschwess/deepC
- **TensorFlow 2.1** (Provides automatic differentiation (GradientTape) to compute the gradient of the deepC output with respect to the sequence input)
- **pysam** (Retrieves hg19 genomic sequence and writes the per-base score bedGraph track) — pysam-developers/pysam
- **pybedtools** (Genomic interval/sequence handling for the 1 Mb + bin windows and bedGraph output)

## Examples

```
import deepCregr; preds = deepCregr.predict(one_hot_seq); grads = tf.GradientTape()  # gradient of deepC output w.r.t. one-hot input
```

## Evaluation signals

- Output file exists and parses as a valid bedGraph with chrom, start, end, and score columns present
- All saliency scores are non-negative (value_in_range >= 0), since absolute value is applied to the gradient × input dot product
- Score track length matches the summed length of the queried 1 Mb + bin hg19 windows (one score per base pair)
- Scores were averaged across overlapping sliding windows before taking the absolute value, so no window-boundary discontinuities beyond expected smoothing artifacts
- bigWig conversion succeeds and spot values in the bigWig match the bedGraph

## Limitations

- deepC was developed under TensorFlow v1.8 and only supports v1.8, 1.14 and 2.1 — other versions are untested, so the deepC compatibility version must match the installed TensorFlow
- Training requires GPU support and predictions run significantly faster with a GPU; saliency over megabase-scale windows is similarly compute-intensive
- Saliency reflects gradient sensitivity of the prediction, not a causal measurement of regulatory activity
- deepC requires additional python modules with pinned versions (numpy >= 1.16.4, pysam ~0.15.2, pybedtools plus a compatible bedtools install)

## Evidence
