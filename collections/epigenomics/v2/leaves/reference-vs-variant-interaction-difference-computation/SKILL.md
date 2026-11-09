---
name: reference-vs-variant-interaction-difference-computation
description: Use when fine-mapping candidate regulatory SNPs (e.g. CTCF-site variants
  such as 17q21 asthma SNPs) to identify which allele most likely alters chromatin
  architecture, by comparing predicted interaction maps between reference and alternate
  alleles.
when_to_use_negative:
- When the goal is predicting genome folding from sequence without variant comparison
  (use plain deepC prediction instead)
- When the variant is a large structural rearrangement or deletion rather than a single
  base pair change — deepC deletion screens use a different workflow
- When comparing deepC against alternative models such as Akita; the paper presents
  no direct comparative benchmark
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0102
tools:
- name: deepC
  role: Sequence-based deep neural network (imported as deepCregr) that predicts chromatin
    interaction maps from megabase-scale reference and variant DNA sequences
  repo: https://github.com/rschwess/deepC
- name: pysam
  role: Extracts reference and variant DNA sequence windows from the hg19 genome at
    SNP loci
  repo: https://github.com/pysam-developers/pysam
- name: pybedtools
  role: Handles BED intervals for sampled CTCF-site SNP loci during sequence extraction
  repo: https://github.com/daler/pybedtools
- name: TensorFlow
  role: Runtime backend for the deepC model; use a supported version (v1.8, 1.14 or
    2.1)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41592-020-0960-3
    title: 'DeepC: predicting 3D genome folding using megabase-scale transfer learning'
  source_content_hash: sha256:a5e6692808a91e33d02332fcf716c6fd577a68e66a3e01716470fe4e423981fb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rschwess/deepC
  tools:
  - TensorFlow
  - deepC
  - pybedtools
  - pysam
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Reference-vs-variant interaction difference computation

## Summary

Quantifies the effect of single-nucleotide variants on 3D genome folding by running a deepC sequence-based model on reference and variant megabase-scale sequence windows and computing the mean absolute difference between predicted interaction maps, then ranking variants by effect size percentile.

## When to use

Use when fine-mapping candidate regulatory SNPs (e.g. CTCF-site variants such as 17q21 asthma SNPs) to identify which allele most likely alters chromatin architecture, by comparing predicted interaction maps between reference and alternate alleles.

## When NOT to use

- When the goal is predicting genome folding from sequence without variant comparison (use plain deepC prediction instead)
- When the variant is a large structural rearrangement or deletion rather than a single base pair change — deepC deletion screens use a different workflow
- When comparing deepC against alternative models such as Akita; the paper presents no direct comparative benchmark

## Inputs

- Trained deepC model (e.g. GM12878 5 kb model, loaded via deepCregr)
- Reference genome FASTA (hg19) accessed through pysam
- BED file of sampled CTCF-site SNP loci (e.g. 1000 SNPs) with reference and alternate alleles
- SNP identifiers with alleles to prioritize (e.g. rs12936231, rs4065275)

## Outputs

- Mean absolute interaction difference (effect size) per SNP between reference and variant predicted interaction maps
- Percentile ranking of each SNP's effect size within the sampled SNP set
- CSV table of effect sizes and percentile rankings for the highlighted SNPs

## How to apply

Load the trained tissue-specific deepC model (e.g. the task_001 GM12878 5 kb model) via `import deepCregr` and use pysam/pybedtools to extract reference and variant megabase-scale sequence windows from the hg19 genome at each candidate SNP locus. Run deepC predictions on both reference and variant windows, producing chromatin interaction maps for each allele. Compute the mean absolute difference between the reference and variant predicted interaction maps as the SNP's effect size. Rank the sampled SNP effect sizes (e.g. the 1000 sampled CTCF-site SNPs) to obtain percentile rankings, then write effect sizes and percentiles for highlighted variants to a CSV table. The rationale is that sequence variants at base pair resolution can be interpreted against predicted chromatin interactions, prioritizing the causal allele (e.g. rs12936231 effect 0.176, top 1%, versus weak rs4065275 at 0.006, top 11%).

## Related tools

- **deepC** (Sequence-based deep neural network (imported as deepCregr) that predicts chromatin interaction maps from megabase-scale reference and variant DNA sequences) — https://github.com/rschwess/deepC
- **pysam** (Extracts reference and variant DNA sequence windows from the hg19 genome at SNP loci) — https://github.com/pysam-developers/pysam
- **pybedtools** (Handles BED intervals for sampled CTCF-site SNP loci during sequence extraction) — https://github.com/daler/pybedtools
- **TensorFlow** (Runtime backend for the deepC model; use a supported version (v1.8, 1.14 or 2.1))

## Examples

```
import deepCregr, pysam; model = deepCregr.loadModel('GM12878_5kb'); pred_ref = deepCregr.predict(refSeq); pred_var = deepCregr.predict(varSeq); effect = abs(pred_ref - pred_var).mean()
```

## Evaluation signals

- Output table contains exactly 1000 SNP rows matching the sampled CTCF-site SNP set
- rs12936231 effect size equals approximately 0.176 and ranks within the top 1% of sampled SNPs
- rs4065275 effect size equals approximately 0.006 and ranks within the top 11%
- Effect sizes are non-negative mean absolute differences and percentile rankings are consistent with the rank order of the effect sizes

## Limitations

- deepC supports only specific TensorFlow versions (v1.8, 1.14 and 2.1); other versions have not been tested
- GPU support is preferable for running predictions and essential for training, so CPU-only runs will be slow
- No direct comparative benchmark between deepC and Akita is presented; the authors defer this to future work
- Additional python dependencies (numpy >= 1.16.4, pysam, pybedtools plus compatible bedtools) must be installed

## Evidence

- [other] deepC predicts the rs12936231 risk allele to have a strong effect on genome folding (mean absolute interaction difference 0.176): [evidence span withheld — anchor status: not_found]
