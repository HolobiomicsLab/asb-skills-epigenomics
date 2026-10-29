---
name: pcawg-cohort-driver-result-aggregation
description: Use when comparing called cancer driver elements from DriverPower (or
  a similar method) against gold-standard gene sets on the same PCAWG cohorts to quantify
  precision, recall and F1 for coding, splice-site and non-coding discovery.
when_to_use_negative:
- Input is raw per-mutation data or feature tables rather than called driver elements
  — run the DriverPower 'model' and 'infer' sub-commands first
- The goal is to discover drivers in a single cohort rather than to benchmark and
  compare methods on shared PCAWG cohorts
- No gold-standard set (e.g. CGC) or comparator method calls are available, so precision/recall/F1
  cannot be defined
edam_topics:
- http://edamontology.org/topic_2648
- http://edamontology.org/topic_0622
tools:
- name: DriverPower
  role: Method under benchmark; its infer sub-command produces the per-cohort driver
    result tables being scored against CGC
  repo: https://github.com/smshuai/DriverPower
- name: bedtools
  role: Genomic interval intersection of each method's called elements with CGC and
    PCAWG-consensus/raw reference sets to define true positives
- name: Python
  role: Computes precision, recall and F1 per method per element class and writes
    f1_scores_comparison.tsv (requires Python >= 3.5 for DriverPower)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-13929-1
    title: Combined burden and functional impact tests for cancer driver discovery
      using DriverPower
  source_content_hash: sha256:71f43da8afbbcc99caaab5970db71fcf972b43da403989284284c93c97ae733a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/smshuai/DriverPower
  tools:
  - DriverPower
  - Python
  - bedtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2648
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# pcawg-cohort-driver-result-aggregation

## Summary

Benchmark DriverPower's driver calls against six published methods (ActiveDriverWGS, ExInAtor, LARVA, MutSig, ncdDetect, oncodriveFML) on the 26 non-melanoma/lymphoma PCAWG cohorts, using CGC and PCAWG-consensus/raw sets as gold standards to compute precision, recall and F1 per element class.

## When to use

Use when comparing called cancer driver elements from DriverPower (or a similar method) against gold-standard gene sets on the same PCAWG cohorts to quantify precision, recall and F1 for coding, splice-site and non-coding discovery.

## When NOT to use

- Input is raw per-mutation data or feature tables rather than called driver elements — run the DriverPower 'model' and 'infer' sub-commands first
- The goal is to discover drivers in a single cohort rather than to benchmark and compare methods on shared PCAWG cohorts
- No gold-standard set (e.g. CGC) or comparator method calls are available, so precision/recall/F1 cannot be defined

## Inputs

- DriverPower per-cohort result tables saved as '[outDir]/[name].result.tsv' by the driverpower infer sub-command
- Driver calls from six comparator methods (ActiveDriverWGS, ExInAtor, LARVA, MutSig, ncdDetect, oncodriveFML) from the PCAWG Drivers and Functional Interpretation Working Group
- Gold-standard reference sets: CGC, PCAWG-consensus and PCAWG-raw driver sets
- Somatic mutation calls from 26 non-melanoma/lymphoma PCAWG cohorts (2583 cancer genomes)

## Outputs

- f1_scores_comparison.tsv — F1 scores for all methods per element class
- Per-method precision, recall and F1 for coding, splice-site and non-coding driver discovery
- True-positive assignments of called elements via bedtools intersection with CGC/PCAWG reference sets

## How to apply

Load DriverPower driver calls per cohort from the result tables '[outDir]/[name].result.tsv' produced by the 'driverpower infer' sub-command for the 26 non-melanoma/lymphoma cohorts. Load the six comparator methods' calls used by the PCAWG Drivers and Functional Interpretation Working Group, then use bedtools genomic interval intersection to overlap each method's called elements with the CGC and PCAWG-consensus/raw reference sets to define true positives. Compute precision, recall and F1 per method per element class (coding, splice site, non-coding) in Python and tabulate all F1 scores into f1_scores_comparison.tsv. Validate against the reported benchmarks: DriverPower coding F1 0.81 (precision 0.84, recall 0.79), splice-site F1 0.91 vs ncdDetect 0.65 and oncodriveFML 0.32, non-coding F1 0.79, within tolerance and highest overall.

## Related tools

- **DriverPower** (Method under benchmark; its infer sub-command produces the per-cohort driver result tables being scored against CGC) — https://github.com/smshuai/DriverPower
- **bedtools** (Genomic interval intersection of each method's called elements with CGC and PCAWG-consensus/raw reference sets to define true positives)
- **Python** (Computes precision, recall and F1 per method per element class and writes f1_scores_comparison.tsv (requires Python >= 3.5 for DriverPower))

## Examples

```
bedtools bed12tobed6 -i ./test_elements.bed12.gz | cut -f1-4 > test_elements.tsv
```

## Evaluation signals

- Computed DriverPower coding F1 matches the reported benchmark of 0.81 (precision 0.84, recall 0.79) within tolerance
- Splice-site F1 reproduces 0.91 for DriverPower versus ncdDetect 0.65 and oncodriveFML 0.32; non-coding F1 reproduces 0.79
- f1_scores_comparison.tsv contains F1 entries for all seven methods (DriverPower plus six comparators) per element class
- DriverPower ranks highest in F1 overall, consistent with 'best balance of precision and recall' though not always top on either metric independently
- bedtools intersection counts of called elements overlapping CGC define true positives reproducibly across methods

## Limitations

- CGC as gold standard biases benchmarking toward known coding drivers; functional impact score schemes are currently biased toward coding mutations
- DriverPower provides the best F1 balance but is not always the top-ranked method when precision or recall is considered independently
- False-positive driver calls may arise from variant-calling artefacts or biological processes not captured by the BMR model (e.g. regional AID activity)

## Evidence

- [abstract] DriverPower has the highest F1 score for both coding and non-coding driver discovery: "Comparing to six published methods used by the PCAWG Drivers and Functional Interpretation Working Group, DriverPower has the highest F1 score for both coding and non-coding driver discovery."
- [discussion] DriverPower provides the best balance of precision and recall, although is not always the top-ranked method when either metric is considered independently: "DriverPower provides the best balance of precision and recall, although is not always the top-ranked method when either metric is considered independently"
- [methods] bedtools bed12tobed6 -i ./test_elements.bed12.gz | cut -f1-4 > test_elements.tsv: "bedtools bed12tobed6 -i ./test_elements.bed12.gz  | cut -f1-4 > test_elements.tsv"
- [readme] DriverPower is a tool used to discover potential coding and non-coding cancer driver elements from tumour whole-genome or whole-exome somatic mutations.: "DriverPower is a tool used to discover potential coding and non-coding cancer driver elements from tumour whole-genome or whole-exome somatic mutations."
