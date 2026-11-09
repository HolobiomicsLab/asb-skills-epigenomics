---
name: composite-dyad-probability-plotting
description: Use when comparing nucleosome positioning around +1 dyads between two
  conditions (e.g., 0 min vs 60 min cadmium) for gene groups defined by RNA-seq response,
  using RoboCOP_MNase dyad probabilities.
when_to_use_negative:
- Only a single condition/timepoint is available, so no between-timepoint dyad shift
  can be computed
- The analysis goal is TF occupancy or DBF footprinting rather than nucleosome dyad
  positioning at promoters
- Input is ATAC-seq rather than MNase-seq when the goal is replicating this cadmium-stress
  nucleosome result (MNase-seq gave markedly better RoboCOP performance in the paper)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0749
tools:
- name: RoboCOP
  role: Computes timepoint-specific nucleosome dyad probability profiles from MNase-seq
    via a multivariate HMM; composites are then computed from these profiles
  repo: https://github.com/HarteminkLab/RoboCOP
- name: DynaCOP
  role: Dynamics workflow (example_dynacop.ipynb and dynacop_figures.ipynb) used to
    generate the cadmium-stress composite shift figures
  repo: https://github.com/HarteminkLab/RoboCOP/blob/master/dynacop_figures.ipynb
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkab553
    title: 'RoboCOP: jointly computing chromatin occupancy profiles for numerous factors
      from chromatin accessibility data'
  source_content_hash: sha256:41f81e4b6d5abfd24fa2082bcafba55d07e697db44cbbb5914ee92345851de71
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HarteminkLab/RoboCOP
  tools:
  - DynaCOP
  - RoboCOP
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# composite-dyad-probability-plotting

## Summary

Generates composite +1 nucleosome dyad probability distributions aligned across promoter sets (e.g., top 100 up-, down-regulated, or constant genes under cadmium stress) from RoboCOP_MNase timepoint-specific occupancy profiles, to reveal NDR expansion or contraction between conditions.

## When to use

Use when comparing nucleosome positioning around +1 dyads between two conditions (e.g., 0 min vs 60 min cadmium) for gene groups defined by RNA-seq response, using RoboCOP_MNase dyad probabilities.

## When NOT to use

- Only a single condition/timepoint is available, so no between-timepoint dyad shift can be computed
- The analysis goal is TF occupancy or DBF footprinting rather than nucleosome dyad positioning at promoters
- Input is ATAC-seq rather than MNase-seq when the goal is replicating this cadmium-stress nucleosome result (MNase-seq gave markedly better RoboCOP performance in the paper)

## Inputs

- MNase-seq data for yeast at two timepoints (0 min and 60 min cadmium treatment) from GEO accession GSE153609
- RNA-seq expression data under cadmium stress from GEO accession GSE153609 (for gene group classification)
- RoboCOP_MNase timepoint-specific nucleosome dyad probability profiles
- dynacop_figures.ipynb notebook (adapted for Figure 6A-C shift plots)

## Outputs

- Composite +1 nucleosome dyad probability distributions per gene group and timepoint
- Figure 6A-C style +1 dyad shift plots comparing 0 min vs 60 min cadmium for upmost, downmost, and constant gene groups
- Direction of shift (NDR expansion for upmost genes; NDR contraction for downmost genes)

## How to apply

Run RoboCOP_MNase independently on the two MNase-seq timepoints (0 min and 60 min cadmium, GEO GSE153609) to obtain timepoint-specific nucleosome dyad probability profiles. Classify genes into upmost (top 100 most up-regulated), downmost (top 100 most down-regulated), and constant (100 unchanged) groups using the RNA-seq expression changes. For each gene group and timepoint, compute the composite probability distribution of the +1 nucleosome dyad aligned across promoters, adapting the dynacop_figures.ipynb notebook (the DynaCOP vignette example_dynacop.ipynb describes running DynaCOP). Compare composites between timepoints: upmost genes should show dyad probability shifting downstream (away from the NDR, expanding it), while downmost genes shift toward the NDR (contraction). The RoboCOP repo must be compiled first with 'bash gccCompile' to produce librobocop.so, whose absolute path is registered as the cshared entry in config.ini.

## Related tools

- **RoboCOP** (Computes timepoint-specific nucleosome dyad probability profiles from MNase-seq via a multivariate HMM; composites are then computed from these profiles) — https://github.com/HarteminkLab/RoboCOP
- **DynaCOP** (Dynamics workflow (example_dynacop.ipynb and dynacop_figures.ipynb) used to generate the cadmium-stress composite shift figures) — https://github.com/HarteminkLab/RoboCOP/blob/master/dynacop_figures.ipynb

## Examples

```
git clone https://github.com/HarteminkLab/RoboCOP.git && cd RoboCOP/robocop && bash gccCompile
```

## Evaluation signals

- Composite dyad probability for upmost (top 100 up-regulated) genes shifts downstream / away from the NDR between 0 min and 60 min, expanding the nucleosome-depleted region
- Composite dyad probability for downmost (top 100 down-regulated) genes shifts toward the NDR (contraction) between timepoints
- Constant (100 unchanged) gene group shows little or no dyad shift between timepoints
- librobocop.so compiled successfully and its absolute path resolves as the cshared entry in config.ini
- Composites reproduce the reported Figure 6A-C pattern when regenerated via the adapted dynacop_figures.ipynb

## Limitations

- No tool version numbers are specified for RoboCOP or dependencies; the conda environment must be rebuilt from robocop-spec-file.txt
- Results depend on RNA-seq-based gene classification cutoffs (top 100 groups), so group definition choices affect the composite shapes
- The paper notes that accuracy of chromatin occupancy profiles improves with deeper sequencing; sequencing depth requirements are not quantified

## Evidence

- [other] Upon cadmium treatment, the RoboCOP_MNase composite +1 nucleosome dyad probability for the 100 most up-regulated genes shifts downstream, expanding the nucleosome-depleted region at their promoters.: [evidence span withheld — anchor status: not_found]
