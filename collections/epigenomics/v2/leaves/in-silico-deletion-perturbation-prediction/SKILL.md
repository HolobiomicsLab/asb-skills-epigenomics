---
name: in-silico-deletion-perturbation-prediction
description: Use when you need to rank functional genomic elements (CTF sites, promoters,
  enhancers) by their predicted impact on chromatin interactions, i.e. identify which
  elements underlie genome folding patterns from sequence.
when_to_use_negative:
- You need experimentally measured Hi-C interaction changes rather than sequence-based
  predicted perturbation effects
- Your question is single-nucleotide variant impact rather than whole-element (CTCF/promoter/enhancer)
  deletion effects — use deepC's SNP-level variant scoring instead
- You lack a trained deepC model for the relevant cell type and suitable tissue-specific
  Hi-C training data
edam_operation: http://edamontology.org/operation_0484
edam_topics:
- http://edamontology.org/topic_3678
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3169
tools:
- name: deepC
  role: Predicts chromatin interaction maps from megabase-scale DNA sequence; used
    to score reference vs deletion windows
  repo: github.com/rschwess/deepC
- name: TensorFlow 2.1
  role: Backend runtime for running deepC predictions with GPU support
- name: pybedtools
  role: Loads and manipulates the merged CTCF/DNase peak BED intervals and shuffled
    background sites
- name: pysam
  role: Extracts megabase-scale reference sequence windows centered on each element
  repo: https://github.com/pysam-developers/pysam
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41592-020-0960-3
    title: 'DeepC: predicting 3D genome folding using megabase-scale transfer learning'
  source_content_hash: sha256:e15343562bf52c98d4680a1e36d75bb9b215fcca7e92493aee81423d0b6d73af
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - TensorFlow 2.1
  - deepC
  - pybedtools
  - pysam
  edam_operation: http://edamontology.org/operation_0484
  edam_topics:
  - http://edamontology.org/topic_3678
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# in-silico-deletion-perturbation-prediction

## Summary

A genome-wide in silico deletion screen that scores regulatory elements (CTCF sites, promoters, enhancers, shuffled backgrounds) by the predicted change in chromatin interaction maps when the element is removed from megabase-scale sequence windows, using deepC to quantify each element's contribution to 3D genome folding.

## When to use

Use when you need to rank functional genomic elements (CTF sites, promoters, enhancers) by their predicted impact on chromatin interactions, i.e. identify which elements underlie genome folding patterns from sequence.

## When NOT to use

- You need experimentally measured Hi-C interaction changes rather than sequence-based predicted perturbation effects
- Your question is single-nucleotide variant impact rather than whole-element (CTCF/promoter/enhancer) deletion effects — use deepC's SNP-level variant scoring instead
- You lack a trained deepC model for the relevant cell type and suitable tissue-specific Hi-C training data

## Inputs

- Trained deepC GM12878 model checkpoint (from task_001)
- Merged BED file of ENCODE GM12878 CTCF peaks (n=45,635) and DNase peaks (n=47,320) plus 3,850 shuffled background sites
- Reference genome assembly (accessed via pysam)
- Element chromatin class annotations (promoter, enhancer, CTCF)

## Outputs

- deletion_screen_results.csv with ~96,805 rows containing element coordinates, chromatin class, and mean absolute interaction difference (reference vs deletion prediction)

## How to apply

Load the trained GM12878 deepC model checkpoint (trained in task_001) plus a merged BED file of ENCODE GM12878 CTCF peaks (n=45,635), DNase peaks (n=47,320) and 3,850 shuffled background sites using pybedtools. For each element, extract megabase-scale sequence windows centered on the element from the reference genome with pysam, and construct a deletion-variant window with the element sequence removed. Run deepC predictions (import deepCregr under TensorFlow 2.1 with GPU support) on both reference and deletion sequences and compute the mean absolute difference between the predicted chromatin interaction maps. Annotate each element with its chromatin class (promoter, enhancer, CTCF) and write the per-element table (~96,805 rows) to deletion_screen_results.csv; expect CTCF sites and active-mark-associated promoters/enhancers to score highest (promoters > enhancers).

## Related tools

- **deepC** (Predicts chromatin interaction maps from megabase-scale DNA sequence; used to score reference vs deletion windows) — github.com/rschwess/deepC
- **TensorFlow 2.1** (Backend runtime for running deepC predictions with GPU support)
- **pybedtools** (Loads and manipulates the merged CTCF/DNase peak BED intervals and shuffled background sites)
- **pysam** (Extracts megabase-scale reference sequence windows centered on each element) — https://github.com/pysam-developers/pysam

## Examples

```
import deepCregr; pred_ref = deepCregr.predict(ref_seq); pred_del = deepCregr.predict(del_seq); score = abs(pred_ref - pred_del).mean()
```

## Evaluation signals

- deletion_screen_results.csv exists with the expected ~96,805 rows and required fields (coordinates, chromatin class, mean absolute interaction difference)
- CTF site deletions rank among the highest-scoring classes, consistent with 'CTCF binding site deletions are most likely to cause strong chromatin interaction changes'
- Promoter deletions score higher on average than enhancer deletions, matching the reported ordering (promoters > enhancers)
- Elements associated with active chromatin marks score higher than inactive-mark elements
- Shuffled background sites (n=3,850) show lower predicted deletion effects than functional element classes

## Limitations

- Scores are model-predicted impacts from deepC, not experimentally measured perturbation outcomes; no direct comparative benchmark against Akita is presented
- Deletion screen accuracy depends on the quality of the trained model; deepC predictions improve substantially when pre-seeded with chromatin-feature-trained hidden layers
- Only elements within megabase-scale windows centered on the model's input size can be assessed; TF versions other than 1.8, 1.14 and 2.1 are untested

## Evidence
