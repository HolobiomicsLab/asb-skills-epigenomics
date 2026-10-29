---
name: viterbi-decoding-of-binary-methylation-vectors
description: Use when bisulfite sequencing data from a heterogeneous sample has been
  reduced to per-CpG fractional methylation and coverage over user-defined regions
  (e.g. CGIs), and the most likely methylation state vector of each underlying subpopulation
  across the region must be recovered.
when_to_use_negative:
- Input regions have extreme methylation changes (e.g. DNMT3A mutation or demethylating
  agent treatment) without retuning transition probability parameters first
- Regions with allele-specific methylation or copy number variants, where subpopulation
  deconvolution results are hard to interpret
- Samples with high 5-hydroxymethylation (e.g. neuronal cell types), since DXM does
  not consider 5hmC
edam_operation: http://edamontology.org/operation_0436
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3295
tools:
- name: DXM
  role: Runs the HMM construction, beta-binomial emissions, Viterbi decoding, and
    iterative termination test via dxm_solveMethylation
  repo: https://github.com/edwardslab-wustl/dxm
- name: python3
  role: Runtime environment (3.5+) required to install and execute DXM
  repo: https://github.com/edwardslab-wustl/dxm
- name: numpy
  role: Required Python package backing DXM's numeric HMM calculations
  repo: https://github.com/edwardslab-wustl/dxm
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkab516
    title: Determining subpopulation methylation profiles from bisulfite sequencing
      data of heterogeneous samples using DXM
  source_content_hash: sha256:c4fb3f30b90534125eeadf349aa5e0eea53bda14fcee7822b1a56b926e14a0f2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edwardslab-wustl/dxm
  tools:
  - DXM
  - numpy
  - python3
  edam_operation: http://edamontology.org/operation_0436
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# viterbi-decoding-of-binary-methylation-vectors

## Summary

Reconstructs the most likely per-subpopulation methylation profiles across a genomic region by running Viterbi decoding over a hidden Markov model with 2x2 distance-binned transition matrices and beta-binomial emissions on fractional methylation and coverage, as implemented in DXM's dxm_solveMethylation.

## When to use

Use when bisulfite sequencing data from a heterogeneous sample has been reduced to per-CpG fractional methylation and coverage over user-defined regions (e.g. CGIs), and the most likely methylation state vector of each underlying subpopulation across the region must be recovered.

## When NOT to use

- Input regions have extreme methylation changes (e.g. DNMT3A mutation or demethylating agent treatment) without retuning transition probability parameters first
- Regions with allele-specific methylation or copy number variants, where subpopulation deconvolution results are hard to interpret
- Samples with high 5-hydroxymethylation (e.g. neuronal cell types), since DXM does not consider 5hmC

## Inputs

- Tab-delimited BED-like methylation file: chromosome, position1, position2, regionName, fractionalMethylation (0-1), integer coverage (e.g. produced via bedtools intersect against cgi.bed)
- User-defined region set (e.g. UCSC CGI bed file or refGene_hg19_10kb_fixed.txt.gz from example_data)
- Maximum coverage threshold via the -c flag (default 400)

## Outputs

- <prefix>_reconstructed_1_subpops.txt — regions with 1 major profile (chr, pos1, pos2, region name, methylation state of major subpopulation)
- <prefix>_reconstructed_2_subpops.txt — regions with 2 major methylation profiles (adds methylation state of minor subpopulation)
- <prefix>_allVitProb.txt — list of all relative posterior probabilities p(x_r|r)

## How to apply

Read a tab-delimited BED-like file with columns chromosome, position1, position2, regionName, fractionalMethylation (0-1), and integer coverage, filtering positions whose coverage exceeds the -c flag (default 400; should match the maximum expected sequencing coverage set via -m). For each uniquely named region, build a hidden Markov model over binary methylation state vectors using 2x2 distance-binned transition matrices that capture transitions of each subpopulation's methylation state from one CpG to the next, and compute beta-binomial emission probabilities (Equation 1) for the observed fractional methylation and coverage at each CpG. Run Viterbi decoding over the region's length-d sequence of binary state vectors to recover per-subpopulation profiles. Apply the iterative termination scheme: compare relative posterior probabilities using the test p(x_{r+1}|r+1) > p(x_r|r) and increment the subpopulation count until the minimal count explaining the region is reached. Write the reconstructed 1-subpopulation and 2-subpopulation profile files plus the allVitProb relative posterior probability file under the chosen output prefix.

## Related tools

- **DXM** (Runs the HMM construction, beta-binomial emissions, Viterbi decoding, and iterative termination test via dxm_solveMethylation) — https://github.com/edwardslab-wustl/dxm
- **python3** (Runtime environment (3.5+) required to install and execute DXM) — https://github.com/edwardslab-wustl/dxm
- **numpy** (Required Python package backing DXM's numeric HMM calculations) — https://github.com/edwardslab-wustl/dxm

## Examples

```
dxm_solveMethylation -i sampleInput.bed -c 400 -o testSample
```

## Evaluation signals

- The three expected output files (<prefix>_reconstructed_1_subpops.txt, <prefix>_reconstructed_2_subpops.txt, <prefix>_allVitProb.txt) exist and follow the documented tab-delimited column formats
- The termination test p(x_{r+1}|r+1) > p(x_r|r) converges, i.e. relative posterior probabilities stop improving as subpopulation count increments, so the minimal subpopulation count is reported
- Reconstructed methylation states are binary vectors per subpopulation, and fractional methylation values in the input lie between 0 (fully unmethylated) and 1 (fully methylated)
- Runtime scales with number of subpopulations, number of CpGs, and maximum coverage; setting -c far above the input's maximum coverage needlessly increases memory usage, so verify -c matches the input
- Predicted subpopulation profiles should be recapitulated in relevant sorted cell subpopulations (e.g. CD4+ T and CD19+ B cells in DLBCL validation)

## Limitations

- DXM does not consider 5-hydroxymethylation (5hmC), which may impact interpretation in samples with high 5hmC such as neuronal cell types
- Care must be taken interpreting results in regions with allele-specific methylation or copy number variants
- As regions get larger and include more CpGs, the number of CpGs that must show a methylation change to be solved as a distinct profile increases; solving a whole-chromosome profile is impractical given assembly gaps and repetitive DNA
- Retuning transition probabilities per genomic compartment or dataset (e.g. extreme methylation changes from DNMT3A mutation) may be needed to improve accuracy
- Computational time scales with number of subpopulations, number of CpGs, and maximum coverage (~2 hours for 2 subpopulations at ~60x average coverage)

## Evidence

- [results] To calculate transition probabilities, DXM uses a set of 2 × 2 transition matrices that capture transitions of one subpopulation's methylation state from one CpG to the next.: [evidence span withheld — anchor status: not_found]
- [results] Apply the termination test comparing relative posterior probabilities p(x_{r+1}|r+1) > p(x_r|r), incrementing the subpopulation count until the minimal count explaining the region is reached.: [evidence span withheld — anchor status: not_found]
- [results] Run Viterbi decoding over each region's length-d sequence of binary state vectors to recover per-subpopulation methylation profiles.: [evidence span withheld — anchor status: not_found]
- [readme] All data should be filtered such that the coverage is below the maximum expected sequencing coverage, set as the -m flag in dxm_solveMethylation.: "All data should be filtered such that the coverage is below the maximum expected sequencing coverage, set as the -m flag in dxm_solveMethylation."
- [readme] DXM computational time scales with number of subpopulations, number of CpGs, and maximum coverage. DXM solved eRRBS and Methyl-Seq samples for 2 subpopulations at average coverage of 60x in ~2 hours.: "DXM computational time scales with number of subpopulations, number of CpGs, and maximum coverage. DXM solved eRRBS and Methyl-Seq samples for 2 subpopulations at average coverage of 60x in ~2 hours."
