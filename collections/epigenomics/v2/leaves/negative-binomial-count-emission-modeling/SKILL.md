---
name: negative-binomial-count-emission-modeling
description: Use when computing genome-wide chromatin occupancy profiles from paired-end
  MNase-seq or ATAC-seq fragment data together with nucleotide sequence, and per-factor
  accessibility counts must be modeled as emission distributions inside an HMM.
when_to_use_negative:
- The analysis goal is peak/footprint identification followed by TF labeling — the
  paper compares against MACS2 + FIMO, which is a different (lower-performing) paradigm
- Only single-end or unpaired accessibility data are available without fragment midpoint
  information
- No nucleotide sequence or PWM/dinucleotide motif models are available for the factors
  of interest, since RoboCOP integrates sequence and accessibility jointly
edam_operation: http://edamontology.org/operation_2425
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: RoboCOP
  role: Multivariate HMM framework whose accessibility emission component is the negative-binomial
    count model over fragment midpoints
  repo: https://github.com/HarteminkLab/RoboCOP
- name: COMPETE
  role: Predecessor method computing a probabilistic landscape of DBF occupancy from
    sequence alone; RoboCOP extends it with accessibility emissions
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkab553
    title: 'RoboCOP: jointly computing chromatin occupancy profiles for numerous factors
      from chromatin accessibility data'
  source_content_hash: sha256:e54bafb7918254103bdfc3bc47763142afda12f7806d40e5cc683331868f2dc5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HarteminkLab/RoboCOP
  tools:
  - COMPETE
  - RoboCOP
  edam_operation: http://edamontology.org/operation_2425
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# Negative-binomial count emission modeling

## Summary

Model chromatin accessibility fragment counts (MNase-seq or ATAC-seq fragment midpoints) as negative-binomial distributions within RoboCOP's multivariate HMM, so that accessibility and sequence evidence (PWM/dinucleotide emissions) jointly drive genome-wide DBF occupancy calls.

## When to use

Use when computing genome-wide chromatin occupancy profiles from paired-end MNase-seq or ATAC-seq fragment data together with nucleotide sequence, and per-factor accessibility counts must be modeled as emission distributions inside an HMM.

## When NOT to use

- The analysis goal is peak/footprint identification followed by TF labeling — the paper compares against MACS2 + FIMO, which is a different (lower-performing) paradigm
- Only single-end or unpaired accessibility data are available without fragment midpoint information
- No nucleotide sequence or PWM/dinucleotide motif models are available for the factors of interest, since RoboCOP integrates sequence and accessibility jointly

## Inputs

- Paired-end MNase-seq and/or ATAC-seq chromatin accessibility data (e.g. GSE66386 ATAC-seq, GSE153609 MNase-seq)
- Genome nucleotide sequence with TF position weight matrices (PWMs)
- RoboCOP configuration file (config.ini) with cshared path to librobocop.so

## Outputs

- Genome-wide probabilistic occupancy scores of nucleosomes and 150 TFs at single-nucleotide resolution
- Learned HMM parameters (negative-binomial emission and Baum-Welch-updated transition probabilities)

## How to apply

Within RoboCOP's multivariate state space model, define the chromatin accessibility emission component as negative-binomial distributions over fragment midpoint counts for each genomic state (nucleosome or TF), while the sequence emission component uses PWM and dinucleotide models over the same states. Initialize transition probabilities with a Boltzmann distribution, then run Baum-Welch EM updates, capping TF transition probabilities at two standard deviations above the mean of their initial transition probabilities (the AT-rich TF threshold δ). Build the model core with `bash gccCompile` into librobocop.so and register its absolute path as the `cshared` entry in config.ini. Note the paper's observation that MNase-seq yields markedly better performance than ATAC-seq because genome-wide fragment distributions and MNase exonuclease activity better reflect DBF sizes.

## Related tools

- **RoboCOP** (Multivariate HMM framework whose accessibility emission component is the negative-binomial count model over fragment midpoints) — https://github.com/HarteminkLab/RoboCOP
- **COMPETE** (Predecessor method computing a probabilistic landscape of DBF occupancy from sequence alone; RoboCOP extends it with accessibility emissions)

## Examples

```
cd RoboCOP/robocop; bash gccCompile; ls "`pwd`/librobocop.so"
```

## Evaluation signals

- librobocop.so exists at the absolute path printed by `ls \`pwd\`/librobocop.so` after running `bash gccCompile`
- The `cshared` entry in config.ini resolves to the actual librobocop.so file path
- Occupancy predictions outperform MACS2 peak/footprint identification followed by FIMO TF-labeling on validation annotations
- TF transition probabilities after Baum-Welch updates do not exceed two standard deviations above the mean of their initial values
- Nucleosome positioning predictions validated against chemical cleavage method annotations; TF binding sites validated against ChIP-chip, ChIP-exo, and ORGANIC annotations

## Limitations

- Markedly better performance is observed with MNase-seq than with ATAC-seq, so accessibility assay choice affects emission model quality
- No prior information about DBF transition probabilities is incorporated in the current model (deferred to future work)
- Accuracy and comprehensiveness of occupancy profiles improve only with deeper sequencing; current code uses MNase-seq or ATAC-seq only

## Evidence
