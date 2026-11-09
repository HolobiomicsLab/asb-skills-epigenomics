---
name: cross-lane-reproducibility-assessment
description: Use when multiple sequencing lanes exist within the same sample group
  (e.g., stimulated or unstimulated STAT1 ChIP-Seq lanes) and you must quantify whether
  their genome-wide read distributions are consistent before downstream peak calling.
when_to_use_negative:
- Only one lane is available per condition, so no within-group lane pair exists to
  compare
- Input consists of already-called peak sets rather than raw genome-wide read-count
  distributions — peak-level overlap analysis answers a different question than distribution-level
  reproducibility
- The goal is cross-condition differential binding (stimulated vs unstimulated) rather
  than within-group assay reproducibility; cross-group K-S results (36-42/48 significant)
  reflect biological difference, not noise
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: HPeak
  role: Downstream HMM-based peak-calling pipeline whose input lanes (extended to
    hypothetical DNA fragments, HDFs) are the ones whose cross-lane reproducibility
    this skill validates beforehand
  repo: github.com/cauyrd/HPeak
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/1471-2105-11-369
    title: 'HPeak: an HMM-based algorithm for defining read-enriched regions in ChIP-Seq
      data'
  source_content_hash: sha256:180de97b61148f257952c43bc87a41e4270ee9682d5dcb4c27cfb0684f8a8eec
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HPeak
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cross-lane-reproducibility-assessment

## Summary

Quantitatively assesses reproducibility of a ChIP-Seq assay by applying two-sample Kolmogorov-Smirnov (K-S) tests to genome-wide read-count distributions between lane pairs, partitioned across 24 chromosomes x 2 strands = 48 chromosome/strand combinations, with Bonferroni correction at threshold 0.001.

## When to use

Use when multiple sequencing lanes exist within the same sample group (e.g., stimulated or unstimulated STAT1 ChIP-Seq lanes) and you must quantify whether their genome-wide read distributions are consistent before downstream peak calling.

## When NOT to use

- Only one lane is available per condition, so no within-group lane pair exists to compare
- Input consists of already-called peak sets rather than raw genome-wide read-count distributions — peak-level overlap analysis answers a different question than distribution-level reproducibility
- The goal is cross-condition differential binding (stimulated vs unstimulated) rather than within-group assay reproducibility; cross-group K-S results (36-42/48 significant) reflect biological difference, not noise

## Inputs

- STAT1 ChIP-Seq dataset of stimulated and unstimulated lanes (downloaded from http://www.bcgsc.ca/data/chipseq), one lane per sample group condition
- Per-lane read count distributions tabulated for each of the 48 chromosome/strand combinations (24 chromosomes x 2 strands)
- Lane-group assignment metadata (which lanes are stimulated vs unstimulated)

## Outputs

- ks_test_results.csv: per-lane-pair summary table of the number of significant chromosome/strand combinations after Bonferroni correction (threshold 0.001), for within-group and cross-group pairs
- K-S test p-values for each lane pair across the 48 chromosome/strand combinations

## How to apply

Download the STAT1 ChIP-Seq dataset and partition the genome into 48 chromosome/strand combinations (24 chromosomes x 2 strands), tabulating the read count distribution for each lane in each combination. For every within-group lane pair (both lanes in the same stimulation condition), run a two-sample Kolmogorov-Smirnov test on the read distributions in each of the 48 combinations. Repeat the K-S test for cross-group (stimulated vs unstimulated) lane pairs as a contrast. Apply a Bonferroni correction with significance threshold 0.001 to the p-values within each pair to call significant chromosome/strand combinations, then count significant combinations per pair. Within-group reproducibility is supported when at most 1 of 48 combinations is significant, whereas cross-group comparisons are expected to yield 36-42 of 48 significant combinations, showing the test discriminates real biological differences from lane-to-lane noise. Write the per-pair counts to a summary table (ks_test_results.csv) and check it against the expected Table 1 pattern.

## Related tools

- **HPeak** (Downstream HMM-based peak-calling pipeline whose input lanes (extended to hypothetical DNA fragments, HDFs) are the ones whose cross-lane reproducibility this skill validates beforehand) — github.com/cauyrd/HPeak

## Examples

```
from scipy.stats import ks_2samp; stat, p = ks_2samp(lane_a_counts, lane_b_counts); significant = sum(1 for p_i in pvals if p_i < 0.001/48)
```

## Evaluation signals

- For every within-group (same stimulation condition) lane pair, at most 1 of the 48 chromosome/strand combinations shows a significant K-S discrepancy at Bonferroni-corrected threshold 0.001
- Cross-group (stimulated vs unstimulated) lane pairs yield 36-42 of 48 significant combinations, confirming the K-S test detects genuine distributional shifts
- ks_test_results.csv contains one row per lane pair with the count of significant chromosome/strand combinations and matches the Table 1 pattern
- Each lane's read distributions are tabulated over exactly 48 combinations (24 chromosomes x 2 strands), covering the full genome
- Bonferroni correction is applied within each pair (threshold 0.001) rather than across all pairs globally

## Limitations

- The assessment is distribution-level on raw reads per chromosome/strand, not peak-level; consistent read distributions do not guarantee identical peak calls
- At most one significant combination is expected within groups, so a single discrepant chromosome/strand is not evidence of assay failure — interpretation thresholds follow the Table 1 pattern
- The reference STAT1 data are single-end short reads extended directionally into hypothetical DNA fragments (HDFs) with fragment-length uncertainty; paired-end or longer-read data may need re-tabulation of distributions
- Cross-group significance (36-42/48) demonstrates biological signal, so this test cannot by itself separate reproducibility failure from differential binding when groups are mixed up

## Evidence

- [other] K-S test with Bonferroni correction supports within-group reproducibility: [evidence span withheld — anchor status: not_found]
- [discussion] Genome partitioned into fixed bins for read counting (HDF coverage context): "HPeak partitions the entire genome into small bins of fixed length (25 bp in this study) and subsequently counts the numbers of HDFs that fall in each bin"
- [discussion] Reads extended directionally into hypothetical DNA fragments before distribution analysis: "we followed the method used by Robertson et al. [11] to extend each short read directionally from its start position to form a hypothetical DNA fragment (HDF)"
- [readme] Downstream pipeline provides statistical output grounded in per-region calls: "Accurate peak calling with detailed information including summit location and p-value"
