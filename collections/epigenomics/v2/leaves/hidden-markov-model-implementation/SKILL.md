---
name: hidden-markov-model-implementation
description: Use when you need to build or reimplement an HMM/state-space model that
  jointly scores nucleosome and transcription factor occupancy genome-wide from paired-end
  MNase-seq and/or ATAC-seq data plus genomic sequence.
when_to_use_negative:
- When the goal is peak calling or footprint identification followed by TF labeling
  (e.g. MACS2 + FIMO) rather than a joint probabilistic occupancy model — RoboCOP
  is an alternative to that approach
- When accessibility data are not paired-end MNase-seq/ATAC-seq fragment data, since
  the negative-binomial emission model over fragment midpoints is tailored to those
  assays
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0749
tools:
- name: RoboCOP
  role: Multivariate HMM that integrates nucleotide sequence and chromatin accessibility
    data to compute the occupancy profile; source code is cloned and compiled into
    librobocop.so
  repo: https://github.com/HarteminkLab/RoboCOP
- name: COMPETE
  role: Earlier method from the same lab that computed a probabilistic landscape of
    DBF occupancy along the genome; RoboCOP improves TF binding site predictions over
    it
- name: NucleoATAC
  role: Nucleosome positioning baseline that RoboCOP-ATAC outperforms on the same
    ATAC-seq data
- name: MACS2
  role: Peak/footprint identification used in the comparison baseline, followed by
    TF-labeling
- name: FIMO
  role: TF-labeling tool used in the comparison baseline after MACS2 peak calling
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkab553
    title: 'RoboCOP: jointly computing chromatin occupancy profiles for numerous factors
      from chromatin accessibility data'
  source_content_hash: sha256:bbcecbc64bb9130c0d1aadcd6ed3f48c6543bfc97736778d5969f7432c452f8a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HarteminkLab/RoboCOP
  tools:
  - COMPETE
  - FIMO
  - MACS2
  - NucleoATAC
  - RoboCOP
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# hidden-markov-model-implementation

## Summary

Implement the multivariate hidden Markov model underlying RoboCOP, which integrates nucleotide sequence (PWM and dinucleotide emission models) with chromatin accessibility data (negative-binomial fragment midpoint counts) to jointly compute genome-wide probabilistic occupancy profiles of nucleosomes and ~150 TFs.

## When to use

Use when you need to build or reimplement an HMM/state-space model that jointly scores nucleosome and transcription factor occupancy genome-wide from paired-end MNase-seq and/or ATAC-seq data plus genomic sequence.

## When NOT to use

- When the goal is peak calling or footprint identification followed by TF labeling (e.g. MACS2 + FIMO) rather than a joint probabilistic occupancy model — RoboCOP is an alternative to that approach
- When accessibility data are not paired-end MNase-seq/ATAC-seq fragment data, since the negative-binomial emission model over fragment midpoints is tailored to those assays

## Inputs

- Paired-end MNase-seq and/or ATAC-seq chromatin accessibility data (e.g. GEO GSE153609, GSE66386)
- Nucleotide sequence of the genome of interest (e.g. Saccharomyces cerevisiae)
- Position weight matrices for the DNA-binding factors being modeled
- config.ini with the cshared entry pointing to librobocop.so

## Outputs

- Genome-wide probabilistic occupancy landscape of nucleosomes and ~150 TFs at single-nucleotide resolution
- Learned transition and emission parameters from Baum-Welch training
- Compiled shared library librobocop.so

## How to apply

Clone the HarteminkLab/RoboCOP repository and study the model specification alongside the example_robocop.ipynb vignette and config_example.ini. Implement the sequence emission component as position weight matrix (PWM) and dinucleotide emission models over genomic states, and the accessibility emission component as negative-binomial distributions over fragment midpoint counts. Initialize transition probabilities with a Boltzmann distribution, then run Baum-Welch EM updates with the modification that caps TF transition probabilities at two standard deviations above the mean of their initial values. Build the model core with 'bash gccCompile' in the robocop directory to produce librobocop.so, and register the absolute path printed by 'ls `pwd`/librobocop.so' as the cshared entry in config.ini.

## Related tools

- **RoboCOP** (Multivariate HMM that integrates nucleotide sequence and chromatin accessibility data to compute the occupancy profile; source code is cloned and compiled into librobocop.so) — https://github.com/HarteminkLab/RoboCOP
- **COMPETE** (Earlier method from the same lab that computed a probabilistic landscape of DBF occupancy along the genome; RoboCOP improves TF binding site predictions over it)
- **NucleoATAC** (Nucleosome positioning baseline that RoboCOP-ATAC outperforms on the same ATAC-seq data)
- **MACS2** (Peak/footprint identification used in the comparison baseline, followed by TF-labeling)
- **FIMO** (TF-labeling tool used in the comparison baseline after MACS2 peak calling)

## Examples

```
cd RoboCOP/robocop && bash gccCompile && ls `pwd`/librobocop.so
```

## Evaluation signals

- Verify librobocop.so exists at the absolute path printed by 'ls `pwd`/librobocop.so' after running bash gccCompile
- Verify the cshared path in config.ini resolves to that exact librobocop.so file (e.g. cshared=/home/myhome/RoboCOP/pkg/robocop/librobocop.so)
- Nucleosome positioning predictions validated against high-precision chemical cleavage method annotations
- TF binding site predictions validated against ChIP-chip, ChIP-exo, and ORGANIC annotations
- Performance should exceed MACS2 + FIMO peak/footprint identification and, for nucleosomes, NucleoATAC on the same data

## Limitations

- MNase-seq gives markedly better RoboCOP performance than ATAC-seq due to fragment distributions and MNase exonuclease activity reflecting DBF sizes
- No prior information about DBF transition probabilities is incorporated; the authors deferred this to future work
- Accuracy and comprehensiveness of profiles improve only with deeper sequencing; no sequencing depth threshold is quantified
- No version numbers are stated for RoboCOP or comparison tools in the source context

## Evidence
