---
name: baum-welch-em-parameter-estimation
description: Use when fitting a hidden Markov model of DNA-binding factor occupancy
  to paired-end MNase-seq and/or ATAC-seq fragment data plus nucleotide sequence,
  requiring EM learning of transition probabilities with constrained TF transitions.
when_to_use_negative:
- When the goal is peak calling or footprint identification rather than a joint probabilistic
  occupancy landscape — the article contrasts RoboCOP with MACS2 peak/footprint identification
  followed by FIMO TF-labeling
- When a quick TF-labeling scan of motifs is sufficient and no chromatin accessibility
  data (MNase-seq or ATAC-seq) are available to inform emission probabilities
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0749
tools:
- name: RoboCOP
  role: Multivariate HMM whose Baum-Welch training procedure, PWM/dinucleotide sequence
    emissions, negative-binomial accessibility emissions, and capped TF transitions
    this skill implements; compiled via gccCompile into librobocop.so
  repo: https://github.com/HarteminkLab/RoboCOP
- name: COMPETE
  role: Earlier sequence-only probabilistic occupancy method that RoboCOP extends
    by adding accessibility-data emissions and EM-learned transitions
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkab553
    title: 'RoboCOP: jointly computing chromatin occupancy profiles for numerous factors
      from chromatin accessibility data'
  source_content_hash: sha256:ac09d6113331e3c7aa740e152515c9d6c9cd8da80a6e32d459743925aea967bd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HarteminkLab/RoboCOP
  tools:
  - COMPETE
  - RoboCOP
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# baum-welch-em-parameter-estimation

## Summary

Trains RoboCOP's multivariate hidden Markov model by Baum-Welch EM, initializing transition probabilities from a Boltzmann distribution and capping TF transition probabilities at two standard deviations above the mean of their initial values, so that sequence (PWM/dinucleotide) and accessibility (negative-binomial) emissions jointly yield genome-wide nucleosome and TF occupancy probabilities.

## When to use

Use when fitting a hidden Markov model of DNA-binding factor occupancy to paired-end MNase-seq and/or ATAC-seq fragment data plus nucleotide sequence, requiring EM learning of transition probabilities with constrained TF transitions.

## When NOT to use

- When the goal is peak calling or footprint identification rather than a joint probabilistic occupancy landscape — the article contrasts RoboCOP with MACS2 peak/footprint identification followed by FIMO TF-labeling
- When a quick TF-labeling scan of motifs is sufficient and no chromatin accessibility data (MNase-seq or ATAC-seq) are available to inform emission probabilities

## Inputs

- Nucleotide sequence of the genome (e.g. S. cerevisiae)
- Paired-end MNase-seq and/or ATAC-seq fragment data (fragment midpoint counts)
- Set of DNA-binding factors (DBFs) with sequence motifs for PWM/dinucleotide emission models
- config.ini with cshared path to librobocop.so

## Outputs

- librobocop.so shared library implementing the HMM core
- Genome-wide probabilistic occupancy scores of nucleosomes and ~150 TFs at single-nucleotide resolution
- Learned HMM transition and emission parameters

## How to apply

Clone HarteminkLab/RoboCOP and study the model specification in example_robocop.ipynb and config_example.ini. Implement the sequence emission component as position weight matrix (PWM) and dinucleotide emission models over genomic states, and the accessibility emission component as negative-binomial distributions over fragment midpoint counts. Initialize transition probabilities with a Boltzmann distribution, then run Baum-Welch EM updates with the modification that TF transition probabilities are capped: the threshold δ is chosen to be two standard deviations above the mean of the initial transition probabilities of all TFs. Build the model core by running `bash gccCompile` in the robocop directory to generate librobocop.so, then register its absolute path as the cshared entry in config.ini (e.g. cshared=/home/myhome/RoboCOP/pkg/robocop/librobocop.so) before running inference.

## Related tools

- **RoboCOP** (Multivariate HMM whose Baum-Welch training procedure, PWM/dinucleotide sequence emissions, negative-binomial accessibility emissions, and capped TF transitions this skill implements; compiled via gccCompile into librobocop.so) — https://github.com/HarteminkLab/RoboCOP
- **COMPETE** (Earlier sequence-only probabilistic occupancy method that RoboCOP extends by adding accessibility-data emissions and EM-learned transitions)

## Examples

```
cd RoboCOP/robocop && bash gccCompile && ls `pwd`/librobocop.so
```

## Evaluation signals

- librobocop.so exists at the absolute path printed by `ls \`pwd\`/librobocop.so` after running bash gccCompile
- The cshared entry in config.ini resolves to the same librobocop.so absolute path
- TF transition probabilities after EM remain capped at the threshold δ = mean(initial TF transition probabilities) + 2 standard deviations
- RoboCOP occupancy predictions outperform the MACS2-plus-FIMO alternative and, for TF binding sites, improve over COMPETE
- Nucleosome positioning predictions agree with chemical-cleavage-method annotations; TF binding sites agree with ChIP-chip, ChIP-exo, and ORGANIC annotations

## Limitations

- No version numbers are stated for RoboCOP or comparison tools (COMPETE, NucleoATAC, MACS2, FIMO)
- Performance is markedly better with MNase-seq than ATAC-seq, because the genome-wide fragment distribution and MNase exonuclease activity better reflect DBF sizes
- Prior information about DBF transition probabilities is not incorporated — deferred to future work
- Accuracy and comprehensiveness of occupancy profiles improve only with deeper sequencing

## Evidence

- [methods] Boltzmann initialization and TF transition cap: [evidence span withheld — anchor status: not_found]
- [methods] Transition cap finding: [evidence span withheld — anchor status: not_found]
- [intro] Model structure: "uses a multivariate hidden Markov model (HMM) (23) to compute a probabilistic occupancy landscape of hundreds of DBFs genome-wide at single-nucleotide resolution"
- [readme] Build step: "bash gccCompile # generates shared library librobocop.so"
- [readme] Config path registration: "cshared=/home/myhome/RoboCOP/pkg/robocop/librobocop.so"
- [discussion] MNase vs ATAC performance: "we observe markedly better performance when using MNase-seq than when using ATAC-seq, for two primary reasons"
