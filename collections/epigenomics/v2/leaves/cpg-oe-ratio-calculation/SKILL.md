---
name: cpg-oe-ratio-calculation
description: Use when you have FASTA sequences of candidate regulatory regions (e.g.,
  HOT vs COLD) and need CpG O/E and GC-related features to test whether CpG/GC-rich
  sequence characteristics discriminate between region classes.
when_to_use_negative:
- Input is already a computed feature table (e.g., k-mer frequencies or CpG O/E values)
  rather than raw region sequences
- The goal is TF motif enrichment analysis — the article notes HOT regions lack canonical
  motifs for transcription factors thought to be bound there, so motif-based features
  will not discriminate HOT from COLD
- Comparing ChIP-seq signal intensity rather than intrinsic sequence properties of
  regions
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: glmnet
  role: Elastic net classifier consuming the CpG O/E and k-mer feature matrix to rank
    CpG/GC features and produce cross-validated AUCs per species
- name: HOT-region-main.R
  role: Repository script used for the HOT region analysis, including sequence feature
    computation
  repo: https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkz460
    title: 'HOT or not: examining the basis of high-occupancy target regions'
  source_content_hash: sha256:0b2cd46de2abbfcf5963b973a140c1b945c70139147f226e4947304f1ce5c574
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions
  tools:
  - HOT-region-main.R
  - glmnet
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cpg-oe-ratio-calculation

## Summary

Calculates the CpG observed/expected (O/E) ratio, alongside GC content, GC skew, and k-mer (k=2,3,4) frequencies, for HOT and COLD genomic regions to build sequence-feature matrices that discriminate HOT from non-HOT regions in elastic net classifiers.

## When to use

Use when you have FASTA sequences of candidate regulatory regions (e.g., HOT vs COLD) and need CpG O/E and GC-related features to test whether CpG/GC-rich sequence characteristics discriminate between region classes.

## When NOT to use

- Input is already a computed feature table (e.g., k-mer frequencies or CpG O/E values) rather than raw region sequences
- The goal is TF motif enrichment analysis — the article notes HOT regions lack canonical motifs for transcription factors thought to be bound there, so motif-based features will not discriminate HOT from COLD
- Comparing ChIP-seq signal intensity rather than intrinsic sequence properties of regions

## Inputs

- FASTA sequences of HOT regions and COLD (non-HOT) regions per species
- Genomic region coordinates (BED-style interval sets) from the HOT/COLD definition step

## Outputs

- Per-region feature table with CpG O/E ratio, GC content, GC skew, and k-mer (k=2,3,4) frequency vectors
- Labeled feature matrix (HOT/COLD) for downstream glmnet elastic net classification

## How to apply

For each genomic region from the HOT and COLD sets, extract the sequence and count CpG dinucleotides; compute the CpG O/E ratio as observed CpG count divided by the count expected from GC content (expected = (C count * G count) / region length). Aggregate these counts into a feature matrix that also contains k-mer frequency vectors for k = 2, 3, and 4, GC content, and GC skew per region, with HOT/COLD class labels attached. Rationale: the article found that CpG/GC-rich k-mers together with the CpG O/E ratio rank as the most important predictive features across all species for distinguishing HOT regions, which are also enriched in R-loops and G-quadruplex structures. The feature matrix feeds species-specific elastic net (glmnet) classifiers with 10-fold cross-validation. Correctness is judged by whether CpG/GC-rich features surface as top-ranked model coefficients and cross-validated AUCs fall in the reported 0.82-0.94 range.

## Related tools

- **glmnet** (Elastic net classifier consuming the CpG O/E and k-mer feature matrix to rank CpG/GC features and produce cross-validated AUCs per species)
- **HOT-region-main.R** (Repository script used for the HOT region analysis, including sequence feature computation) — https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions

## Examples

```
Rscript Scripts/HOT-region-main.R
```

## Evaluation signals

- CpG O/E ratio and GC/CpG-rich k-mers appear among the top-ranked coefficients in the fitted elastic net models for all species
- Cross-validation AUCs of the species-specific classifiers fall within the reported 0.82-0.94 range
- Feature matrix rows match 1:1 with the input HOT/COLD region sets and carry correct class labels
- CpG O/E values are computed as observed CpG dinucleotide counts normalized by the expectation from GC content per region (sanity-check on GC-rich promoters, e.g., house-keeping gene promoters, showing O/E > 1)

## Limitations

- The article attributes artificial ChIP-seq enrichment on HOT regions to these GC/CpG-rich sequence features, so high discriminatory AUC may reflect ChIP-seq artifact propensity (hyper-ChIPable regions) rather than genuine TF co-binding biology
- Lack of cross-species comparison of HOT region sequence features existed prior to this work, so per-species thresholds were tuned separately with glmnet rather than a universal model
- Repository README only documents that 'scripts have been used for analysis of HOT regions'; no standalone utility or parameter documentation for CpG O/E computation is provided

## Evidence

- [other] Compute k-mer frequency vectors for k = 2, 3, and 4 over each region, plus GC content, GC skew, and CpG observed/expected (O/E) ratio per region.: [evidence span withheld — anchor status: not_found]
- [results] CpG/GC-rich k-mers together with CpG observed/expected ratio rank as the most important predictive features across all species.: [evidence span withheld — anchor status: not_found]
- [abstract] We observe sequence characteristics and genomic features that are discriminatory of HOT regions, such as GC/CpG-rich k-mers, enrichment of RNA-DNA hybrids (R-loops) and DNA tertiary structures: "We observe sequence characteristics and genomic features that are discriminatory of HOT regions, such as GC/CpG-rich k-mers, enrichment of RNA-DNA hybrids (R-loops) and DNA tertiary structures"
- [readme] The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R).: "The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R)."
