---
name: multiple-testing-qvalue-correction
description: 'Use when a genome-wide scan (e.g., driverpower infer over eight test
  element sets: CDS, splice site, 5''-UTR, 3''-UTR, promoter, enhancer, lncRNA, lncRNA
  promoter) has produced raw p-values that must be converted to q-values before calling
  driver candidates.'
when_to_use_negative:
- Input is a single pre-specified genomic element tested without a genome-wide multiplicity
  burden (no FDR correction needed)
- The background mutation rate (BMR) model has not yet been trained — run 'driverpower
  model' on train_feature.hdf5 and the training response table first
- Variants lack CADD PHRED scores and burden-test-only calling without functional
  adjustment is the intended mode
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0203
tools:
- name: DriverPower
  role: Computes per-element burden/functional-impact p-values via 'driverpower infer';
    Benjamini-Hochberg correction at q < 0.1 turns these into the driver candidate
    call set
  repo: https://github.com/smshuai/DriverPower
- name: bedtools
  role: Intersects CADD PHRED score file with test elements and aggregates mean per-element
    scores used as functional evidence during inference
- name: Python >= 3.5
  role: Runtime environment (installable via Anaconda/conda or pip) in which DriverPower
    and its post-hoc FDR aggregation are executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-13929-1
    title: Combined burden and functional impact tests for cancer driver discovery
      using DriverPower
  source_content_hash: sha256:192d3de310ff0119f52e2b18d336dacfdc90cc1bf28cbd53d08ceace1e931b3a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/smshuai/DriverPower
  tools:
  - DriverPower
  - Python >=
  - Python >= 3.5
  - bedtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# multiple-testing-qvalue-correction

## Summary

Applies Benjamini-Hochberg FDR correction to per-element p-values from DriverPower's burden/functional-impact tests and retains driver candidates with q < 0.1, controlling the false discovery rate when testing thousands of coding and non-coding elements across 2583 PCAWG cancer genomes.

## When to use

Use when a genome-wide scan (e.g., driverpower infer over eight test element sets: CDS, splice site, 5'-UTR, 3'-UTR, promoter, enhancer, lncRNA, lncRNA promoter) has produced raw p-values that must be converted to q-values before calling driver candidates.

## When NOT to use

- Input is a single pre-specified genomic element tested without a genome-wide multiplicity burden (no FDR correction needed)
- The background mutation rate (BMR) model has not yet been trained — run 'driverpower model' on train_feature.hdf5 and the training response table first
- Variants lack CADD PHRED scores and burden-test-only calling without functional adjustment is the intended mode

## Inputs

- Raw p-values per test element from driverpower infer (binomial or negative binomial, chosen by auto dispersion test)
- test_feature.hdf5 (test feature table) and test response table
- Pre-trained BMR model information from the 'driverpower model' step
- Per-element CADD PHRED mean scores (CADD_per_ele_score.tsv) with functional cutoff CADD:0.01

## Outputs

- Corrected q-values (Benjamini-Hochberg) with driver candidates retained at q < 0.1
- Aggregated coding and non-coding driver candidate lists in DriverPower.result.tsv (expected: 217 coding and 95 non-coding candidates on PCAWG)

## How to apply

Run 'driverpower infer' per test element set with the test feature table (test_feature.hdf5), the test response table, and the pre-trained BMR model, supplying per-element CADD scores via --funcScore CADD_per_ele_score.tsv and --funcScoreCut 'CADD:0.01'. Keep --method auto, which selects between binomial and negative binomial p-values via the dispersion test of the training data, and --geoMean True (geometric mean of nMut and nSample). After inference, apply Benjamini-Hochberg correction to the p-values and retain driver candidates with q < 0.1. Finally, aggregate the coding and non-coding candidate lists across element sets into DriverPower.result.tsv.

## Related tools

- **DriverPower** (Computes per-element burden/functional-impact p-values via 'driverpower infer'; Benjamini-Hochberg correction at q < 0.1 turns these into the driver candidate call set) — https://github.com/smshuai/DriverPower
- **bedtools** (Intersects CADD PHRED score file with test elements and aggregates mean per-element scores used as functional evidence during inference)
- **Python >= 3.5** (Runtime environment (installable via Anaconda/conda or pip) in which DriverPower and its post-hoc FDR aggregation are executed)

## Examples

```
driverpower infer --testInput test_feature.hdf5 --response test_response.tsv --model model. --funcScore CADD_per_ele_score.tsv --funcScoreCut 'CADD:0.01' --method auto --geoMean True
```

## Evaluation signals

- Applied to 2583 PCAWG cancer genomes across the eight element sets, the corrected call set reproduces the reported 217 coding and 95 non-coding driver candidates
- Every retained candidate has q < 0.1 after Benjamini-Hochberg correction and no corrected candidate exceeds that threshold
- --method auto resolution matches the dispersion test outcome of the training data (binomial vs. negative binomial p-values)
- Functional filtering flag --funcScoreCut 'CADD:0.01' is honoured: only elements meeting the CADD cutoff receive the functional-adjusted p-value
- Results aggregate cleanly across the eight element sets into a single DriverPower.result.tsv with no duplicated or lost elements

## Limitations

- False-positive driver calls may arise from variant-calling artefacts and biological processes not captured by the BMR model (e.g. regional AID activity)
- Many tumour types have fewer than 30 samples, limiting statistical power even after FDR control (only 2583 genomes across many types)
- Functional impact score schemes are currently biased toward coding mutations, so functional adjustment of non-coding element p-values is imperfect

## Evidence

- [abstract] Testing across a collection of 2583 cancer genomes from the PCAWG project, DriverPower identifies 217 coding and 95 non-coding driver candidates.: "Testing across a collection of 2583 cancer genomes from the PCAWG project, DriverPower identifies 217 coding and 95 non-coding driver candidates."
- [methods] Default is auto, which means decision will be made automatically based on the dispersion test of training data.: "Default is auto, which means decision will be made automatically based on the dispersion test of training data."
- [methods] --funcScoreCut: [optional] Cutoff of each functional impact scores.: "``--funcScoreCut``: [*optional*] Cutoff of each functional impact scores. The format of this parameter is a string in "NAME1:CUTOFF1;NAME2:CUTOFF2...""
- [methods] --geoMean: [optional] Whether to use geometric mean of nMut and nSample in test.: "``--geoMean``: [*optional*] Whether to use geometric mean of nMut and nSample in test. Default is ``True``."
- [methods] The infer sub-command is used to call drivers from test data with pre-trained models.: "The ``infer`` sub-command is used to call drivers from test data with pre-trained models."
