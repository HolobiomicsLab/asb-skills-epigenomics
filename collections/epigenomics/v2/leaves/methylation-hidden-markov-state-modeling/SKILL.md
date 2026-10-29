---
name: methylation-hidden-markov-state-modeling
description: Use when bisulfite sequencing data from a heterogeneous sample must be
  deconvolved into major allelic subpopulation methylation profiles over user-defined
  regions, without prior knowledge of subpopulation count.
when_to_use_negative:
- Samples with high 5-hydroxymethylation (5hmC) such as neuronal cell types — DXM
  does not consider 5hmC and interpretation may be impacted
- Regions with allele-specific methylation or copy number variants, where subpopulation
  profiles may reflect allelic or dosage effects rather than distinct cell subpopulations
- Samples with extreme methylation changes such as DNMT3A mutation or demethylating
  agent treatment, unless transition probability parameters are retuned for the dataset
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
tools:
- name: DXM
  role: Implements the HMM with distance-binned 2x2 transition matrices, beta-binomial
    emissions, Viterbi decoding, and iterative termination to deconvolve subpopulation
    methylation profiles
  repo: https://github.com/edwardslab-wustl/dxm
- name: python3
  role: Runtime for DXM (requires Python 3.5+)
- name: numpy
  role: Required Python package for DXM's numerical HMM computations
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkab516
    title: Determining subpopulation methylation profiles from bisulfite sequencing
      data of heterogeneous samples using DXM
  source_content_hash: sha256:72e0c0da8669954c1e562c0b145e74451a139a7683be7bd4479889559e9d4a89
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edwardslab-wustl/dxm
  tools:
  - DXM
  - numpy
  - python3
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# methylation-hidden-markov-state-modeling

## Summary

Model binary methylation states across CpGs within a genomic region as a hidden Markov process, using distance-binned 2x2 transition matrices and beta-binomial emissions to reconstruct per-subpopulation methylation profiles via Viterbi decoding (DXM approach).

## When to use

Use when bisulfite sequencing data from a heterogeneous sample must be deconvolved into major allelic subpopulation methylation profiles over user-defined regions, without prior knowledge of subpopulation count.

## When NOT to use

- Samples with high 5-hydroxymethylation (5hmC) such as neuronal cell types — DXM does not consider 5hmC and interpretation may be impacted
- Regions with allele-specific methylation or copy number variants, where subpopulation profiles may reflect allelic or dosage effects rather than distinct cell subpopulations
- Samples with extreme methylation changes such as DNMT3A mutation or demethylating agent treatment, unless transition probability parameters are retuned for the dataset

## Inputs

- Tab-delimited BED-like methylation file with columns: chromosome, position1, position2, regionName, fractionalMethylation (0–1), coverage (integer)
- BED file of regions of interest (e.g. CGI bed from UCSC Table Browser, or refGene_hg19_10kb_fixed.txt.gz for TSS ±5kb)
- Posterior probability file from dxm_solveMethylation (for downstream dxm_callIDMR)

## Outputs

- <prefix>_reconstructed_1_subpops.txt — regions with 1 major profile (chr, pos1, pos2, region name, methylation state of only subpopulation)
- <prefix>_reconstructed_2_subpops.txt — regions with 2 major profiles (chr, pos1, pos2, region name, minor then major subpopulation methylation state)
- <prefix>_allVitProb.txt — list of all relative posterior probabilities
- <prefix>_DXM_dmrs.txt — intrasample differentially methylated regions from dxm_callIDMR

## How to apply

Read the tab-delimited BED-like methylation file (chromosome, position1, position2, regionName, fractionalMethylation, coverage) and filter positions whose coverage exceeds the maximum expected sequencing coverage set by the -c flag in dxm_solveMethylation (default 400). For each uniquely named region, build a hidden Markov model with 2x2 distance-binned transition matrices governing switches between binary methylation states of each subpopulation from one CpG to the next. Compute beta-binomial emission probabilities (Equation 1) for the observed fractional methylation and coverage at each CpG. Run Viterbi decoding over each region's length-d sequence of binary state vectors to recover per-subpopulation methylation profiles. Apply the termination test comparing relative posterior probabilities p(x_{r+1}|r+1) > p(x_r|r), incrementing the subpopulation count until the minimal count explaining the region is reached. Optionally precede this with dxm_estimateFracs -k to estimate fractional prevalence of subpopulations, though that utility is incompatible with dxm_solveMethylation, which solves prevalence itself.

## Related tools

- **DXM** (Implements the HMM with distance-binned 2x2 transition matrices, beta-binomial emissions, Viterbi decoding, and iterative termination to deconvolve subpopulation methylation profiles) — https://github.com/edwardslab-wustl/dxm
- **python3** (Runtime for DXM (requires Python 3.5+))
- **numpy** (Required Python package for DXM's numerical HMM computations)

## Examples

```
dxm_solveMethylation -i sampleInput.bed -c 400 -o testSample
```

## Evaluation signals

- Outputs follow the documented schema: tab-delimited <prefix>_reconstructed_1_subpops.txt and <prefix>_reconstructed_2_subpops.txt with 5 and 6 columns respectively, plus <prefix>_allVitProb.txt
- All input positions have coverage below the -c maximum coverage threshold (default 400) before solving
- Each region has a unique regionName; fractional methylation values fall between 0 and 1 (divide by 100 if preprocessing outputs 0–100)
- The relative posterior probability file shows the termination criterion p(x_{r+1}|r+1) > p(x_r|r) driving the minimal subpopulation count per region
- DXM predictions can be checked against sorted-cell validation, as done with CD4+ T and CD19+ B cells from DLBCL samples

## Limitations

- DXM does not consider 5-hydroxymethylation (5hmC), which may impact interpreting results in samples with high levels of 5hmC
- Care must be taken in interpreting DXM results in regions with allele-specific methylation or copy number variants
- As regions get larger and include more CpGs, the number of CpGs that must have a methylation change to be solved as a distinct profile increases
- Retuning DXM parameters (e.g. transition probabilities) for genomic compartments or individual datasets could improve accuracy, especially for extreme methylation changes such as DNMT3A mutation
- Computational time scales with number of subpopulations, number of CpGs, and maximum coverage; setting -c above the maximum coverage needlessly increases memory usage

## Evidence

- [methods] 2x2 distance-binned transition matrices governing switches between binary methylation states: [evidence span withheld — anchor status: not_found]
- [readme] maximum coverage filter: "All data should be filtered such that the coverage is below the maximum expected sequencing coverage, set as the -m flag in dxm_solveMethylation."
- [results] Viterbi termination scheme: [evidence span withheld — anchor status: not_found]
- [abstract] no prior subpopulation count required: "DXM does not require prior knowledge of the number of subpopulations or types of cells to expect"
- [readme] output files: "<prefix>_reconstructed_1_subpops.txt  - regions with 1 major profile"
- [readme] computational scaling: "DXM computational time scales with number of subpopulations, number of CpGs, and maximum coverage."
