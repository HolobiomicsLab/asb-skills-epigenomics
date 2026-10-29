---
name: functional-impact-score-adjustment-of-driver-calls
description: Use when you already have DriverPower burden-test-only calls and per-element
  functional impact scores (e.g. averaged per-mutation CADD PHRED per binID) and want
  to re-rank/infer drivers with functional adjustment to boost precision.
when_to_use_negative:
- No functional impact scores are available for the tested elements — running without
  --funcScore is the burden-only baseline, not this skill.
- Functional score schemes are biased toward coding mutations, so applying this to
  poorly scored non-coding elements may not improve calls (article notes the bias
  in discussion).
- You only need the background mutation rate model — use 'driverpower model' instead
  of the functional-adjusted infer step.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_2640
tools:
- name: DriverPower
  role: Runs 'driverpower infer' with --funcScore and --funcScoreCut to apply posterior
    functional adjustment to burden-test calls
  repo: github.com/smshuai/DriverPower
- name: Python >= 3.5
  role: Runtime environment for DriverPower and for preparing/averaging the per-element
    functional score table
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-13929-1
    title: Combined burden and functional impact tests for cancer driver discovery
      using DriverPower
  source_content_hash: sha256:332a14e68a97d012d6b011acd4bc6c0945f16a1f72c7317b2bd01bbc319ec098
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DriverPower
  - Python >=
  - Python >= 3.5
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# functional-impact-score-adjustment-of-driver-calls

## Summary

Posterior functional adjustment of DriverPower driver calls: re-run 'driverpower infer' with per-element functional impact scores (CADD, DANN, EIGEN, LINSIGHT) via --funcScore/--funcScoreCut to refine the burden-test-only candidate set. Incorporating functional evidence increases driver-discovery precision, e.g. in Panc-AdenoCA it gained ACVR1B, RBM10 and ZFP36L2 and removed the likely false positive FAU.

## When to use

Use when you already have DriverPower burden-test-only calls and per-element functional impact scores (e.g. averaged per-mutation CADD PHRED per binID) and want to re-rank/infer drivers with functional adjustment to boost precision.

## When NOT to use

- No functional impact scores are available for the tested elements — running without --funcScore is the burden-only baseline, not this skill.
- Functional score schemes are biased toward coding mutations, so applying this to poorly scored non-coding elements may not improve calls (article notes the bias in discussion).
- You only need the background mutation rate model — use 'driverpower model' instead of the functional-adjusted infer step.

## Inputs

- Test feature table (test_feature.hdf5)
- Test response table
- Pre-trained DriverPower BMR model information
- Per-element functional score table (TSV keyed by binID; columns per scheme: CADD, DANN, EIGEN, LINSIGHT), e.g. averaged per-mutation CADD PHRED scores per element

## Outputs

- ./output/DriverPower.result.tsv (burden-test-only driver calls)
- ./output/DriverPower_func.result.tsv (functional-adjusted driver calls)
- Diff of gained drivers (expect ACVR1B, RBM10, ZFP36L2 in Panc-AdenoCA) and lost drivers (expect FAU)

## How to apply

First generate the burden-only baseline with 'driverpower infer' using the test feature table, test response table and pre-trained model info (--method auto, --geoMean True), omitting --funcScore, writing ./output/DriverPower.result.tsv. Build a functional score table (TSV keyed by binID, one column per scheme such as CADD, DANN, EIGEN, LINSIGHT), e.g. by averaging per-mutation CADD PHRED scores retrieved via the CADD web interface per element. Re-run 'driverpower infer' with the same inputs plus --funcScore and --funcScoreCut in 'NAME:CUTOFF' format with cutoffs in (0,1] (e.g. 'CADD:0.01;DANN:0.03;EIGEN:0.3'), writing ./output/DriverPower_func.result.tsv. Diff the two call sets: in Panc-AdenoCA the expected result is a three-driver gain (ACVR1B, RBM10, ZFP36L2) and loss of one likely false positive (FAU), and the burden-only precision is 74.6% (156/209) for CGC genes. The rationale is that burden tests perform poorly on rare driver events, while functional impact tests rescue them and suppress non-functional false positives.

## Related tools

- **DriverPower** (Runs 'driverpower infer' with --funcScore and --funcScoreCut to apply posterior functional adjustment to burden-test calls) — github.com/smshuai/DriverPower
- **Python >= 3.5** (Runtime environment for DriverPower and for preparing/averaging the per-element functional score table)

## Examples

```
driverpower infer --feature test_feature.hdf5 --response test_y.tsv --models models --funcScore functional_scores.tsv --funcScoreCut "CADD:0.01" -o ./output/DriverPower_func.result.tsv
```

## Evaluation signals

- Burden-only baseline precision in Panc-AdenoCA matches the reported 74.6% (156/209) for CGC genes (Fig. 2a).
- Functional-adjusted run adds exactly three drivers (ACVR1B, RBM10, ZFP36L2) and loses FAU relative to the burden-only set.
- --funcScoreCut cutoffs are all in (0,1] and follow the 'NAME:CUTOFF' semicolon-delimited format (e.g. 'CADD:0.01;DANN:0.03;EIGEN:0.3').
- Functional score table is keyed by binID with one column per scheme matching the names used in --funcScoreCut.
- Both runs use the same --method auto and --geoMean True settings so the only difference is the functional adjustment.

## Limitations

- Functional impact score schemes are currently biased toward coding mutations, limiting benefit for non-coding elements.
- False-positive driver calls may arise from variant-calling artefacts or biological processes not captured by the BMR model (e.g. regional AID activity), which functional adjustment alone may not fix.
- DriverPower's BMR is trained on predominantly neutral regions and does not model negatively selected sites, limiting recall.

## Evidence

- [abstract] By incorporating functional impact scores, we are able to further increase the accuracy of driver discovery.: "By incorporating functional impact scores, we are able to further increase the accuracy of driver discovery."
- [methods] call driver candidates **without** functional information, aka, burden-test only: "call driver candidates **without** functional information, aka, burden-test only"
- [methods] The format of this parameter is a string in "NAME1:CUTOFF1;NAME2:CUTOFF2...": "``--funcScoreCut``: [*optional*] Cutoff of each functional impact scores. The format of this parameter is a string in "NAME1:CUTOFF1;NAME2:CUTOFF2...""
- [intro] Mutational burden tests work best for calling frequently recurrent driver events and perform poorly when applied to rare driver events: "Mutational burden tests work best for calling frequently recurrent driver events and perform poorly when applied to rare driver events"
- [discussion] functional impact score schemes are currently biased toward coding mutations; therefore, improved functional scoring schemes will also help us identify more functionally relevant non-coding cancer: "functional impact score schemes are currently biased toward coding mutations; therefore, improved functional scoring schemes will also help us identify more functionally relevant non-coding cancer"
