---
name: cadd-functional-score-integration
description: Use when calling cancer driver candidates with DriverPower and you want
  to incorporate functional impact evidence (CADD scores) alongside mutational burden
  tests, on coding or non-coding element sets.
when_to_use_negative:
- You only need burden-test-only driver calls without functional adjustment — omit
  --funcScore and call candidates without functional information
- Input mutations lack hg19 coordinates or CADD-compatible variant identifiers, so
  per-mutation CADD scores cannot be retrieved or intersected with elements
- The functional score scheme is not CADD — functional impact score schemes are currently
  biased toward coding mutations, so non-coding integration may add little signal
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0622
tools:
- name: DriverPower
  role: Consumes per-element CADD scores via --funcScore/--funcScoreCut during 'driverpower
    infer' to adjust burden tests with functional impact evidence
  repo: https://github.com/smshuai/DriverPower
- name: bedtools
  role: Intersects per-mutation CADD score files with test elements (bedtools intersect)
    and aggregates to per-element mean scores (bedtools groupby)
- name: XGBoost
  role: Provides the pre-trained GBM background mutation rate models (poisson objective,
    default 3-fold) used alongside the CADD-adjusted inference
  repo: https://github.com/dmlc/xgboost/blob/master/doc/parameter.md
provenance:
  source_task_ids:
  - task_006
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-13929-1
    title: Combined burden and functional impact tests for cancer driver discovery
      using DriverPower
  source_content_hash: sha256:05a3ad4eb209b6f71c836ba4a5f95550b8cb36b0fccb318d221784889704d0f4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/smshuai/DriverPower
  tools:
  - DriverPower
  - Python >=
  - XGBoost
  - bedtools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# cadd-functional-score-integration

## Summary

Integrates per-mutation CADD PHRED scores into DriverPower's burden test as a functional impact covariate, aggregating per-element mean CADD scores with bedtools and supplying them via --funcScore/--funcScoreCut to increase driver discovery accuracy.

## When to use

Use when calling cancer driver candidates with DriverPower and you want to incorporate functional impact evidence (CADD scores) alongside mutational burden tests, on coding or non-coding element sets.

## When NOT to use

- You only need burden-test-only driver calls without functional adjustment — omit --funcScore and call candidates without functional information
- Input mutations lack hg19 coordinates or CADD-compatible variant identifiers, so per-mutation CADD scores cannot be retrieved or intersected with elements
- The functional score scheme is not CADD — functional impact score schemes are currently biased toward coding mutations, so non-coding integration may add little signal

## Inputs

- Somatic mutation callset (e.g. GDC MuTect2 WES VCFs or PCAWG WGS mutations, hg19)
- Test element set (CDS, splice site, 5'-UTR, 3'-UTR, promoter, enhancer, lncRNA, lncRNA promoter; e.g. test_elements.bed12.gz)
- Test feature table (test_feature.hdf5) and test response table (nMut, nSample, effective length per element)
- Pre-trained GBM BMR model information (e.g. trained on PCAWG Liver-HCC or Panc-AdenoCA features)
- Per-mutation CADD PHRED score file from the CADD web interface

## Outputs

- Per-element mean CADD score table (e.g. CADD_per_ele_score.tsv)
- Driver candidate list with q-values per element set in '[outDir]/[name].result.tsv'
- Aggregated coding and non-coding candidate lists written to DriverPower.result.tsv

## How to apply

Retrieve per-mutation CADD PHRED scores via the CADD web interface (up to 100K variants per query, avoiding the ~80 GB full SNV file), then compute per-element mean scores by intersecting the score file with test elements using bedtools intersect and aggregating with bedtools groupby (mean). Run 'driverpower infer' with the test feature table (e.g. test_feature.hdf5), test response table, and the pre-trained BMR model, passing the per-element CADD table via --funcScore and a cutoff via --funcScoreCut in the format 'CADD:0.01'. Leave --method at its default 'auto' (dispersion-test-based choice between binomial and negative binomial) and --geoMean at its default True (geometric mean of nMut and nSample). Apply Benjamini-Hochberg correction to p-values and retain driver candidates with q < 0.1 from '[outDir]/[name].result.tsv'. Rationale: the paper shows that incorporating functional impact scores further increases the accuracy of driver discovery beyond burden-only testing.

## Related tools

- **DriverPower** (Consumes per-element CADD scores via --funcScore/--funcScoreCut during 'driverpower infer' to adjust burden tests with functional impact evidence) — https://github.com/smshuai/DriverPower
- **bedtools** (Intersects per-mutation CADD score files with test elements (bedtools intersect) and aggregates to per-element mean scores (bedtools groupby))
- **XGBoost** (Provides the pre-trained GBM background mutation rate models (poisson objective, default 3-fold) used alongside the CADD-adjusted inference) — https://github.com/dmlc/xgboost/blob/master/doc/parameter.md

## Examples

```
driverpower infer --inFile test_feature.hdf5 --response test_y.tsv --model modelInfo. --funcScore CADD_per_ele_score.tsv --funcScoreCut "CADD:0.01"
```

## Evaluation signals

- On PCAWG (2583 genomes), the CADD-integrated run reproduces 217 coding and 95 non-coding driver candidates at q < 0.1
- On TCGA-LIHC WES with Liver-HCC-trained models, 14 coding drivers are recovered in 364 samples, all but one known driver genes
- On TCGA-PAAD WES with Panc-AdenoCA-trained models, 6 coding drivers are recovered in 180 samples, all known driver genes
- The per-element CADD table contains one mean CADD PHRED row per test element after bedtools intersect/groupby
- Output '[outDir]/[name].result.tsv' exists with BH-corrected q-values and candidates retained at q < 0.1

## Limitations

- Functional impact score schemes are currently biased toward coding mutations, limiting power for non-coding drivers
- False-positive driver calls may arise from variant-calling artefacts or biological processes not captured by the BMR model (e.g. regional AID activity)
- Only ~3.7% of the genome was studied; more non-coding drivers may exist in other regulatory elements requiring more complete annotations
- DriverPower's BMR is trained on predominantly neutral regions and does not model negatively selected sites, limiting recall

## Evidence

- [methods] CADD web interface retrieval: [evidence span withheld — anchor status: not_found]
- [methods] funcScoreCut format: "--funcScoreCut "CADD:0.01""
- [methods] funcScoreCut parameter format: "``--funcScoreCut``: [*optional*] Cutoff of each functional impact scores. The format of this parameter is a string in "NAME1:CUTOFF1;NAME2:CUTOFF2...""
- [methods] auto method selection: "Default is auto, which means decision will be made automatically based on the dispersion test of training data."
- [abstract] PCAWG result with CADD: "Testing across a collection of 2583 cancer genomes from the PCAWG project, DriverPower identifies 217 coding and 95 non-coding driver candidates."
- [abstract] accuracy motivation: "By incorporating functional impact scores, we are able to further increase the accuracy of driver discovery."
- [readme] README citation/context: "DriverPower is a tool used to discover potential coding and non-coding cancer driver elements from tumour whole-genome or whole-exome somatic mutations."
