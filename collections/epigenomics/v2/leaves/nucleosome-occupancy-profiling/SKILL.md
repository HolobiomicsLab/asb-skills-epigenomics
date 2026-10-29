---
name: nucleosome-occupancy-profiling
description: Use when paired-end MNase-seq (or ATAC-seq) data are available for two
  or more conditions and the goal is to quantify nucleosome positioning dynamics,
  such as +1 dyad shifts at promoter NDRs under stress.
when_to_use_negative:
- Input is single-end or low-depth accessibility data where fragment-size information
  is insufficient — the article notes accuracy improves with deeper sequencing
- The goal is TF binding site prediction rather than nucleosome positioning dynamics;
  use the full RoboCOP TF-occupancy workflow instead
- No matched RNA-seq data exist to define transcriptional-response gene groups, so
  upmost/downmost/constant composites cannot be constructed
edam_operation: http://edamontology.org/operation_0436
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0749
tools:
- name: RoboCOP
  role: Multivariate state space model run as RoboCOP_MNase to compute timepoint-specific
    nucleosome dyad probability profiles from MNase-seq plus sequence
  repo: https://github.com/HarteminkLab/RoboCOP
- name: DynaCOP
  role: Dynamic comparison workflow whose figures are generated via dynacop_figures.ipynb
    to produce the composite +1 dyad shift plots
  repo: https://github.com/HarteminkLab/RoboCOP/blob/master/analysis/example_dynacop.ipynb
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkab553
    title: 'RoboCOP: jointly computing chromatin occupancy profiles for numerous factors
      from chromatin accessibility data'
  source_content_hash: sha256:bad668fbf20ccf17675d4d6d8933a82a47ef135123f84a0657ada54436604994
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HarteminkLab/RoboCOP
  tools:
  - DynaCOP
  - RoboCOP
  edam_operation: http://edamontology.org/operation_0436
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# nucleosome-occupancy-profiling

## Summary

Compute genome-wide probabilistic nucleosome occupancy profiles from MNase-seq data using RoboCOP, a multivariate state space model integrating nucleotide sequence with chromatin accessibility, and compare profiles across conditions (e.g., 0 min vs 60 min cadmium stress) to detect +1 nucleosome dyad shifts at promoters.

## When to use

Use when paired-end MNase-seq (or ATAC-seq) data are available for two or more conditions and the goal is to quantify nucleosome positioning dynamics, such as +1 dyad shifts at promoter NDRs under stress.

## When NOT to use

- Input is single-end or low-depth accessibility data where fragment-size information is insufficient — the article notes accuracy improves with deeper sequencing
- The goal is TF binding site prediction rather than nucleosome positioning dynamics; use the full RoboCOP TF-occupancy workflow instead
- No matched RNA-seq data exist to define transcriptional-response gene groups, so upmost/downmost/constant composites cannot be constructed

## Inputs

- Paired-end MNase-seq data for yeast under 0 min and 60 min cadmium treatment (GEO accession GSE153609)
- RNA-seq expression data for the same timepoints (GEO accession GSE153609) for gene-response classification
- Yeast genome nucleotide sequence (RoboCOP integrates sequence with accessibility data)
- config.ini with the cshared path to librobocop.so (patterned on config_example.ini)

## Outputs

- Timepoint-specific genome-wide nucleosome dyad probability profiles from RoboCOP_MNase (0 min and 60 min)
- Composite +1 nucleosome dyad probability distributions aligned across promoters for upmost, downmost, and constant gene groups
- Figure 6A-C shift plots showing dyad probability shifts (NDR expansion for upmost, contraction for downmost genes) between 0 min and 60 min

## How to apply

Download the MNase-seq (0 min and 60 min cadmium treatment) and RNA-seq data from GEO under accession GSE153609. Clone github.com/HarteminkLab/RoboCOP, create the conda environment from robocop-spec-file.txt, then in the robocop directory run 'bash gccCompile' to generate librobocop.so and register its absolute path as the cshared entry in config.ini (e.g., cshared=/home/myhome/RoboCOP/pkg/robocop/librobocop.so). Run RoboCOP_MNase independently on the 0 min and 60 min MNase-seq datasets to obtain timepoint-specific nucleosome dyad probability profiles; MNase-seq yields markedly better performance than ATAC-seq because fragment distributions reflect DBF sizes. Using the RNA-seq expression changes, classify genes into upmost (top 100 most up-regulated), downmost (top 100 most down-regulated), and constant (100 unchanged) groups, then compute composite +1 nucleosome dyad probability distributions aligned across promoters for each group and timepoint, adapting dynacop_figures.ipynb to generate the Figure 6A-C shift plots.

## Related tools

- **RoboCOP** (Multivariate state space model run as RoboCOP_MNase to compute timepoint-specific nucleosome dyad probability profiles from MNase-seq plus sequence) — https://github.com/HarteminkLab/RoboCOP
- **DynaCOP** (Dynamic comparison workflow whose figures are generated via dynacop_figures.ipynb to produce the composite +1 dyad shift plots) — https://github.com/HarteminkLab/RoboCOP/blob/master/analysis/example_dynacop.ipynb

## Examples

```
git clone https://github.com/HarteminkLab/RoboCOP.git && cd RoboCOP/robocop && bash gccCompile
```

## Evaluation signals

- librobocop.so exists after 'bash gccCompile' and its absolute path resolves when set as cshared in config.ini
- RoboCOP_MNase completes independently on both the 0 min and 60 min datasets, yielding timepoint-specific dyad probability profiles
- Composite +1 dyad probability for the 100 most up-regulated genes shifts downstream (away from the NDR, expanding it) from 0 min to 60 min
- Composite +1 dyad probability for the 100 most down-regulated genes shifts toward the NDR (contraction) from 0 min to 60 min
- Constant-gene composites show no appreciable dyad shift between timepoints, matching the reported Figure 6A-C pattern

## Limitations

- MNase-seq gives markedly better RoboCOP performance than ATAC-seq; ATAC-based profiles are less reliable
- Accuracy and comprehensiveness of chromatin occupancy profiles improve with deeper sequencing; depth requirements are not quantified
- No version numbers are stated for RoboCOP or comparison tools (NucleoATAC, MACS2, FIMO), so exact reproducibility depends on the repo state

## Evidence
