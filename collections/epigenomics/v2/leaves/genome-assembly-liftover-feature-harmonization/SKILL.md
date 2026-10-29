---
name: genome-assembly-liftover-feature-harmonization
description: Use when benchmark pairs or annotations are given on an older assembly
  (e.g. hg19 BENGI) but features were computed on a newer one (hg38 cCREs-ELS/GENCODE
  v40), requiring coordinate conversion and per-pair feature averaging before cross-method
  comparison.
when_to_use_negative:
- Pairs and genomic features are already expressed on the same assembly and registry
  (e.g. fully hg38 cCREs-ELS/GENCODE v40), so no cross-assembly harmonization is needed
- Candidate interactions extend beyond 500 KB, since the CENTRE pipeline explicitly
  restricts the ET pair search to that interval
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: CENTRE
  role: Supplies the hg38 feature values (generic and cell-type-specific) that must
    be averaged and mapped onto the hg19 BENGI pairs; R/Bioconductor package
  repo: https://github.com/slrvv/CENTRE
- name: UCSC Genome Browser liftover
  role: Converts BENGI ET pair coordinates from hg19 to hg38
- name: TargetFinder
  role: Downstream benchmark model (303 EPW features) trained on the hg19 BENGI annotation
    using the harmonized feature table
  repo: https://github.com/shwhalen/targetfinder
- name: GenomicRanges (findOverlaps)
  role: Overlaps uplifted regions with the ENCODE cCRE Registry v2 on hg38
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad687
    title: 'CENTRE: a gradient boosting algorithm for Cell-type-specific ENhancer-Target
      pREdiction'
  source_content_hash: sha256:1526167046a3cfbe0e7d35cdf83375fa1a8f36a88fdafc2e6b8d059550e782c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/slrvv/CENTRE
  tools:
  - CENTRE
  - GenomicRanges (findOverlaps)
  - R
  - TargetFinder
  - UCSC Genome Browser liftover
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# Genome assembly liftover and feature harmonization

## Summary

Lifts enhancer-target (ET) pairs between genome assemblies (hg19→hg38) with the UCSC Genome Browser liftover facility, re-anchors them on the ENCODE cCRE Registry v2 and GENCODE TSSs, and harmonizes CENTRE hg38 feature values onto the hg19 BENGI annotation so legacy benchmark models like TargetFinder can be compared against CENTRE.

## When to use

Use when benchmark pairs or annotations are given on an older assembly (e.g. hg19 BENGI) but features were computed on a newer one (hg38 cCREs-ELS/GENCODE v40), requiring coordinate conversion and per-pair feature averaging before cross-method comparison.

## When NOT to use

- Pairs and genomic features are already expressed on the same assembly and registry (e.g. fully hg38 cCREs-ELS/GENCODE v40), so no cross-assembly harmonization is needed
- Candidate interactions extend beyond 500 KB, since the CENTRE pipeline explicitly restricts the ET pair search to that interval

## Inputs

- BENGI All-Pairs.Natural-Ratio ET pairs on hg19 (interacting/not-interacting labels)
- UCSC hg19ToHg38 liftOver chain file
- ENCODE Registry of cCREs, version 2, on hg38
- GENCODE v40 transcription start sites (hg38)
- CENTRE-computed feature values (generic + cell-type-specific) on hg38 pairs

## Outputs

- Lifted, cCRE-anchored hg38 ET pair set restricted to 500 KB
- Harmonized feature table averaged per original hg19 pair for TargetFinder/Random Forest input
- Per-dataset AUPRC and F1-score comparison table across CENTRE, TargetFinder (full and reduced) and Random Forest

## How to apply

Run every ET pair of the BENGI dataset through the UCSC Genome Browser liftover facility (hg19ToHg38 chain) to obtain hg38 coordinates. Overlap the uplifted regions with version 2 of the ENCODE Registry of cCREs on hg38 (using findOverlaps from GenomicRanges) and GENCODE v40 TSSs, keeping only pairs within the commonly accepted 500 KB distance threshold. Because a single hg19 pair can lift to multiple hg38 cCRE/TSS combinations, aggregate CENTRE hg38 feature values by averaging over all hg38 pairs that map to the same original hg19 pair. Feed the resulting harmonized table to the reimplemented TargetFinder model (GradientBoostingClassifier, n_estimators=4000, learning_rate=0.1, max_depth=5, max_features='log2', 303 EPW-window features) and the Random Forest baseline, then compute AUPRC and F1 from the 12 chromosome-based outer CV folds.

## Related tools

- **CENTRE** (Supplies the hg38 feature values (generic and cell-type-specific) that must be averaged and mapped onto the hg19 BENGI pairs; R/Bioconductor package) — https://github.com/slrvv/CENTRE
- **UCSC Genome Browser liftover** (Converts BENGI ET pair coordinates from hg19 to hg38)
- **TargetFinder** (Downstream benchmark model (303 EPW features) trained on the hg19 BENGI annotation using the harmonized feature table) — https://github.com/shwhalen/targetfinder
- **GenomicRanges (findOverlaps)** (Overlaps uplifted regions with the ENCODE cCRE Registry v2 on hg38)

## Examples

```
liftOver bengi_pairs_hg19.bed hg19ToHg38.over.chain.gz bengi_pairs_hg38.bed unmapped.bed
```

## Evaluation signals

- Every uplifted ET pair resolves to a cCREs-ELS region and a GENCODE v40 TSS via findOverlaps on hg38; unmapped/broken pairs are dropped and logged
- After filtering, all retained pairs satisfy the 500 KB distance threshold
- The number of unique original hg19 pairs is preserved after averaging hg38 feature values over pairs mapping to the same hg19 pair
- Resulting per-dataset AUPRC/F1 ordering reproduces the reported Fig. 4A/4B pattern (CENTRE ≥ TargetFinder in 11 of 13 BENGI datasets)

## Limitations

- Liftovers can fail or split a pair into multiple hg38 loci, requiring explicit aggregation (averaging) of feature values per original pair
- The reduced TargetFinder model (13 features: DNase, H3K4me3, H3K27ac, CTCF + distance) is the fallback for tissues lacking the full genomic assay set
- Performance comparisons are sensitive to the choice of validation data, and F1 scores remain low on many BENGI datasets

## Evidence

- [methods] We updated all ET pairs of the BENGI datasets on the hg38 annotation using the UCSC Genome Browser liftover facility: "We updated all ET pairs of the BENGI datasets on the hg38 annotation using the UCSC Genome Browser liftover facility"
- [methods] Then we overlapped the uplifted regions with the latest version 2 of ENCODE Registry of cCREs on hg38: "Then we overlapped the uplifted regions with the latest version 2 of ENCODE Registry of cCREs on hg38"
- [results] We however limit the search for ET pairs to a distance of 500 KB which is a commonly accepted threshold: "We however limit the search for ET pairs to a distance of 500 KB which is a commonly accepted threshold"
- [readme] Cell-type specific histone ChIP-seq in BAM format for H3K27ac, H3K4me3 and H3K4me1.: [evidence span withheld — anchor status: external_doc]
