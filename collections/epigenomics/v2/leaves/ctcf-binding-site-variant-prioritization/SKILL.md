---
name: ctcf-binding-site-variant-prioritization
description: Use when fine-mapping a locus to decide which of several non-coding SNPs
  overlapping CTCF binding sites is the likely causal variant driving altered chromatin
  architecture or gene expression.
when_to_use_negative:
- The question concerns large structural variants or element deletions rather than
  single base-pair SNPs — deepC's SNP scoring workflow here targets single nucleotide
  changes
- No tissue-specific trained deepC model is available for the relevant cell type,
  since transfer-learned tissue-specific models are needed for accurate interaction
  predictions
- Candidate variants do not lie in regulatory elements (e.g. CTCF binding sites) relevant
  to genome folding, so interaction-map differences are unlikely to be informative
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3169
tools:
- name: deepC
  role: Sequence-based deep neural network that predicts chromatin interactions from
    megabase-scale DNA sequence, used to generate reference and variant interaction
    maps for each SNP
  repo: github.com/rschwess/deepC
- name: pysam
  role: Extracts reference and variant genomic sequences at SNP loci from the hg19
    genome
  repo: https://github.com/pysam-developers/pysam
- name: pybedtools
  role: Manipulates BED intervals to define the SNP loci and sequence windows for
    prediction
- name: TensorFlow
  role: Runtime backend for deepC models (v1.8, 1.14, or 2.1 supported); GPU recommended
    for predictions
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41592-020-0960-3
    title: 'DeepC: predicting 3D genome folding using megabase-scale transfer learning'
  source_content_hash: sha256:36a09aa189a673e807e40c6d6c227eefda644fa18b99749b11607ffc35574450
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - TensorFlow
  - deepC
  - pybedtools
  - pysam
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# ctcf-binding-site-variant-prioritization

## Summary

Prioritize candidate regulatory SNPs by using deepC to predict the effect of single base-pair variants on 3D genome folding: each variant is scored by the mean absolute difference between reference and variant predicted chromatin interaction maps, then ranked against a background set of sampled CTCF-site SNPs to nominate the likely causal variant.

## When to use

Use when fine-mapping a locus to decide which of several non-coding SNPs overlapping CTCF binding sites is the likely causal variant driving altered chromatin architecture or gene expression.

## When NOT to use

- The question concerns large structural variants or element deletions rather than single base-pair SNPs — deepC's SNP scoring workflow here targets single nucleotide changes
- No tissue-specific trained deepC model is available for the relevant cell type, since transfer-learned tissue-specific models are needed for accurate interaction predictions
- Candidate variants do not lie in regulatory elements (e.g. CTCF binding sites) relevant to genome folding, so interaction-map differences are unlikely to be informative

## Inputs

- Reference genome assembly (hg19) accessed via pysam/pybedtools
- Candidate SNP loci with reference and alternate alleles (e.g. rs12936231, rs4065275)
- Background set of ~1000 sampled CTCF-site SNP loci
- Pre-trained deepC model for the relevant tissue/cell type (e.g. GM12878 5 kb resolution)

## Outputs

- CSV table of SNP effect sizes (mean absolute interaction-map difference) and percentile rankings for all sampled SNPs
- Prioritized list of candidate causal variants (e.g. rs12936231 at 0.176 effect size, top 1%)

## How to apply

Load a pre-trained tissue-specific deepC model (e.g. the task_001 GM12878 5 kb model) via `import deepCregr` and use pysam/pybedtools to extract reference and variant megabase-scale sequence windows from the hg19 genome at each candidate SNP locus. Run deepC predictions on both reference and variant sequences to obtain predicted interaction maps. Compute the mean absolute difference between the reference and variant predicted interaction maps as the variant's effect size. Sample a background set (e.g. ~1000 SNPs at CTCF sites) and rank each variant's effect size to obtain a percentile; variants in the top ~1% of the distribution are strong candidates. Validate by checking the output table has one row per sampled SNP and that the highlighted variants reproduce reported effect sizes and percentiles (e.g. rs12936231: 0.176, top 1%; rs4065275: 0.006, top 11%).

## Related tools

- **deepC** (Sequence-based deep neural network that predicts chromatin interactions from megabase-scale DNA sequence, used to generate reference and variant interaction maps for each SNP) — github.com/rschwess/deepC
- **pysam** (Extracts reference and variant genomic sequences at SNP loci from the hg19 genome) — https://github.com/pysam-developers/pysam
- **pybedtools** (Manipulates BED intervals to define the SNP loci and sequence windows for prediction)
- **TensorFlow** (Runtime backend for deepC models (v1.8, 1.14, or 2.1 supported); GPU recommended for predictions)

## Examples

```
import deepCregr; model = deepCregr.loadModel('GM12878_5kb'); pred = deepCregr.predict(model, variant_seq); effect = abs(pred_ref - pred_var).mean()
```

## Evaluation signals

- Output CSV contains exactly 1000 SNP rows, one per sampled CTCF-site variant
- rs12936231 effect size ≈ 0.176 and percentile within the top 1% of the background distribution
- rs4065275 effect size ≈ 0.006 and percentile within the top 11%, confirming weak predicted impact
- Ranking percentiles are consistent with the ordering of raw effect sizes across all 1000 SNPs
- Results reproduce the reported Figure 3d values

## Limitations

- No direct comparative benchmark between deepC and Akita is presented; the authors defer this to future work
- deepC predictions are substantially better when pre-seeded with hidden layers trained on a compendium of chromatin features, so unseeded models may underperform
- Running predictions without GPU support is feasible but significantly slower; training requires GPU support

## Evidence

- [results] deepC predicts the rs12936231 risk allele to have a strong effect on genome folding (mean absolute interaction difference 0.176), ranking within the top 1% of sampled CTCF SNPs: [evidence span withheld — anchor status: not_found]
