---
name: bonferroni-multiple-testing-correction
description: Use when many p-values are produced simultaneously (e.g., 48 K-S tests
  per lane pair on genome-partitioned read distributions) and per-combination significance
  calls must be corrected for multiple testing.
when_to_use_negative:
- Only one hypothesis test is being run, so no family-wise correction is needed
- The p-values come from different, unrelated families of tests — pooling them into
  one Bonferroni correction would be overly conservative and not what the assessment
  prescribes (correction is applied within each lane pair)
- The goal is peak calling on ChIP-Seq read data itself (e.g., HPeak HMM-based enrichment),
  rather than calling significance on genome-partitioned test statistics
edam_topics:
- http://edamontology.org/topic_0220
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/1471-2105-11-369
    title: 'HPeak: an HMM-based algorithm for defining read-enriched regions in ChIP-Seq
      data'
  source_content_hash: sha256:d2002ec2c4f7a1176d70c64ada7161df9f335a5b4b6a5d5fcdce1775c7ffa5a9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools: []
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0220
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# bonferroni-multiple-testing-correction

## Summary

Applies Bonferroni correction to control the family-wise error rate when many statistical tests are run in parallel, e.g., two-sample Kolmogorov-Smirnov tests across 48 chromosome/strand combinations per ChIP-Seq lane pair, using a significance threshold of 0.001 to call significant discrepancies.

## When to use

Use when many p-values are produced simultaneously (e.g., 48 K-S tests per lane pair on genome-partitioned read distributions) and per-combination significance calls must be corrected for multiple testing.

## When NOT to use

- Only one hypothesis test is being run, so no family-wise correction is needed
- The p-values come from different, unrelated families of tests — pooling them into one Bonferroni correction would be overly conservative and not what the assessment prescribes (correction is applied within each lane pair)
- The goal is peak calling on ChIP-Seq read data itself (e.g., HPeak HMM-based enrichment), rather than calling significance on genome-partitioned test statistics

## Inputs

- Vector of raw p-values from the tests in one family (e.g., 48 two-sample K-S test p-values, one per chromosome/strand combination for a ChIP-Seq lane pair)
- Family definition (which tests belong to the same correction unit — here, within each lane pair)
- Significance threshold (0.001 in the STAT1 K-S reproducibility assessment)

## Outputs

- Per-test significance calls (adjusted p-values or pass/fail flags) for each of the 48 chromosome/strand combinations
- Summary table of significant-combination counts per lane pair (ks_test_results.csv)

## How to apply

First tabulate the read count distributions for each lane across the 48 chromosome/strand combinations (24 chromosomes × 2 strands) and run a two-sample Kolmogorov-Smirnov test for the lane pair in each combination, yielding 48 p-values. Then apply the Bonferroni correction within each lane pair: compare each raw p-value to the threshold divided among the tests in the family — here the significance threshold is 0.001 applied to the 48-combination family — and call a combination significant only if its corrected p-value passes. Rationale: with 48 simultaneous tests, uncorrected p-values would inflate the family-wise error rate, so the correction is what makes 'at most 1/48 significant' a defensible reproducibility verdict. Finally, summarize the number of significant combinations per lane pair (within-group vs. cross-group) in a table such as ks_test_results.csv.

## Examples

```
from statsmodels.stats.multitest import multipletests; reject, p_adj, _, _ = multipletests(ks_pvals_48, alpha=0.001, method='bonferroni')
```

## Evaluation signals

- The number of significant chromosome/strand combinations for within-group lane pairs is at most 1 out of 48
- Cross-group (stimulated vs unstimulated) lane pairs show 36-42 out of 48 significant combinations, matching the Table 1 pattern
- Adjusted significance threshold equals 0.001 applied across the family of 48 tests per pair (not per test individually)
- Summary table ks_test_results.csv contains one count row per lane pair, split by within-group and cross-group comparisons
- No combination is called significant on a raw p-value alone without passing the Bonferroni-corrected criterion

## Limitations

- Bonferroni is conservative, so true effects may be missed when the family is large; the assessment pattern itself accepts at most 1/48 within-group significance rather than zero
- The correction must be applied within each lane pair's 48-combination family; applying it across all lane pairs at once would change the error-rate guarantees
- With paired-end data, fragment/insert-size handling (e.g., -isize in HPeak) affects the underlying read distributions before any p-values are computed, so errors upstream propagate into the corrected calls

## Evidence

- [other] Bonferroni threshold within pair: [evidence span withheld — anchor status: not_found]
- [other] K-S test family across 48 combinations: [evidence span withheld — anchor status: not_found]
- [other] Reproducibility finding: [evidence span withheld — anchor status: not_found]
- [readme] p-values from peak calling context: "Accurate peak calling with detailed information including summit location and p-value"
