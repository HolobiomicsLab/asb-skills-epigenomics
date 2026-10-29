---
name: cancer-driver-call-overlap-analysis
description: Use when benchmarking a driver discovery method's called elements against
  a gold-standard set (e.g. CGC) on PCAWG whole genomes, to compute precision, recall
  and F1 per element class (coding, splice site, non-coding).
when_to_use_negative:
- The goal is to call drivers from raw somatic mutations rather than to score already-produced
  driver calls against a gold standard
- No curated gold-standard reference set (e.g. CGC or PCAWG-consensus) is available,
  so true positives cannot be defined by interval overlap
- The input is per-mutation significance data rather than element-level driver calls
  that can be intersected with reference regions
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_2648
tools:
- name: DriverPower
  role: Source of the driver calls being benchmarked; result tables from its 'infer'
    sub-command are read per cohort
  repo: https://github.com/smshuai/DriverPower
- name: bedtools
  role: Genomic interval intersection of called elements with CGC and PCAWG-consensus/raw
    reference sets to define true positives
- name: Python
  role: Computes precision, recall and F1 per method per element class and writes
    f1_scores_comparison.tsv (requires Python >= 3.5 per the DriverPower docs)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-13929-1
    title: Combined burden and functional impact tests for cancer driver discovery
      using DriverPower
  source_content_hash: sha256:bad73a62a082032a926697a8dbd8572d4a6adbeada081e0465de93d070fd3df0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/smshuai/DriverPower
  tools:
  - DriverPower
  - Python
  - bedtools
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_2648
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# cancer-driver-call-overlap-analysis

## Summary

Benchmark cancer driver discovery calls by intersecting each method's called genomic elements with a gold-standard reference set (CGC, plus PCAWG-consensus/raw sets) and computing precision, recall and F1 score per element class. Used to reproduce the DriverPower benchmark against six published PCAWG methods.

## When to use

Use when benchmarking a driver discovery method's called elements against a gold-standard set (e.g. CGC) on PCAWG whole genomes, to compute precision, recall and F1 per element class (coding, splice site, non-coding).

## When NOT to use

- The goal is to call drivers from raw somatic mutations rather than to score already-produced driver calls against a gold standard
- No curated gold-standard reference set (e.g. CGC or PCAWG-consensus) is available, so true positives cannot be defined by interval overlap
- The input is per-mutation significance data rather than element-level driver calls that can be intersected with reference regions

## Inputs

- DriverPower per-cohort driver calls in '[outDir]/[name].result.tsv' tables for the 26 non-melanoma/lymphoma PCAWG cohorts
- Driver calls from six comparator methods (ActiveDriverWGS, ExInAtor, LARVA, MutSig, ncdDetect, oncodriveFML) from the PCAWG Drivers and Functional Interpretation Working Group
- Gold-standard reference sets: CGC, plus PCAWG-consensus and PCAWG-raw driver element sets

## Outputs

- Precision, recall and F1 score per method per element class (coding, splice site, non-coding)
- f1_scores_comparison.tsv containing tabulated F1 scores for all methods

## How to apply

Load each method's driver calls per cohort: for DriverPower, read the per-cohort result tables saved as '[outDir]/[name].result.tsv' by the 'driverpower infer' sub-command for the 26 non-melanoma/lymphoma PCAWG cohorts; for the comparators, load calls from ActiveDriverWGS, ExInAtor, LARVA, MutSig, ncdDetect and oncodriveFML as used by the PCAWG Drivers and Functional Interpretation Working Group. Overlap each method's called elements with the CGC and PCAWG-consensus/raw reference sets using bedtools genomic interval intersection to define true positives. Compute precision, recall and F1 per method per element class (coding, splice site, non-coding) in Python, tabulate all F1 scores in f1_scores_comparison.tsv, and validate against the reported benchmarks: DriverPower F1 0.81 for coding (precision 0.84, recall 0.79), 0.91 for splice sites (vs ncdDetect 0.65 and oncodriveFML 0.32), and 0.79 for non-coding, with DriverPower ranked highest overall.

## Related tools

- **DriverPower** (Source of the driver calls being benchmarked; result tables from its 'infer' sub-command are read per cohort) — https://github.com/smshuai/DriverPower
- **bedtools** (Genomic interval intersection of called elements with CGC and PCAWG-consensus/raw reference sets to define true positives)
- **Python** (Computes precision, recall and F1 per method per element class and writes f1_scores_comparison.tsv (requires Python >= 3.5 per the DriverPower docs))

## Examples

```
bedtools bed12tobed6 -i ./test_elements.bed12.gz | cut -f1-4 > test_elements.tsv
```

## Evaluation signals

- Computed DriverPower coding-driver metrics reproduce the reported benchmark: F1 0.81 with precision 0.84 and recall 0.79 within an acceptable tolerance
- Splice-site F1 values match the reported 0.91 for DriverPower vs 0.65 for ncdDetect and 0.32 for oncodriveFML
- Non-coding F1 for DriverPower matches the reported 0.79
- DriverPower ranks highest in F1 among the seven methods across coding and non-coding element classes
- The output file f1_scores_comparison.tsv exists and contains one row per method per element class

## Limitations

- CGC is biased toward known coding drivers, so overlap-based benchmarking may not fully reflect non-coding driver discovery performance
- DriverPower's background mutation model is trained on predominantly neutral regions, limiting recall; it does not model negatively selected sites
- Somatic variants are systematically undercalled in non-coding regions (~216 TERT promoter mutations estimated missed in PCAWG), which depresses measured recall
- Functional impact score schemes are currently biased toward coding mutations, affecting cross-class comparability
- DriverPower provides the best precision-recall balance but is not always the top-ranked method when either metric is considered independently

## Evidence

- [abstract] DriverPower has the highest F1 score for both coding and non-coding driver discovery.: "Comparing to six published methods used by the PCAWG Drivers and Functional Interpretation Working Group, DriverPower has the highest F1 score for both coding and non-coding driver discovery."
- [discussion] Best balance of precision and recall, not always top-ranked per metric.: "DriverPower provides the best balance of precision and recall, although is not always the top-ranked method when either metric is considered independently"
- [methods] The infer sub-command produces the driver calls loaded for benchmarking.: "The ``infer`` sub-command is used to call drivers from test data with pre-trained models."
- [readme] DriverPower discovers coding and non-coding driver elements from somatic mutations.: "DriverPower is a tool used to discover potential coding and non-coding cancer driver elements from tumour whole-genome or whole-exome somatic mutations."
