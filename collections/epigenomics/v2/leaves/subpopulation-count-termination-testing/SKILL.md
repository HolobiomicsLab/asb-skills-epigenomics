---
name: subpopulation-count-termination-testing
description: Use when deconvolving bisulfite sequencing data from heterogeneous samples
  where the number of allelic subpopulations per region is unknown and must be inferred
  from the data itself.
when_to_use_negative:
- Regions where interpretation is confounded by allele-specific methylation or copy
  number variants, which can mimic subpopulation structure
- Samples with high 5-hydroxymethylation (e.g. neuronal cell types), since DXM does
  not consider 5hmC
- When the number of subpopulations is already known and only prevalence estimation
  is needed — dxm_estimateFracs is incompatible with dxm_solveMethylation
edam_operation: http://edamontology.org/operation_3565
edam_topics:
- http://edamontology.org/topic_3674
tools:
- name: DXM
  role: Implements the HMM with 2x2 transition matrices, beta-binomial emissions,
    Viterbi decoding, and the iterative termination test via dxm_solveMethylation
  repo: https://github.com/edwardslab-wustl/dxm
- name: python3
  role: Runtime environment required by DXM (3.5+)
- name: numpy
  role: Required Python package for DXM's numerical HMM computations
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkab516
    title: Determining subpopulation methylation profiles from bisulfite sequencing
      data of heterogeneous samples using DXM
  source_content_hash: sha256:cffdfae48e7ce50b312a35fe24cd74df4369e4472f07a4e42f0c15b52f46367a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edwardslab-wustl/dxm
  tools:
  - DXM
  - numpy
  - python3
  edam_operation: http://edamontology.org/operation_3565
  edam_topics:
  - http://edamontology.org/topic_3674
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# subpopulation-count-termination-testing

## Summary

DXM's iterative Viterbi-based termination test determines the minimal number of underlying subpopulations needed to explain a region's bisulfite methylation data, without prior knowledge of the subpopulation count.

## When to use

Use when deconvolving bisulfite sequencing data from heterogeneous samples where the number of allelic subpopulations per region is unknown and must be inferred from the data itself.

## When NOT to use

- Regions where interpretation is confounded by allele-specific methylation or copy number variants, which can mimic subpopulation structure
- Samples with high 5-hydroxymethylation (e.g. neuronal cell types), since DXM does not consider 5hmC
- When the number of subpopulations is already known and only prevalence estimation is needed — dxm_estimateFracs is incompatible with dxm_solveMethylation

## Inputs

- Tab-delimited BED-like methylation file with columns: chromosome, position1, position2, regionName, fractionalMethylation (0-1), coverage (integer)
- Per-CpG fractional methylation and coverage from bisulfite sequencing (e.g. bismark .cov output converted to BED-like format)

## Outputs

- <prefix>_reconstructed_1_subpops.txt — regions with one major methylation profile
- <prefix>_reconstructed_2_subpops.txt — regions with two major methylation profiles (minor then major subpopulation states)
- <prefix>_allVitProb.txt — list of all relative posterior probabilities used by the termination test

## How to apply

For each uniquely named region in the tab-delimited BED-like input (chr, position1, position2, regionName, fractionalMethylation, coverage), build a hidden Markov model with 2x2 distance-binned transition matrices governing binary methylation state switches between CpGs. Compute beta-binomial emission probabilities (Equation 1) from the observed fractional methylation and coverage at each CpG, then run Viterbi decoding over the region's length-d sequence of binary state vectors to recover per-subpopulation methylation profiles. Starting from one subpopulation, apply the termination test comparing relative posterior probabilities — increment the subpopulation count while p(x_{r+1}|r+1) > p(x_r|r) — until the minimal count explaining the region is reached. Write the solved profiles and relative posterior probabilities to the output prefix; coverage should be pre-filtered below the maximum expected sequencing coverage via the -c flag (default 400).

## Related tools

- **DXM** (Implements the HMM with 2x2 transition matrices, beta-binomial emissions, Viterbi decoding, and the iterative termination test via dxm_solveMethylation) — https://github.com/edwardslab-wustl/dxm
- **python3** (Runtime environment required by DXM (3.5+))
- **numpy** (Required Python package for DXM's numerical HMM computations)

## Examples

```
dxm_solveMethylation -i sampleInput.bed -c 400 -o testSample
```

## Evaluation signals

- Output files <prefix>_reconstructed_1_subpops.txt, <prefix>_reconstructed_2_subpops.txt, and <prefix>_allVitProb.txt are produced under the chosen output prefix
- Relative posterior probabilities in <prefix>_allVitProb.txt show the termination condition met: p(x_{r+1}|r+1) is not greater than p(x_r|r) for the final subpopulation count
- Each region in the reconstructed profile files has a unique regionName and methylation states consistent with the input coverage-filtered data
- Runtime and memory scale sensibly with the -c maximum coverage flag; -c should not exceed the maximum observed coverage in the input file

## Limitations

- DXM computational time scales with number of subpopulations, number of CpGs, and maximum coverage; solving eRRBS and Methyl-Seq samples for 2 subpopulations at average 60x coverage takes ~2 hours
- Retuning DXM parameters (e.g. transition probabilities) for specific genomic compartments or datasets with extreme methylation changes (e.g. DNMT3A mutation or demethylating agent treatment) may be needed to improve accuracy
- Region size affects sensitivity: as regions get larger and include more CpGs, more CpGs must show methylation change to be solved as a distinct methylation profile

## Evidence

- [methods] Viterbi decoding with iterative termination test: [evidence span withheld — anchor status: not_found]
- [abstract] No prior knowledge of subpopulation count: "DXM does not require prior knowledge of the number of subpopulations or types of cells to expect"
- [readme] Coverage filtering flag: "All data should be filtered such that the coverage is below the maximum expected sequencing coverage, set as the -m flag in dxm_solveMethylation."
- [readme] Output files from dxm_solveMethylation: "The outputs of dxm_solveMethylation are: - <prefix>_reconstructed_1_subpops.txt  - regions with 1 major profile"
- [readme] Computational scaling: "DXM computational time scales with number of subpopulations, number of CpGs, and maximum coverage. DXM solved eRRBS and Methyl-Seq samples for 2 subpopulations at average coverage of 60x in ~2 hours."
