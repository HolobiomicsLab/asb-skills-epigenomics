---
name: driver-call-set-differential-comparison-across-test-schemes
description: Use when you need to quantify how adding functional impact scores (CADD,
  DANN, EIGEN, LINSIGHT) changes DriverPower driver calls for a tumour cohort, or
  to report burden-only precision against a CGC-gene truth set.
when_to_use_negative:
- When only a single call set is needed and no functional score table (CADD/DANN/EIGEN/LINSIGHT
  per element) is available — the differential comparison requires both burden-only
  and functional-adjusted runs
- When working with non-coding driver discovery, since functional impact score schemes
  are currently biased toward coding mutations and the reported invariants (ACVR1B,
  RBM10, ZFP36L2, FAU) are coding drivers
- When inputs are raw somatic mutations rather than the processed feature/response
  tables DriverPower's 'infer' subcommand requires — run the data-preparation and
  model steps first
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_2648
tools:
- name: DriverPower
  role: Runs both 'infer' invocations (burden-only and with --funcScore/--funcScoreCut)
    whose call sets are compared
  repo: https://github.com/smshuai/DriverPower
- name: Python >= 3.5
  role: Runtime environment for DriverPower and for diffing the two result TSVs to
    identify gained/lost drivers
  repo: https://github.com/smshuai/DriverPower
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-13929-1
    title: Combined burden and functional impact tests for cancer driver discovery
      using DriverPower
  source_content_hash: sha256:2e917459347eb4fe731de61955ccd4cb17a45e3af8f90c1932bcdf327dc88301
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/smshuai/DriverPower
  tools:
  - DriverPower
  - Python >=
  - Python >= 3.5
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_2648
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# DriverPower with/without functional-adjustment driver call set comparison

## Summary

Runs DriverPower's 'infer' subcommand twice on the same test features/response — once burden-only (no --funcScore) and once with posterior functional adjustment via --funcScoreCut — then diffs the two coding driver call sets to quantify precision impact (e.g. Panc-AdenoCA: gain ACVR1B, RBM10, ZFP36L2; loss of likely false positive FAU; burden-only precision 74.6%, 156/209 CGC genes).

## When to use

Use when you need to quantify how adding functional impact scores (CADD, DANN, EIGEN, LINSIGHT) changes DriverPower driver calls for a tumour cohort, or to report burden-only precision against a CGC-gene truth set.

## When NOT to use

- When only a single call set is needed and no functional score table (CADD/DANN/EIGEN/LINSIGHT per element) is available — the differential comparison requires both burden-only and functional-adjusted runs
- When working with non-coding driver discovery, since functional impact score schemes are currently biased toward coding mutations and the reported invariants (ACVR1B, RBM10, ZFP36L2, FAU) are coding drivers
- When inputs are raw somatic mutations rather than the processed feature/response tables DriverPower's 'infer' subcommand requires — run the data-preparation and model steps first

## Inputs

- test feature table (test_feature.hdf5) with 1373 genomic features per element
- test response table (per-element nMut/nSample, e.g. from prepare.py)
- pre-trained DriverPower BMR model (GLM and/or GBM) information
- per-element functional score table (TSV keyed by binID; columns CADD, DANN, EIGEN, LINSIGHT) for the functional-adjustment run

## Outputs

- DriverPower.result.tsv — burden-only driver call set (no functional information)
- DriverPower_func.result.tsv — functional-adjusted driver call set
- differential call-set report: drivers gained and lost after functional adjustment
- burden-only precision estimate vs CGC-gene truth set (e.g. 156/209 = 74.6% in Panc-AdenoCA)

## How to apply

First generate the burden-only baseline by running 'driverpower infer' with the test feature table (test_feature.hdf5), test response table and pre-trained model information using --method auto (dispersion-test based selection) and --geoMean True, omitting --funcScore so only the mutational burden test is used; write results to DriverPower.result.tsv. Next, build a per-element functional score table (TSV keyed by binID with one column per scheme, e.g. averaged per-mutation CADD PHRED scores per element). Re-run 'driverpower infer' with identical inputs plus --funcScore and --funcScoreCut in 'NAME:CUTOFF' format with cutoffs in (0,1] (e.g. 'CADD:0.01;DANN:0.03;EIGEN:0.3') to apply posterior functional adjustment, writing DriverPower_func.result.tsv. Diff the two call sets to identify gained and lost drivers, then validate against expected invariants (gained: ACVR1B, RBM10, ZFP36L2; lost: FAU in Panc-AdenoCA) and check the burden-only precision (74.6%, 156/209 CGC genes, Fig. 2a).

## Related tools

- **DriverPower** (Runs both 'infer' invocations (burden-only and with --funcScore/--funcScoreCut) whose call sets are compared) — https://github.com/smshuai/DriverPower
- **Python >= 3.5** (Runtime environment for DriverPower and for diffing the two result TSVs to identify gained/lost drivers) — https://github.com/smshuai/DriverPower

## Examples

```
driverpower infer --testInput test_feature.hdf5 --testResponse test.bed --model model. --method auto --geoMean True --funcScore func_scores.tsv --funcScoreCut "CADD:0.01;DANN:0.03;EIGEN:0.3"
```

## Evaluation signals

- Burden-only run contains no functional adjustment: the command must omit --funcScore, and DriverPower.result.tsv precision matches 74.6% (156/209) for CGC genes as in Fig. 2a
- Functional-adjusted run uses --funcScoreCut with all cutoffs in (0,1], e.g. 'CADD:0.01;DANN:0.03;EIGEN:0.3'
- Diff of the two call sets for Panc-AdenoCA shows exactly three gained drivers (ACVR1B, RBM10, ZFP36L2) and one lost gene (FAU)
- Both runs use identical inputs and identical --method auto and --geoMean True settings, so any call-set differences are attributable to functional adjustment alone
- Output files DriverPower.result.tsv and DriverPower_func.result.tsv exist and contain the same binID schema differing only in driver membership

## Limitations

- Functional impact score schemes are currently biased toward coding mutations, so functional adjustment benefits are best established for coding driver discovery
- False-positive driver calls may arise from variant-calling artefacts or biological processes not captured by the BMR model (e.g. regional AID activity), which burden-only precision against CGC genes cannot fully resolve
- DriverPower's BMR is trained on predominantly neutral regions and does not model negatively selected sites, limiting recall in both call sets

## Evidence

- [methods] call driver candidates without functional information, aka, burden-test only: "call driver candidates **without** functional information, aka, burden-test only"
- [methods] functional score cutoff filtering: "``--funcScoreCut``: [*optional*] Cutoff of each functional impact scores. The format of this parameter is a string in "NAME1:CUTOFF1;NAME2:CUTOFF2...""
- [methods] auto method selection via dispersion test: "Default is auto, which means decision will be made automatically based on the dispersion test of training data."
- [abstract] functional scores increase accuracy: "By incorporating functional impact scores, we are able to further increase the accuracy of driver discovery."
- [readme] DriverPower purpose: "DriverPower is a tool used to discover potential coding and non-coding cancer driver elements from tumour whole-genome or whole-exome somatic mutations."
