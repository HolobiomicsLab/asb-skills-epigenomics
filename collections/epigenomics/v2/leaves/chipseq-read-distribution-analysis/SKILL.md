---
name: chipseq-read-distribution-analysis
description: Use when two or more sequencing lanes from the same ChIP-Seq sample group
  (e.g., stimulated or unstimulated STAT1) must be checked for consistent genome-wide
  read distributions across chromosome/strand combinations, or when within-group vs
  cross-group reproducibility must be contrasted.
when_to_use_negative:
- Input is a called peak set or binding-site list rather than per-lane read count
  distributions over the genome — this skill assesses raw read distribution consistency,
  not peak overlap.
- Data are probe-based assays (e.g. ChIP-chip intensities) rather than genome-wide
  sequencing reads; the paper treats ChIP-chip as the lower-resolution comparator,
  not the target of this test.
- Lanes being compared belong to different biological conditions and the goal is reproducibility
  assessment — cross-group comparisons there will legitimately show 36–42/48 significant
  combinations and must not be read as assay failure.
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: HPeak
  role: HMM-based read-enrichment framework on the same STAT1 ChIP-Seq data whose
    HDF extension and genome binning (25 bp bins) define the read-distribution representation
    that the per-chromosome/strand K-S comparison operates on
  repo: github.com/cauyrd/HPeak
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/1471-2105-11-369
    title: 'HPeak: an HMM-based algorithm for defining read-enriched regions in ChIP-Seq
      data'
  source_content_hash: sha256:9d5384924a9437030f53945a0406b61475bfccf6b7a995fd8ddabc657560eb04
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HPeak
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# chipseq-read-distribution-analysis

## Summary

Quantifies ChIP-Seq assay reproducibility by applying two-sample Kolmogorov-Smirnov tests to genome-wide read count distributions over all 48 chromosome/strand combinations (24 chromosomes × 2 strands), comparing lanes within a sample group. Developed on the STAT1 ChIP-Seq dataset, where it showed near-identical read distributions within groups but strong divergence between stimulated and unstimulated lanes.

## When to use

Use when two or more sequencing lanes from the same ChIP-Seq sample group (e.g., stimulated or unstimulated STAT1) must be checked for consistent genome-wide read distributions across chromosome/strand combinations, or when within-group vs cross-group reproducibility must be contrasted.

## When NOT to use

- Input is a called peak set or binding-site list rather than per-lane read count distributions over the genome — this skill assesses raw read distribution consistency, not peak overlap.
- Data are probe-based assays (e.g. ChIP-chip intensities) rather than genome-wide sequencing reads; the paper treats ChIP-chip as the lower-resolution comparator, not the target of this test.
- Lanes being compared belong to different biological conditions and the goal is reproducibility assessment — cross-group comparisons there will legitimately show 36–42/48 significant combinations and must not be read as assay failure.

## Inputs

- STAT1 ChIP-Seq sequencing lane data (stimulated and unstimulated groups), e.g. from http://www.bcgsc.ca/data/chipseq
- Per-lane, per-chromosome/strand read count distributions (48 combinations per lane)
- Lane-to-group assignment (which lanes are stimulated vs unstimulated)

## Outputs

- Summary table ks_test_results.csv with counts of significant chromosome/strand combinations per lane pair
- Per-combination K-S test p-values for within-group and cross-group lane pairs
- Bonferroni-corrected significance calls per chromosome/strand combination at threshold 0.001

## How to apply

Download the STAT1 ChIP-Seq dataset and partition the genome into 24 chromosomes × 2 strands = 48 chromosome/strand combinations, tabulating the sequencing read count distribution for each lane in each combination. For every within-group lane pair (both lanes share the same stimulation condition), run a two-sample Kolmogorov-Smirnov test per combination; repeat for every cross-group (stimulated vs unstimulated) lane pair. Within each pair, apply a Bonferroni correction with significance threshold 0.001 to the 48 p-values, and count significant chromosome/strand combinations. The rationale is that reads, optionally extended directionally into hypothetical DNA fragments (HDFs) and binned across the genome as in HPeak-style processing, provide a genome-wide distribution whose pairwise similarity reflects assay reproducibility independent of peak calling.

## Related tools

- **HPeak** (HMM-based read-enrichment framework on the same STAT1 ChIP-Seq data whose HDF extension and genome binning (25 bp bins) define the read-distribution representation that the per-chromosome/strand K-S comparison operates on) — github.com/cauyrd/HPeak

## Examples

```
from scipy.stats import ks_2samp; stat, p = ks_2samp(lane_a_counts, lane_b_counts)  # per chromosome/strand, 48 tests per lane pair
```

## Evaluation signals

- Every lane pair yields exactly 48 chromosome/strand K-S tests (24 chromosomes × 2 strands); any missing combination is a tabulation error.
- Within-group lane pairs show at most 1 out of 48 combinations significant at Bonferroni threshold 0.001, supporting assay reproducibility.
- Cross-group (stimulated vs unstimulated) lane pairs show 36–42 out of 48 significant combinations, matching the Table 1 pattern.
- The output ks_test_results.csv reproduces the expected within-group vs cross-group contrast (≈0–1 significant vs 36–42 significant combinations).

## Limitations

- The method requires per-lane, per-chromosome/strand read tabulation, so lanes with insufficient depth or missing strand information will under-power the K-S tests.
- Bonferroni correction at 0.001 is applied within each pair; conclusions hold for this dataset design (STAT1 stimulated/unstimulated) and should be interpreted with caution when transferred to other ChIP-Seq datasets.
- Read distributions assess consistency of genome-wide coverage, not agreement in called peaks; identical distributions do not guarantee identical biological peak calls.

## Evidence

- [discussion] HDF extension defines the read representation: "we followed the method used by Robertson et al. [11] to extend each short read directionally from its start position to form a hypothetical DNA fragment (HDF)"
- [discussion] genome-wide binning/tabulation basis: "HPeak partitions the entire genome into small bins of fixed length (25 bp in this study) and subsequently counts the numbers of HDFs that fall in each bin"
- [discussion] STAT1 dataset on which the assessment is grounded: "When comparing publicly available STAT1 ChIP-Seq and ChIP-chip data"
- [readme] peak-caller output context for read distributions: "Accurate peak calling with detailed information including summit location and p-value"
