---
name: position-weight-matrix-emission-modeling
description: Use when building a chromatin occupancy model that scores TF or nucleosome
  states from nucleotide sequence alongside accessibility data, and sequence emissions
  must be probabilistically defined per HMM state.
when_to_use_negative:
- The goal is peak/footprint identification followed by TF-labeling (e.g. MACS2 +
  FIMO); RoboCOP's joint HMM is an alternative to that workflow, not a drop-in for
  it
- Only accessibility data without nucleotide sequence or motif PWMs are available,
  so sequence emissions cannot be defined
- A single-factor occupancy model suffices (e.g. the earlier COMPETE approach) rather
  than joint occupancy of hundreds of DBFs
edam_operation: http://edamontology.org/operation_0436
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0749
tools:
- name: RoboCOP
  role: Multivariate HMM whose sequence emission component is implemented as PWM and
    dinucleotide emission models over genomic states; source code and gccCompile build
    script live in the repository
  repo: https://github.com/HarteminkLab/RoboCOP
- name: COMPETE
  role: Prior tool from the same lab that computed a probabilistic landscape of DBF
    occupancy from sequence; RoboCOP extends this by adding accessibility emissions
- name: FIMO
  role: Alternative TF-labeling approach used as a comparison baseline against RoboCOP's
    joint HMM predictions
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkab553
    title: 'RoboCOP: jointly computing chromatin occupancy profiles for numerous factors
      from chromatin accessibility data'
  source_content_hash: sha256:eb4fee047977aa178756f21929a33b8a97c33ca382e8e4142cd2688d56e9f97e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HarteminkLab/RoboCOP
  tools:
  - COMPETE
  - FIMO
  - RoboCOP
  edam_operation: http://edamontology.org/operation_0436
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# Position Weight Matrix Emission Modeling

## Summary

Implements the sequence-based emission component of RoboCOP's multivariate HMM, where DNA-binding factor and nucleosome states emit nucleotide observations via position weight matrix (PWM) and dinucleotide emission models over genomic states.

## When to use

Use when building a chromatin occupancy model that scores TF or nucleosome states from nucleotide sequence alongside accessibility data, and sequence emissions must be probabilistically defined per HMM state.

## When NOT to use

- The goal is peak/footprint identification followed by TF-labeling (e.g. MACS2 + FIMO); RoboCOP's joint HMM is an alternative to that workflow, not a drop-in for it
- Only accessibility data without nucleotide sequence or motif PWMs are available, so sequence emissions cannot be defined
- A single-factor occupancy model suffices (e.g. the earlier COMPETE approach) rather than joint occupancy of hundreds of DBFs

## Inputs

- Nucleotide sequence of the genome (yeast genome for RoboCOP applications)
- TF motif PWMs for the DNA-binding factors being modeled
- Paired-end MNase-seq and/or ATAC-seq fragment data for accessibility emissions
- config.ini with cshared path to librobocop.so

## Outputs

- Genome-wide probabilistic occupancy landscape of nucleosomes and 150 TFs at single-nucleotide resolution
- Compiled shared library librobocop.so implementing the HMM model core
- Learned emission and transition parameters from Baum-Welch EM training

## How to apply

Clone the HarteminkLab/RoboCOP repository and study the model specification (emission and transition definitions) alongside the example_robocop.ipynb vignette and config_example.ini. Implement the sequence emission component as position weight matrix (PWM) and dinucleotide emission models over genomic states, so each genomic position emits an observation conditioned on the hidden DBF/nucleosome state. Pair these sequence emissions with a chromatin accessibility emission component implemented as negative-binomial distributions over fragment midpoint counts from MNase-seq or ATAC-seq data. Initialize transition probabilities with a Boltzmann distribution, then run Baum-Welch EM updates with the modification that caps TF transition probabilities at two standard deviations above the mean of their initial values. Finally, run 'bash gccCompile' in the robocop directory to build the model core into librobocop.so and register its absolute path as the cshared entry in config.ini.

## Related tools

- **RoboCOP** (Multivariate HMM whose sequence emission component is implemented as PWM and dinucleotide emission models over genomic states; source code and gccCompile build script live in the repository) — https://github.com/HarteminkLab/RoboCOP
- **COMPETE** (Prior tool from the same lab that computed a probabilistic landscape of DBF occupancy from sequence; RoboCOP extends this by adding accessibility emissions)
- **FIMO** (Alternative TF-labeling approach used as a comparison baseline against RoboCOP's joint HMM predictions)

## Examples

```
cd RoboCOP/robocop && bash gccCompile && ls `pwd`/librobocop.so
```

## Evaluation signals

- librobocop.so exists at the absolute path printed by `ls \`pwd\`/librobocop.so` after running bash gccCompile
- The cshared entry in config.ini resolves to the exact absolute path of librobocop.so, matching the documented build-and-configure procedure
- TF transition probabilities after Baum-Welch updates remain capped at two standard deviations above the mean of the initial (Boltzmann-initialized) transition probabilities
- Model output is a genome-wide probabilistic occupancy profile at single-nucleotide resolution covering nucleosomes and 150 TFs for the yeast genome
- Nucleosome positioning predictions agree with chemical cleavage method annotations, and TF binding site predictions agree with ChIP-chip, ChIP-exo, and ORGANIC annotations better than COMPETE and MACS2+FIMO baselines

## Limitations

- Markedly better performance is observed with MNase-seq than ATAC-seq, because the genome-wide fragment distribution and MNase exonuclease activity reflect DBF sizes
- Prior information about DBF transition probabilities is not incorporated; the authors defer this to future work
- Accuracy and comprehensiveness of the chromatin occupancy profiles improve only with deeper sequencing
- No version numbers are stated for the tools (RoboCOP, COMPETE, NucleoATAC, MACS2, FIMO) involved

## Evidence

- [methods] Sequence emissions as PWM and dinucleotide models: [evidence span withheld — anchor status: not_found]
