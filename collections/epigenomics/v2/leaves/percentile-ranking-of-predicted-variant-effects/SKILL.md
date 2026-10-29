---
name: percentile-ranking-of-predicted-variant-effects
description: Use when you have deepC-predicted effect sizes for one or more candidate
  SNPs and need a population-level comparison (percentile) against sampled background
  SNPs to decide which variant most plausibly perturbs genome folding.
when_to_use_negative:
- You have only one or a handful of variants with no sampled background SNP distribution
  — a percentile ranking is meaningless without the ~1000-SNP comparison set
- The variants are large structural variants (deletions of promoters/enhancers/CTCF
  sites) rather than single base pair SNPs — the deletion-screen workflow, not SNP
  percentile ranking, applies
- Predictions are being made on genomes other than the hg19 reference with matched
  deepC model coordinates without adjusting sequence extraction and model window coordinates
edam_operation: http://edamontology.org/operation_3225
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: deepC
  role: Sequence-based deep neural network (imported as deepCregr) that predicts chromatin
    interaction maps from megabase-scale DNA sequence; run on reference and variant
    windows to compute effect sizes
  repo: https://github.com/rschwess/deepC
- name: pysam
  role: Extracts reference and variant sequences at SNP loci from the hg19 genome
    for deepC input
  repo: https://github.com/pysam-developers/pysam
- name: pybedtools
  role: Handles BED intervals of sampled CTCF-site SNP loci during sequence extraction
  repo: https://github.com/daler/pybedtools
- name: TensorFlow
  role: Backend runtime (supported versions 1.8, 1.14, 2.1) for running deepC predictions
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41592-020-0960-3
    title: 'DeepC: predicting 3D genome folding using megabase-scale transfer learning'
  source_content_hash: sha256:620671d3e6ad561da8809db489df7d209517bc13484fc49682504a1d5997062a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rschwess/deepC
  tools:
  - TensorFlow
  - deepC
  - pybedtools
  - pysam
  edam_operation: http://edamontology.org/operation_3225
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# percentile-ranking-of-predicted-variant-effects

## Summary

Ranks sequence-based deepC predicted variant effect sizes (mean absolute difference between reference and variant predicted chromatin interaction maps) across a sampled SNP background to convert raw effect magnitudes into percentile scores, enabling prioritization of likely causal variants such as the 17q21 asthma SNPs rs12936231 and rs4065275.

## When to use

Use when you have deepC-predicted effect sizes for one or more candidate SNPs and need a population-level comparison (percentile) against sampled background SNPs to decide which variant most plausibly perturbs genome folding.

## When NOT to use

- You have only one or a handful of variants with no sampled background SNP distribution — a percentile ranking is meaningless without the ~1000-SNP comparison set
- The variants are large structural variants (deletions of promoters/enhancers/CTCF sites) rather than single base pair SNPs — the deletion-screen workflow, not SNP percentile ranking, applies
- Predictions are being made on genomes other than the hg19 reference with matched deepC model coordinates without adjusting sequence extraction and model window coordinates

## Inputs

- hg19 reference genome (indexed FASTA for pysam retrieval)
- BED file of ~1000 sampled CTCF-site SNP loci (e.g. 17q21 asthma SNPs rs12936231 and rs4065275)
- Trained tissue-specific deepC model checkpoint (e.g. GM12878 5 kb model)
- deepC python package (deepCregr module) and TensorFlow backend

## Outputs

- CSV table of SNP effect sizes (mean absolute reference-vs-variant interaction map difference) and percentile rankings, one row per SNP (1000 rows)
- Percentile scores for highlighted SNPs (e.g. rs12936231: 0.176 effect, top 1%; rs4065275: 0.006 effect, top 11%)

## How to apply

Load the tissue-specific deepC model (e.g. the task_001 GM12878 5 kb model via `import deepCregr`) and use pysam/pybedtools to extract reference and variant megabase-scale sequence windows at each of ~1000 sampled CTCF-site SNP loci from the hg19 genome. Run deepC predictions on both reference and variant sequences, then compute the mean absolute difference between the predicted interaction maps as each SNP's effect size. Rank the ~1000 effect sizes to assign each variant a percentile; in the article, rs12936231 (effect 0.176, top 1%) was prioritized over rs4065275 (effect 0.006, top 11%) as the likely causal asthma variant. Validation compares the output CSV (1000 SNP rows) against reported Figure 3d values.

## Related tools

- **deepC** (Sequence-based deep neural network (imported as deepCregr) that predicts chromatin interaction maps from megabase-scale DNA sequence; run on reference and variant windows to compute effect sizes) — https://github.com/rschwess/deepC
- **pysam** (Extracts reference and variant sequences at SNP loci from the hg19 genome for deepC input) — https://github.com/pysam-developers/pysam
- **pybedtools** (Handles BED intervals of sampled CTCF-site SNP loci during sequence extraction) — https://github.com/daler/pybedtools
- **TensorFlow** (Backend runtime (supported versions 1.8, 1.14, 2.1) for running deepC predictions)

## Examples

```
import deepCregr; model = deepCregr.deepC(model_path='GM12878_5kb.h5'); ref_map = model.predict(ref_seq); var_map = model.predict(var_seq); effect = abs(ref_map - var_map).mean()
```

## Evaluation signals

- Output CSV contains exactly 1000 SNP rows (one per sampled CTCF-site SNP)
- rs12936231 effect size equals ~0.176 with a top 1% percentile ranking
- rs4065275 effect size equals ~0.006 with a top 11% percentile ranking
- Percentile rankings are consistent with the values reported in Figure 3d of the article
- Effect sizes are non-negative mean absolute differences between reference and variant predicted interaction maps

## Limitations

- deepC predictions are from megabase-scale sequence only; the article's comparison with Akita was deferred, so ranking relies on deepC's predictions alone ('future comparative study and consolidation')
- The percentile background depends on the sampled SNP set (1000 CTCF-site SNPs); different sampling schemes change percentiles and thus variant prioritization
- deepC was developed under TensorFlow v1.8 and supports only v1.8, 1.14 and 2.1; other versions are untested, so reproducibility requires version matching
- Predictions run significantly faster with a GPU, though they are feasible without one

## Evidence

- [results] deepC predicts the rs12936231 risk allele to have a strong effect on genome folding (mean absolute interaction difference 0.176), ranking within the top 1% of sampled CTCF SNPs: [evidence span withheld — anchor status: not_found]
