---
name: one-hot-dna-sequence-encoding
description: Use when a neural model such as deepC requires one-hot encoded A/C/G/T
  input built from hg19 reference sequence over megabase-scale windows, e.g. for saliency
  scoring or interaction prediction.
when_to_use_negative:
- Input is already a one-hot encoded sequence track or numeric feature matrix rather
  than raw hg19 FASTA plus interval coordinates
- The target sequence is not A/C/G/T reference DNA (e.g. pre-collapsed consensus,
  protein, or multi-base variant haplotypes) where single-base one-hot encoding does
  not apply
- The downstream model expects a different window length or base alphabet than deepC's
  (1 Mb + bin) windows of A/C/G/T
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: pysam
  role: Retrieves hg19 genomic sequence for the 1 Mb + bin windows to be one-hot encoded
  repo: https://github.com/pysam-developers/pysam
- name: pybedtools
  role: Python library installed alongside pysam for handling genomic intervals and
    writing downstream bedGraph tracks from windowed encoding results
- name: deepC
  role: Deep neural network that consumes the one-hot encoded megabase-scale sequence
    as input; imported from within a python script as `import deepCregr`
  repo: https://github.com/rschwess/deepC
- name: TensorFlow
  role: Backend framework (v2.1 recommended; v1.8, 1.14 also supported) under which
    the one-hot sequence tensors are passed through deepC and gradients are taken
    with respect to the input
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-020-0960-3
    title: 'DeepC: predicting 3D genome folding using megabase-scale transfer learning'
  source_content_hash: sha256:cf3b0f81cdee111fbd1955a188bf98cb9bb176ed384eee82a6ca9d5e138ac83e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/pysam-developers/pysam
  tools:
  - TensorFlow
  - deepC
  - pybedtools
  - pysam
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# one-hot-dna-sequence-encoding

## Summary

Retrieve hg19 genomic sequence in successive (1 Mb + bin) sliding windows with pysam/pybedtools and one-hot encode the A/C/G/T bases into the 4-channel input format required by deepC, which predicts chromatin interactions from megabase-scale DNA sequence.

## When to use

Use when a neural model such as deepC requires one-hot encoded A/C/G/T input built from hg19 reference sequence over megabase-scale windows, e.g. for saliency scoring or interaction prediction.

## When NOT to use

- Input is already a one-hot encoded sequence track or numeric feature matrix rather than raw hg19 FASTA plus interval coordinates
- The target sequence is not A/C/G/T reference DNA (e.g. pre-collapsed consensus, protein, or multi-base variant haplotypes) where single-base one-hot encoding does not apply
- The downstream model expects a different window length or base alphabet than deepC's (1 Mb + bin) windows of A/C/G/T

## Inputs

- hg19 reference genome FASTA (indexed for pysam FastaFile access)
- genomic interval coordinates (chrom, start, end) for successive 1 Mb + bin sliding windows
- trained deepC checkpoint compatible with the one-hot input format (e.g. task_001-trained GM12878 model)

## Outputs

- one-hot encoded sequence tensor of shape (window_length, 4) per (1 Mb + bin) window
- sequence input tensor ready for deepC forward pass / gradient computation

## How to apply

Fetch hg19 sequence for successive (1 Mb + bin) windows using pysam FastaFile access (or pybedtools to slice intervals from the reference FASTA). Convert each base into a 4-channel one-hot vector over A/C/G/T, producing an input tensor of shape (window_length, 4) per sliding window. Feed the tensor to the deepC model loaded via `import deepCregr` (TensorFlow 2.1-compatible version, run on GPU where available) for a forward pass. For downstream saliency, keep the one-hot input attached to a tf.GradientTape so the gradient of the model output with respect to the input can be computed with TensorFlow automatic differentiation; the element-wise dot product of gradient with the one-hot input only works because of this encoding. Overlapping sliding windows are later averaged to obtain per-base tracks.

## Related tools

- **pysam** (Retrieves hg19 genomic sequence for the 1 Mb + bin windows to be one-hot encoded) — https://github.com/pysam-developers/pysam
- **pybedtools** (Python library installed alongside pysam for handling genomic intervals and writing downstream bedGraph tracks from windowed encoding results)
- **deepC** (Deep neural network that consumes the one-hot encoded megabase-scale sequence as input; imported from within a python script as `import deepCregr`) — https://github.com/rschwess/deepC
- **TensorFlow** (Backend framework (v2.1 recommended; v1.8, 1.14 also supported) under which the one-hot sequence tensors are passed through deepC and gradients are taken with respect to the input)

## Examples

```
import pysam; fa = pysam.FastaFile('hg19.fa'); seq = fa.fetch('chr1', start, start + 1000000)
```

## Evaluation signals

- Each window's encoded tensor has exactly 4 channels and one channel set per base position (valid A/C/G/T one-hot encoding)
- The sum of encoded window lengths and offsets reconstructs the requested hg19 interval coordinates exactly (no base loss or duplication at sliding-window overlaps)
- A forward pass through deepC on the encoded tensor produces a finite interaction prediction without shape errors
- Gradients computed with respect to the one-hot input yield finite values at every base position
- Downstream bedGraph/bigWig saliency scores derived from the encoded windows are all non-negative (value_in_range >= 0)

## Limitations

- deepC is only compatible with specific TensorFlow versions (1.8, 1.14, 2.1); other versions have not been tested, so the encoding-to-model pipeline may break under other TF releases
- The encoding assumes hg19 genomic coordinates and (1 Mb + bin) window lengths as used by deepC; other assemblies or window scales are not validated here
- Saliency values derived from the encoding are only meaningful after averaging over overlapping sliding windows; single-window scores are not the final per-base track

## Evidence
