---
name: beta-binomial-emission-probability-computation
description: Use when an HMM-style methylation deconvolution (e.g. DXM's Viterbi decoding
  over user-defined regions) needs emission probabilities for observed fractional
  methylation plus read coverage at each CpG of a region.
when_to_use_negative:
- Input is raw bisulfite reads or aligner output rather than a processed BED-like
  fractional-methylation/coverage file — convert and collapse across strands first
- Samples with high 5-hydroxymethylation (e.g. neuronal cell types), since DXM does
  not consider 5hmC
- Regions with allele-specific methylation or copy number variants, where care must
  be taken interpreting DXM results
edam_operation: http://edamontology.org/operation_0436
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: DXM
  role: Implements the HMM with 2x2 distance-binned transition matrices, beta-binomial
    emission computation (Equation 1), Viterbi decoding, and the iterative subpopulation
    termination test via dxm_solveMethylation
  repo: https://github.com/edwardslab-wustl/dxm
- name: python3
  role: Runtime required by DXM (3.5+) in which the emission and decoding computations
    are written
  repo: https://github.com/edwardslab-wustl/dxm
- name: numpy
  role: Required Python package backing DXM's vectorized HMM/emission calculations
  repo: https://github.com/edwardslab-wustl/dxm
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkab516
    title: Determining subpopulation methylation profiles from bisulfite sequencing
      data of heterogeneous samples using DXM
  source_content_hash: sha256:e8508ab2cec9353cbe506d6153767ef6d8d06793ca71dd670574ff4145a0dbf7
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# beta-binomial-emission-probability-computation

## Summary

Computes per-CpG emission likelihoods for a hidden Markov model over bisulfite sequencing methylation data, using the beta-binomial distribution (DXM Equation 1) applied to each CpG's fractional methylation value and integer coverage so the decoder can recover per-subpopulation methylation states.

## When to use

Use when an HMM-style methylation deconvolution (e.g. DXM's Viterbi decoding over user-defined regions) needs emission probabilities for observed fractional methylation plus read coverage at each CpG of a region.

## When NOT to use

- Input is raw bisulfite reads or aligner output rather than a processed BED-like fractional-methylation/coverage file — convert and collapse across strands first
- Samples with high 5-hydroxymethylation (e.g. neuronal cell types), since DXM does not consider 5hmC
- Regions with allele-specific methylation or copy number variants, where care must be taken interpreting DXM results

## Inputs

- Tab-delimited BED-like methylation file: <chr> <position1> <position2> <regionName> <fractionalMethylation> <coverage>, with fractional methylation in [0,1] and integer coverage
- Maximum expected sequencing coverage threshold (set via the -c flag of dxm_solveMethylation, default 400)
- Optionally a CGI or promoter bed file (e.g. cgi.bed from UCSC Table Browser) used with bedtools intersect to assign region names

## Outputs

- Per-CpG beta-binomial emission likelihoods consumed internally by Viterbi decoding
- <prefix>_allVitProb.txt — list of all relative posterior probabilities used in the termination test
- <prefix>_reconstructed_1_subpops.txt and <prefix>_reconstructed_2_subpops.txt with per-subpopulation methylation states per region

## How to apply

First convert methylation calls to a tab-delimited BED-like file with columns chromosome, position1, position2, regionName, fractionalMethylation, coverage, ensuring fractional methylation is scaled to the 0-1 range and coverage is an integer filtered below the maximum expected coverage (-c/-m, default 400). For each uniquely named region, build the HMM over binary methylation states using 2x2 distance-binned transition matrices governing state switches between adjacent CpGs. Compute the beta-binomial emission probability (Equation 1 in DXM) for the observed fractional methylation and coverage at every CpG, which accounts for sampling uncertainty at low coverage rather than treating the fraction as exact. Feed these emissions into Viterbi decoding over the region's length-d sequence of binary state vectors, and use the relative posterior probabilities p(x_{r+1}|r+1) > p(x_r|r) in the iterative termination scheme to decide the minimal number of subpopulations explaining the region.

## Related tools

- **DXM** (Implements the HMM with 2x2 distance-binned transition matrices, beta-binomial emission computation (Equation 1), Viterbi decoding, and the iterative subpopulation termination test via dxm_solveMethylation) — https://github.com/edwardslab-wustl/dxm
- **python3** (Runtime required by DXM (3.5+) in which the emission and decoding computations are written) — https://github.com/edwardslab-wustl/dxm
- **numpy** (Required Python package backing DXM's vectorized HMM/emission calculations) — https://github.com/edwardslab-wustl/dxm

## Examples

```
dxm_solveMethylation -i sampleInput.bed -c 400 -o testSample
```

## Evaluation signals

- Fractional methylation column is normalized to 0-1 (divide by 100 if the upstream tool outputs 0-100); DXM generates relative coordinates internally, so chromosome/position2 are unused for the HMM itself
- All input positions have coverage below the maximum expected coverage set by -c/-m (default 400); raising -c above the max coverage only needlessly increases memory usage
- Each region has a unique regionName, since DXM computes across all CpGs of a given region
- Run produces <prefix>_reconstructed_1_subpops.txt, <prefix>_reconstructed_2_subpops.txt, and <prefix>_allVitProb.txt; the termination test p(x_{r+1}|r+1) > p(x_r|r) selects the minimal subpopulation count
- The 2-subpopulation output has the expected column order: chromosome, position1, position2, region name, minor subpopulation state, major subpopulation state

## Limitations

- DXM does not consider 5-hydroxymethylation (5hmC), which may impact interpretation in samples with high 5hmC such as neuronal cell types
- Retuning DXM parameters (e.g. transition probabilities) may be needed for genomic compartments, individual datasets, or samples with extreme methylation changes such as DNMT3A mutation or demethylating agent treatment
- Larger regions require more CpGs with methylation change to be solved as a distinct profile; solving a global profile across an entire chromosome is impractical given assembly gaps and repetitive DNA

## Evidence

- [methods] Compute beta-binomial emission probabilities (Equation 1) for the observed fractional methylation and coverage at each CpG.: [evidence span withheld — anchor status: not_found]
- [methods] Transition matrices govern state switches between CpGs.: [evidence span withheld — anchor status: not_found]
- [readme] Fractional methylation must be in [0,1].: "fractionalMethylation - values should be between 0 (fully unmethylated) and 1 (fully methylated), if the program you use for preprocessing outputs values between 0 and 100 please divide by 100 first."
- [readme] Coverage threshold for input data.: "All data should be filtered such that the coverage is below the maximum expected sequencing coverage, set as the -m flag in dxm_solveMethylation."
- [readme] Relative posterior probabilities are output for all regions.: "<prefix>_allVitProb.txt  - list of all relative posterior probabilities"
