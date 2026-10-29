---
name: deconvolution-accuracy-evaluation
description: Use when validating a methylation deconvolution method (e.g. DXM) on
  simulated mixtures with known subpopulation prevalences, where reference profiles
  are binarized to 0/1 and per-region accuracy can be computed.
when_to_use_negative:
- Input is real patient tumor data without a known ground-truth subpopulation composition
  — accuracy cannot be computed; use qualitative validation (e.g. sorted CD4+ T /
  CD19+ B cell recapitulation) instead
- Samples with high 5-hydroxymethylation (e.g. neuronal cell types) — DXM does not
  consider 5hmC, so accuracy estimates are confounded
- Regions with allele-specific methylation or copy number variants — interpretation
  of deconvolution accuracy in these regions requires special care
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3295
tools:
- name: DXM
  role: 'Deconvolution engine: dxm_solveMethylation reconstructs subpopulation methylation
    profiles from each simulated mixture for comparison against ground truth'
  repo: https://github.com/edwardslab-wustl/dxm
- name: python3
  role: Runtime environment for DXM (3.5+) and for the benchmark simulation/accuracy-comparison
    scripts
- name: numpy
  role: Generates the idealized heterogeneous mixtures at fixed subpopulation prevalences
    and coverage depths
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkab516
    title: Determining subpopulation methylation profiles from bisulfite sequencing
      data of heterogeneous samples using DXM
  source_content_hash: sha256:b4e864cfa0db1653f5382ba01b8cd73bba191547f299547b2610cd4c3314d886
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edwardslab-wustl/dxm
  tools:
  - DXM
  - numpy
  - python3
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# deconvolution-accuracy-evaluation

## Summary

Benchmarks DXM's ability to deconvolve subpopulation methylation profiles from simulated idealized heterogeneous bisulfite mixtures (e.g. HMEC with CD4+ T or HCC1954 cells) by comparing solved profiles against binarized ground-truth reference profiles.

## When to use

Use when validating a methylation deconvolution method (e.g. DXM) on simulated mixtures with known subpopulation prevalences, where reference profiles are binarized to 0/1 and per-region accuracy can be computed.

## When NOT to use

- Input is real patient tumor data without a known ground-truth subpopulation composition — accuracy cannot be computed; use qualitative validation (e.g. sorted CD4+ T / CD19+ B cell recapitulation) instead
- Samples with high 5-hydroxymethylation (e.g. neuronal cell types) — DXM does not consider 5hmC, so accuracy estimates are confounded
- Regions with allele-specific methylation or copy number variants — interpretation of deconvolution accuracy in these regions requires special care

## Inputs

- Bisulfite sequencing methylation data from sorted cell types (e.g. HMEC, CD4+ T cells, HCC1954) as per-site fractional methylation with coverage
- Tab-delimited BED-like DXM input files: <chr> <position1> <position2> <regionName> <fractionalMethylation> <coverage>
- Binarized (0/1) reference cell-type methylation profiles
- Simulated heterogeneous mixtures at fixed subpopulation prevalences and coverage depths (e.g. >30x)

## Outputs

- <prefix>_reconstructed_2_subpops.txt per mixture (chromosome, position1, position2, region name, minor subpopulation methylation state, major subpopulation methylation state)
- <prefix>_allVitProb.txt relative posterior probabilities
- Per-region subpopulation profile accuracy and major-subpopulation identification rates aggregated into dxm_benchmark_accuracy.csv

## How to apply

Convert sorted cell-type bisulfite methylation data into the tab-delimited BED-like DXM input format (chromosome, position1, position2, regionName, fractionalMethylation, coverage), collapsing data across strands and filtering sites so coverage stays below the maximum expected sequencing coverage (-m in dxm_solveMethylation). Binarize reference profiles to 0/1 and generate idealized heterogeneous mixtures at fixed subpopulation prevalences and coverage depths (including >30x) with numpy. Run dxm_solveMethylation on each mixture with -c set to the maximum coverage in the input file (default 400) and a per-mixture -o prefix, producing <prefix>_reconstructed_2_subpops.txt and <prefix>_allVitProb.txt. Compare solved subpopulation profiles to the ground-truth binarized references to compute per-region profile accuracy and major-subpopulation identification rate, then aggregate accuracy across mixtures restricted to >30x coverage. Correct application is confirmed when aggregated accuracy approximates the reported benchmarks (~96.2% for HMEC:CD4+T and ~95.1% for HMEC:HCC1954) and each mixture resolves the correct number of major subpopulations.

## Related tools

- **DXM** (Deconvolution engine: dxm_solveMethylation reconstructs subpopulation methylation profiles from each simulated mixture for comparison against ground truth) — https://github.com/edwardslab-wustl/dxm
- **python3** (Runtime environment for DXM (3.5+) and for the benchmark simulation/accuracy-comparison scripts)
- **numpy** (Generates the idealized heterogeneous mixtures at fixed subpopulation prevalences and coverage depths)

## Examples

```
dxm_solveMethylation -i simulatedMixture.bed -c 400 -o mixtureTest
```

## Evaluation signals

- Aggregated solved-profile accuracy across mixtures at >30x coverage approximates the reported benchmarks: ~96.2% for HMEC:CD4+T and ~95.1% for HMEC:HCC1954 mixtures
- Each simulated mixture resolves the correct number of major subpopulations in the dxm_solveMethylation output files
- Solved fractional methylation states in <prefix>_reconstructed_2_subpops.txt match binarized (0/1) ground-truth reference profiles per region
- Input coverage values are all below the maximum expected sequencing coverage passed via -m, and -c is set to the maximum coverage in the input file
- Output files <prefix>_reconstructed_1_subpops.txt, <prefix>_reconstructed_2_subpops.txt, and <prefix>_allVitProb.txt exist and follow the documented tab-delimited schemas

## Limitations

- Accuracy is only valid at sufficient depth: benchmarked accuracy was reported when coverage is greater than 30x; lower coverage is not guaranteed
- Reference profiles must be binarized to 0/1 for this idealized benchmark; real mixtures have intermediate methylation states
- DXM does not consider 5-hydroxymethylation (5hmC), which may impact interpreting results in samples with high levels of 5hmC
- Care must be taken interpreting DXM results in regions with allele-specific methylation or copy number variants
- Retuning DXM parameters (e.g. transition probabilities) for genomic compartments or individual datasets could improve accuracy, particularly for extreme methylation changes such as DNMT3A mutation

## Evidence

- [results] Benchmark accuracy on simulated idealized mixtures: [evidence span withheld — anchor status: not_found]
- [readme] Input format and coverage filtering rationale: "All data should be filtered such that the coverage is below the maximum expected sequencing coverage, set as the -m flag in dxm_solveMethylation."
- [readme] dxm_solveMethylation invocation and -c parameter: "dxm_solveMethylation -i sampleInput.bed -c 400 -o testSample"
- [readme] Output files of dxm_solveMethylation: "The outputs of dxm_solveMethylation are: - <prefix>_reconstructed_1_subpops.txt  - regions with 1 major profile"
- [methods] Strand collapsing recommendation: "We generally recommend collapsing data across strands prior to running DXM."
