---
name: variant-effect-prediction-from-chromatin-interaction-models
description: Use when prioritizing non-coding SNPs (e.g., GWAS loci such as the 17q21
  asthma variants rs12936231 and rs4065275) for likely causal status based on their
  predicted effect on chromatin interaction maps.
when_to_use_negative:
- No direct deepC-vs-Akita comparative benchmark is presented in the paper, so this
  skill should not be used as evidence that deepC outperforms Akita
- deepC predicts chromatin interactions, not gene expression, so do not use it to
  claim expression effects without an additional linkage analysis
- The installed tensorflow version is outside the tested range (v1.8, 1.14, 2.1);
  other versions are untested and may break predictions
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_0102
tools:
- name: deepC
  role: Deep neural network that predicts reference and variant Hi-C interaction maps
    from megabase-scale DNA sequence; imported as deepCregr for in-script predictions
  repo: github.com/rschwess/deepC
- name: pysam
  role: Extracts reference and variant sequences at SNP loci from the hg19 genome
    FASTA
  repo: pysam-developers/pysam
- name: pybedtools
  role: Handles BED interval operations for extracting sequence windows at the 1000
    sampled CTCF-site SNP loci
  repo: github.com/rschwess/deepC
- name: TensorFlow
  role: Runtime for executing deepC model predictions; deepC was developed under v1.8
    and supports v1.8, 1.14 and 2.1
  repo: github.com/ibab/tensorflow-wavenet
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41592-020-0960-3
    title: 'DeepC: predicting 3D genome folding using megabase-scale transfer learning'
  source_content_hash: sha256:6b697a21d249f8d99d39d594fad2a656f196687bf46ea8c18bc50e9e53de91a4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - TensorFlow
  - deepC
  - pybedtools
  - pysam
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# variant-effect-prediction-from-chromatin-interaction-models

## Summary

Predicts the impact of single base-pair variants on 3D genome folding by running a trained deepC model on reference and variant megabase-scale sequence windows and computing effect sizes as the mean absolute difference between predicted Hi-C interaction maps, enabling fine-mapping of causal SNPs among candidates.

## When to use

Use when prioritizing non-coding SNPs (e.g., GWAS loci such as the 17q21 asthma variants rs12936231 and rs4065275) for likely causal status based on their predicted effect on chromatin interaction maps.

## When NOT to use

- No direct deepC-vs-Akita comparative benchmark is presented in the paper, so this skill should not be used as evidence that deepC outperforms Akita
- deepC predicts chromatin interactions, not gene expression, so do not use it to claim expression effects without an additional linkage analysis
- The installed tensorflow version is outside the tested range (v1.8, 1.14, 2.1); other versions are untested and may break predictions

## Inputs

- Reference genome FASTA (hg19) for sequence extraction
- BED or table of candidate SNP loci (e.g., 1000 CTCF-site SNPs) with reference and alternate alleles
- Pretrained deepC model checkpoint for the relevant tissue and resolution (e.g., GM12878 5 kb model)

## Outputs

- CSV table of SNP effect sizes (mean absolute interaction-map difference) and percentile rankings per variant
- Prioritized list of candidate causal variants (e.g., rs12936231 ranked top 1%)

## How to apply

Load the pretrained tissue-specific deepC model (e.g., the task_001 GM12878 5 kb model via `import deepCregr`) and use pysam/pybedtools to extract reference and variant megabase-scale sequence windows from the hg19 genome at each sampled SNP locus (e.g., ~1000 CTCF-site SNPs). Run deepC predictions on both reference and variant windows, then compute the mean absolute difference between the reference and variant predicted interaction maps as the SNP effect size. Rank all SNP effect sizes to obtain percentile rankings per variant. Prioritize the variant with the highest effect size and percentile (e.g., rs12936231 at 0.176, top 1%, over rs4065275 at 0.006, top 11%). Write effect sizes and percentile rankings to a CSV table, validating against published figures (e.g., Figure 3d).

## Related tools

- **deepC** (Deep neural network that predicts reference and variant Hi-C interaction maps from megabase-scale DNA sequence; imported as deepCregr for in-script predictions) — github.com/rschwess/deepC
- **pysam** (Extracts reference and variant sequences at SNP loci from the hg19 genome FASTA) — pysam-developers/pysam
- **pybedtools** (Handles BED interval operations for extracting sequence windows at the 1000 sampled CTCF-site SNP loci) — github.com/rschwess/deepC
- **TensorFlow** (Runtime for executing deepC model predictions; deepC was developed under v1.8 and supports v1.8, 1.14 and 2.1) — github.com/ibab/tensorflow-wavenet

## Examples

```
import deepCregr; model = deepCregr.load('GM12878_5kb'); ref_map, var_map = model.predict(ref_seq), model.predict(var_seq); effect = abs(ref_map - var_map).mean()
```

## Evaluation signals

- Output CSV contains exactly one row per sampled SNP (1000 rows for the CTCF-site SNP set)
- rs12936231 effect size is 0.176 (mean absolute interaction difference) and ranks within the top 1% of sampled CTCF SNPs, matching Figure 3d
- rs4065275 effect size is 0.006 and ranks within the top 11%, matching Figure 3d
- All computed effect sizes are non-negative, consistent with a mean absolute difference metric
- Percentile rankings for all SNPs fall within [0, 1] and are consistent with the computed effect-size ordering

## Limitations

- deepC was developed under tensorflow v1.8 and only supports v1.8, 1.14 and 2.1; other versions have not been tested
- Training deepC models requires GPU support for hours to days; predictions are feasible without a GPU but run significantly faster with one
- Effect sizes reflect predicted chromatin interaction changes only; the paper does not directly benchmark deepC against Akita, deferring comparative study to future work

## Evidence
