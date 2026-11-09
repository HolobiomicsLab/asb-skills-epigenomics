---
name: sliding-window-saliency-averaging
description: Use when you need to interpret which individual base pairs in a megabase-scale
  DNA sequence drive a trained deepC chromatin interaction prediction, e.g. fine-mapping
  sequence determinants of chromatin architecture at base pair resolution.
when_to_use_negative:
- The goal is TAD boundary calling from an existing Hi-C contact matrix rather than
  base-level model interpretation (use a Hi-C domain caller such as TopDom instead)
- The task is predicting chromatin interactions from sequence, not explaining an already-trained
  deepC prediction at single-base resolution
- The trained model checkpoint is not a deepC model loadable via `import deepCregr`
  under TensorFlow v1.8, v1.14 or v2.1, so gradients of the correct output cannot
  be obtained
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: deepC
  role: Sequence-based deep learning model whose output is differentiated to yield
    per-base saliency scores; loaded in python via `import deepCregr`
  repo: github.com/rschwess/deepC
- name: TensorFlow
  role: Provides automatic differentiation (GradientTape) to compute the gradient
    of the deepC output with respect to the one-hot sequence input; use a deepC-compatible
    version (v1.8, 1.14 or 2.1)
  repo: github.com/rschwess/deepC
- name: pysam
  role: Retrieves hg19 genomic sequence for successive 1 Mb + bin windows and helps
    write the per-base scores as bedGraph
  repo: pysam-developers/pysam
- name: pybedtools
  role: Handles the sliding (1 Mb + bin) windows and writing/conversion of the bedGraph
    score track
  repo: pysam-developers/pysam
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-020-0960-3
    title: 'DeepC: predicting 3D genome folding using megabase-scale transfer learning'
  source_content_hash: sha256:a1559d8bb4c116ed5396e271e727058d79b4db19893fac23767ca25527d3331a
schema_version: 0.2.0
metadata:
  license_tier: unknown
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

# sliding-window-saliency-averaging

## Summary

Computes per-base saliency scores for deepC chromatin-interaction predictions by differentiating the model output with respect to the one-hot DNA sequence input, then averaging gradients over overlapping sliding (1 Mb + bin) windows to produce a non-negative per-base importance track.

## When to use

Use when you need to interpret which individual base pairs in a megabase-scale DNA sequence drive a trained deepC chromatin interaction prediction, e.g. fine-mapping sequence determinants of chromatin architecture at base pair resolution.

## When NOT to use

- The goal is TAD boundary calling from an existing Hi-C contact matrix rather than base-level model interpretation (use a Hi-C domain caller such as TopDom instead)
- The task is predicting chromatin interactions from sequence, not explaining an already-trained deepC prediction at single-base resolution
- The trained model checkpoint is not a deepC model loadable via `import deepCregr` under TensorFlow v1.8, v1.14 or v2.1, so gradients of the correct output cannot be obtained

## Inputs

- Trained deepC model checkpoint (e.g. GM12878) loadable via `import deepCregr`
- hg19 genomic sequence for 1 Mb + bin windows (fetched with pysam/pybedtools)
- One-hot encoded A/C/G/T sequence tensors of shape (window_length, 4)

## Outputs

- Per-base non-negative saliency score track
- bedGraph file with chrom, start, end, score columns
- bigWig file converted from the bedGraph

## How to apply

Load a task_001-trained GM12878 deepC checkpoint from within a python script via `import deepCregr` using the TensorFlow 2.1-compatible deepC version on GPU where available. Retrieve hg19 genomic sequence for successive 1 Mb + bin windows with pysam/pybedtools and one-hot encode the A/C/G/T bases. Run a forward pass and compute the gradient of the deepC model output with respect to the one-hot sequence input using TensorFlow automatic differentiation. Take the element-wise dot product of the gradient with the one-hot input, average the saliency over the overlapping sliding (1 Mb + bin) windows, and take the absolute value to obtain a non-negative per-base score track. Write the scores as a bedGraph file (chrom, start, end, score) with pysam/pybedtools and convert to bigWig.

## Related tools

- **deepC** (Sequence-based deep learning model whose output is differentiated to yield per-base saliency scores; loaded in python via `import deepCregr`) — github.com/rschwess/deepC
- **TensorFlow** (Provides automatic differentiation (GradientTape) to compute the gradient of the deepC output with respect to the one-hot sequence input; use a deepC-compatible version (v1.8, 1.14 or 2.1)) — github.com/rschwess/deepC
- **pysam** (Retrieves hg19 genomic sequence for successive 1 Mb + bin windows and helps write the per-base scores as bedGraph) — pysam-developers/pysam
- **pybedtools** (Handles the sliding (1 Mb + bin) windows and writing/conversion of the bedGraph score track) — pysam-developers/pysam

## Examples

```
import deepCregr
```

## Evaluation signals

- Output file exists and parses as a valid bedGraph with chrom, start, end, score columns
- A score column is present and populated for every base in the analyzed windows
- All saliency scores are non-negative (value_in_range >= 0) after taking the absolute value
- Scores are averaged consistently across all overlapping sliding (1 Mb + bin) windows covering each base, so no position has missing/zero coverage where at least one window overlaps
- bigWig conversion of the bedGraph succeeds and the bigWig interval coordinates match the bedGraph

## Limitations

- deepC was developed under TensorFlow v1.8 and only v1.8, 1.14 and 2.1 are supported; other versions are untested, so automatic differentiation may fail on other TensorFlow builds
- Saliency depends on the trained model: deepC predicts substantially better when pre-seeded with hidden layers optimized to predict a compendium of chromatin features, so gradients from a poorly trained checkpoint are less meaningful
- Taking the absolute value discards the sign/direction of each base's effect, yielding a non-negative but non-directional importance track
- pysam is only tested at v0.15.2 in the deepC repository, and pybedtools requires a compatible version of bedtools installed on the system

## Evidence
