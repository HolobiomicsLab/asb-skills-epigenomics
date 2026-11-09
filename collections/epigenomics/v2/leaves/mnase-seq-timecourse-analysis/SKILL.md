---
name: mnase-seq-timecourse-analysis
description: Use when you have paired timepoint MNase-seq (plus matched RNA-seq) data
  and need to detect stress- or condition-induced nucleosome repositioning, such as
  +1 dyad shifts at promoters of transcriptionally responsive genes.
when_to_use_negative:
- Input is ATAC-seq rather than MNase-seq — the article reports markedly better RoboCOP
  performance with MNase-seq, and this skill's cadmium-stress workflow is MNase-based
- The goal is per-factor TF binding quantification with ChIP validation rather than
  timepoint-to-timepoint nucleosome positioning dynamics
- You have only a single condition/timepoint with no matched RNA-seq to define upmost/downmost/constant
  gene groups
edam_operation: http://edamontology.org/operation_0439
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3176
- http://edamontology.org/topic_0203
tools:
- name: RoboCOP
  role: Multivariate state space model run per timepoint on MNase-seq data to compute
    probabilistic nucleosome occupancy and dyad profiles; source code compiled to
    librobocop.so
  repo: https://github.com/HarteminkLab/RoboCOP
- name: DynaCOP
  role: Dynamic analysis workflow (example_dynacop.ipynb, dynacop_figures.ipynb) adapted
    to generate composite +1 dyad shift plots across timepoints and gene groups
  repo: https://github.com/HarteminkLab/RoboCOP/blob/master/analysis/example_dynacop.ipynb
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkab553
    title: 'RoboCOP: jointly computing chromatin occupancy profiles for numerous factors
      from chromatin accessibility data'
  source_content_hash: sha256:ff5527d4ed38c5d3aaa75351aa13120290b0340f3db59424dd78ae2ab0d3644d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HarteminkLab/RoboCOP
  tools:
  - DynaCOP
  - RoboCOP
  edam_operation: http://edamontology.org/operation_0439
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3176
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# mnase-seq-timecourse-analysis

## Summary

Analyzes timecourse MNase-seq data (e.g., 0 min vs 60 min cadmium stress in yeast) by running RoboCOP_MNase per timepoint to compute timepoint-specific nucleosome occupancy profiles, then comparing composite +1 nucleosome dyad probability distributions across promoter-aligned gene groups defined from matched RNA-seq.

## When to use

Use when you have paired timepoint MNase-seq (plus matched RNA-seq) data and need to detect stress- or condition-induced nucleosome repositioning, such as +1 dyad shifts at promoters of transcriptionally responsive genes.

## When NOT to use

- Input is ATAC-seq rather than MNase-seq — the article reports markedly better RoboCOP performance with MNase-seq, and this skill's cadmium-stress workflow is MNase-based
- The goal is per-factor TF binding quantification with ChIP validation rather than timepoint-to-timepoint nucleosome positioning dynamics
- You have only a single condition/timepoint with no matched RNA-seq to define upmost/downmost/constant gene groups

## Inputs

- Paired-end MNase-seq reads for two or more timepoints (e.g., 0 min and 60 min cadmium stress; GEO accession GSE153609)
- Matched RNA-seq expression data for the same timepoints (GSE153609) to define transcriptional response groups
- Yeast gene annotations/promoter coordinates for aligning +1 nucleosome dyads across genes
- RoboCOP configuration file (config.ini) with the cshared entry pointing to librobocop.so

## Outputs

- Timepoint-specific nucleosome occupancy and dyad probability profiles from RoboCOP_MNase (one run per timepoint)
- Gene groups: upmost (top 100 most up-regulated), downmost (top 100 most down-regulated), constant (100 unchanged)
- Composite +1 nucleosome dyad probability distributions per gene group per timepoint (Figure 6A-C style shift plots)
- Qualitative direction of shift: NDR expansion (dyad shifts downstream) for upmost genes, NDR contraction for downmost genes

## How to apply

Download the MNase-seq (0 min and 60 min cadmium treatment) and RNA-seq data from GEO under accession GSE153609. Clone the RoboCOP repository, run 'bash gccCompile' in the robocop directory to generate librobocop.so, and register its absolute path (e.g., cshared=/home/myhome/RoboCOP/pkg/robocop/librobocop.so) in config.ini following config_example.ini; create a conda environment from robocop-spec-file.txt if needed. Run RoboCOP_MNase independently on the 0 min and 60 min MNase-seq datasets to obtain timepoint-specific nucleosome occupancy and dyad probability profiles. Using the RNA-seq expression changes, classify genes into upmost (top 100 most up-regulated), downmost (top 100 most down-regulated), and constant (100 unchanged) groups. For each group and timepoint, compute composite probability distributions of the +1 nucleosome dyad aligned across promoters, adapting dynacop_figures.ipynb (the DynaCOP vignette is example_dynacop.ipynb), and compare 0 min vs 60 min composites for NDR expansion or contraction.

## Related tools

- **RoboCOP** (Multivariate state space model run per timepoint on MNase-seq data to compute probabilistic nucleosome occupancy and dyad profiles; source code compiled to librobocop.so) — https://github.com/HarteminkLab/RoboCOP
- **DynaCOP** (Dynamic analysis workflow (example_dynacop.ipynb, dynacop_figures.ipynb) adapted to generate composite +1 dyad shift plots across timepoints and gene groups) — https://github.com/HarteminkLab/RoboCOP/blob/master/analysis/example_dynacop.ipynb

## Examples

```
git clone https://github.com/HarteminkLab/RoboCOP.git && cd RoboCOP/robocop && bash gccCompile && ls "`pwd`/librobocop.so"
```

## Evaluation signals

- Both timepoints (0 min and 60 min) run successfully through RoboCOP_MNase with the cshared path to librobocop.so correctly set in config.ini
- Composite +1 dyad probability for the 100 most up-regulated genes shifts downstream (away from the NDR, expanding the nucleosome-depleted region) from 0 min to 60 min
- Composite +1 dyad probability for the 100 most down-regulated genes shifts toward the NDR (contraction) between 0 min and 60 min
- Constant-gene (100 unchanged) composite shows minimal dyad shift between timepoints, serving as a negative control
- Gene group sizes match the specification: top 100 up, top 100 down, 100 constant

## Limitations

- MNase-seq sequencing depth matters: accuracy and comprehensiveness of chromatin occupancy profiles improve with deeper sequencing
- No tool version numbers are specified for RoboCOP or comparison tools, so exact reproducibility may require pinning the repository state
- RoboCOP currently uses only MNase-seq or ATAC-seq as the accessibility input; combining or substituting data types changes performance

## Evidence
