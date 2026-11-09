---
name: in-silico-mixture-generation-from-reference-profiles
description: Use when benchmarking a methylation deconvolution tool (e.g., DXM) and
  ground-truth subpopulation profiles and prevalences are needed that cannot be obtained
  from real mixed samples.
when_to_use_negative:
- The goal is to validate deconvolution on real heterogeneous biological samples with
  unknown ground truth (e.g., DLBCL patient cohorts), where simulated binarized profiles
  do not reflect biological methylation distributions
- The reference profiles contain partial methylation values that must be preserved
  rather than binarized to 0/1, since this skill specifically generates idealized
  binarized mixtures
- The tool under test models copy-number variation or allele-specific methylation,
  which DXM and this idealized mixture design do not account for
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3295
tools:
- name: DXM
  role: Deconvolution tool whose subpopulation methylation profile solutions on the
    simulated mixtures are benchmarked against ground-truth binarized references
  repo: https://github.com/edwardslab-wustl/dxm
- name: numpy
  role: Python package used to generate the idealized heterogeneous mixtures at fixed
    subpopulation prevalences and coverage depths
- name: python3
  role: Runtime environment (Python 3.5+) for the simulation scripts and for running
    DXM
  repo: https://github.com/edwardslab-wustl/dxm
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkab516
    title: Determining subpopulation methylation profiles from bisulfite sequencing
      data of heterogeneous samples using DXM
  source_content_hash: sha256:1515081eb081db183bcfc173d33f757a8ea16b84f9914bca9bf2e8616251b37a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edwardslab-wustl/dxm
  tools:
  - DXM
  - numpy
  - python3
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# In-silico mixture generation from reference profiles

## Summary

Generates idealized simulated heterogeneous methylation samples by mixing binarized reference cell-type methylation profiles (e.g., HMEC with CD4+ T or HCC1954) at fixed subpopulation prevalences and sequencing coverages using numpy, to benchmark deconvolution methods like DXM.

## When to use

Use when benchmarking a methylation deconvolution tool (e.g., DXM) and ground-truth subpopulation profiles and prevalences are needed that cannot be obtained from real mixed samples.

## When NOT to use

- The goal is to validate deconvolution on real heterogeneous biological samples with unknown ground truth (e.g., DLBCL patient cohorts), where simulated binarized profiles do not reflect biological methylation distributions
- The reference profiles contain partial methylation values that must be preserved rather than binarized to 0/1, since this skill specifically generates idealized binarized mixtures
- The tool under test models copy-number variation or allele-specific methylation, which DXM and this idealized mixture design do not account for

## Inputs

- Sorted cell-type bisulfite sequencing methylation profiles (e.g., HMEC, CD4+ T cells, HCC1954) as fractional methylation values with per-site coverage
- Specified subpopulation prevalences for each simulated mixture
- Target sequencing coverage depths (including >30x) for the simulated mixtures

## Outputs

- Simulated mixture methylation input files in DXM bed-like format (chr, position1, position2, regionName, fractionalMethylation, coverage)
- Ground-truth binarized reference subpopulation profiles for accuracy comparison
- Benchmark accuracy table (e.g., dxm_benchmark_accuracy.csv) with per-region profile accuracy and major-subpopulation identification rate per mixture

## How to apply

Convert sorted cell-type bisulfite methylation data into the DXM tab-delimited bed-like format (chromosome, position1, position2, regionName, fractionalMethylation, coverage), collapsing data across strands and filtering sites so coverage stays below the maximum expected sequencing coverage. Binarize each reference cell-type profile to 0/1 fractional methylation values, then use numpy to generate idealized mixtures at fixed subpopulation prevalences (e.g., HMEC:CD4+T and HMEC:HCC1954 pairs) and coverage depths, including depths greater than 30x. Run the deconvolver on each simulated mixture (for DXM, dxm_solveMethylation with the -c flag set to the maximum coverage in the input file) and compare solved subpopulation profiles against the ground-truth binarized references to compute per-region profile accuracy and major-subpopulation identification rate. At >30x coverage, DXM profile accuracy should average approximately 96.2% for HMEC-CD4+T and 95.1% for HMEC-HCC1954 mixtures, and each mixture should resolve the correct number of major subpopulations.

## Related tools

- **DXM** (Deconvolution tool whose subpopulation methylation profile solutions on the simulated mixtures are benchmarked against ground-truth binarized references) — https://github.com/edwardslab-wustl/dxm
- **numpy** (Python package used to generate the idealized heterogeneous mixtures at fixed subpopulation prevalences and coverage depths)
- **python3** (Runtime environment (Python 3.5+) for the simulation scripts and for running DXM) — https://github.com/edwardslab-wustl/dxm

## Examples

```
dxm_solveMethylation -i simulated_mixture.bed -c 400 -o testMixture
```

## Evaluation signals

- Aggregate solved-profile accuracy across mixtures at >30x coverage approximates the reported benchmark values (~96.2% for HMEC-CD4+T, ~95.1% for HMEC-HCC1954)
- Each simulated mixture resolves the correct number of major subpopulations (two for the HMEC:CD4+T and HMEC:HCC1954 pairs)
- All simulated mixture sites have coverage strictly below the maximum expected sequencing coverage supplied via the -c flag in dxm_solveMethylation
- Reference profiles contain only binarized 0/1 fractional methylation values, and input files follow the 6-column bed-like schema (chr, position1, position2, regionName, fractionalMethylation, coverage)
- Comparison against ground-truth binarized references yields per-region accuracy values and a correctly populated dxm_benchmark_accuracy.csv

## Limitations

- Mixtures are idealized with binarized 0/1 reference profiles, so accuracy estimates do not capture partial methylation states present in real heterogeneous samples
- DXM does not consider 5-hydroxymethylation (5hmC) and care is needed in regions with allele-specific methylation or copy number variants, which this simulation design does not model
- DXM accuracy depends on region size and coverage; larger regions with more CpGs require more CpGs with methylation change to be solved as a distinct profile

## Evidence

- [results] In simulated idealized mixtures, DXM profile accuracy is very high, averaging 96.2% for HMEC-CD4+T cell mixtures and 95.1% for HMEC-HCC1954 cell mixtures when coverage is greater than 30×.: [evidence span withheld — anchor status: not_found]
- [readme] DXM input format and filtering requirements: "The methylation data input file should be in a tab-delimited bed-like format: <chr> <position1> <position2> <regionName> <fractionalMethylation> <coverage>"
- [readme] Coverage filtering recommendation: "All data should be filtered such that the coverage is below the maximum expected sequencing coverage, set as the -m flag in dxm_solveMethylation."
- [methods] Strand collapsing recommendation: "We generally recommend collapsing data across strands prior to running DXM."
- [intro] Benchmarking against methylPurify: "We benchmarked DXM on a wide set of simulated mixtures using bisulfite sequencing reads from sorted hematopoietic cell types and found that DXM outperformed methylPurify"
