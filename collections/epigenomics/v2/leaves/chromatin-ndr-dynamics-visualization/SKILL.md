---
name: chromatin-ndr-dynamics-visualization
description: Use when comparing timepoint-specific nucleosome dyad positions (e.g.,
  0 min vs 60 min cadmium MNase-seq) across promoter-aligned gene groups to detect
  +1 nucleosome shifts and nucleosome-depleted region remodeling.
when_to_use_negative:
- Input is ATAC-seq rather than MNase-seq and the goal is the cadmium-stress dyad-shift
  analysis — the paper reports markedly better RoboCOP performance with MNase-seq,
  so composite dyad shifts from ATAC-seq would be less reliable
- The question is single-locus nucleosome positioning rather than composite promoter-averaged
  +1 dyad behavior across gene groups
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3176
- http://edamontology.org/topic_0749
tools:
- name: RoboCOP
  role: Runs RoboCOP_MNase independently on 0 min and 60 min MNase-seq datasets to
    compute timepoint-specific nucleosome occupancy/dyad probability profiles
  repo: https://github.com/HarteminkLab/RoboCOP
- name: dynacop_figures.ipynb
  role: Adapted to compute composite +1 dyad probability distributions per gene group
    and generate the Figure 6A-C shift plots
  repo: https://github.com/HarteminkLab/RoboCOP/blob/master/dynacop_figures.ipynb
- name: example_dynacop.ipynb
  role: Vignette describing how to run DynaCOP in jupyter notebook format
  repo: https://github.com/HarteminkLab/RoboCOP/blob/master/analysis/example_dynacop.ipynb
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkab553
    title: 'RoboCOP: jointly computing chromatin occupancy profiles for numerous factors
      from chromatin accessibility data'
  source_content_hash: sha256:0780e4ed6adfb3b81904f7fcccb63d9b95f783cba91f7f567052fd451253d600
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HarteminkLab/RoboCOP
  tools:
  - RoboCOP
  - dynacop_figures.ipynb
  - example_dynacop.ipynb
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3176
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# chromatin-ndr-dynamics-visualization

## Summary

Visualizes cadmium-stress chromatin dynamics by computing composite +1 nucleosome dyad probability distributions across promoters for gene groups defined by RNA-seq response, using RoboCOP_MNase timepoint-specific occupancy profiles and DynaCOP figure notebooks to reveal NDR expansion or contraction.

## When to use

Use when comparing timepoint-specific nucleosome dyad positions (e.g., 0 min vs 60 min cadmium MNase-seq) across promoter-aligned gene groups to detect +1 nucleosome shifts and nucleosome-depleted region remodeling.

## When NOT to use

- Input is ATAC-seq rather than MNase-seq and the goal is the cadmium-stress dyad-shift analysis — the paper reports markedly better RoboCOP performance with MNase-seq, so composite dyad shifts from ATAC-seq would be less reliable
- The question is single-locus nucleosome positioning rather than composite promoter-averaged +1 dyad behavior across gene groups

## Inputs

- MNase-seq data for 0 min and 60 min cadmium treatment of yeast (GEO accession GSE153609)
- RNA-seq expression data under cadmium stress (GEO accession GSE153609)
- RoboCOP_MNase timepoint-specific nucleosome dyad probability profiles (outputs of running RoboCOP on each timepoint)
- Gene group definitions: upmost (top 100 up-regulated), downmost (top 100 down-regulated), constant (100 unchanged)

## Outputs

- Composite probability distributions of the +1 nucleosome dyad aligned across promoters, per gene group and timepoint
- Figure 6A-C style shift plots comparing 0 min vs 60 min composite dyad profiles per gene group
- Qualitative assessment of NDR expansion (upmost) or contraction (downmost) at promoters

## How to apply

Download paired 0 min and 60 min cadmium MNase-seq and RNA-seq data from GEO (GSE153609). Clone RoboCOP, compile the C shared library with 'bash gccCompile' to produce librobocop.so, and register its absolute path as the cshared entry in config.ini. Run RoboCOP_MNase independently on each timepoint's MNase-seq data to obtain timepoint-specific nucleosome dyad probability profiles. Classify genes into upmost (top 100 most up-regulated), downmost (top 100 most down-regulated), and constant (100 unchanged) groups from RNA-seq expression changes. For each group and timepoint, align promoters and compute composite +1 nucleosome dyad probability distributions, adapting dynacop_figures.ipynb to generate the Figure 6A-C shift plots. Judge correctness by confirming composites reproduce the reported pattern: +1 dyad probability shifts away from the NDR (expansion) for upmost genes and toward the NDR (contraction) for downmost genes between 0 min and 60 min.

## Related tools

- **RoboCOP** (Runs RoboCOP_MNase independently on 0 min and 60 min MNase-seq datasets to compute timepoint-specific nucleosome occupancy/dyad probability profiles) — https://github.com/HarteminkLab/RoboCOP
- **dynacop_figures.ipynb** (Adapted to compute composite +1 dyad probability distributions per gene group and generate the Figure 6A-C shift plots) — https://github.com/HarteminkLab/RoboCOP/blob/master/dynacop_figures.ipynb
- **example_dynacop.ipynb** (Vignette describing how to run DynaCOP in jupyter notebook format) — https://github.com/HarteminkLab/RoboCOP/blob/master/analysis/example_dynacop.ipynb

## Examples

```
git clone https://github.com/HarteminkLab/RoboCOP.git; cd RoboCOP/robocop; bash gccCompile; ls "`pwd`/librobocop.so"
```

## Evaluation signals

- Composite +1 dyad probability for the upmost 100 genes shifts downstream (away from the NDR) between 0 min and 60 min, expanding the nucleosome-depleted region
- Composite +1 dyad probability for the downmost 100 genes shifts toward the NDR (contraction) between timepoints
- Constant (unchanged) gene group shows little to no composite +1 dyad shift between 0 min and 60 min
- librobocop.so builds successfully and its absolute path is registered as the cshared entry in config.ini before RoboCOP runs
- Composite profiles are reproducible when regenerating figures with dynacop_figures.ipynb on the same RoboCOP outputs

## Limitations

- RoboCOP (and this visualization) currently uses only MNase-seq or ATAC-seq as accessibility input; MNase-seq gives markedly better performance due to genome-wide fragment distribution and MNase exonuclease activity
- No tool version numbers are stated in the article for RoboCOP or dependencies; environment is pinned via robocop-spec-file.txt conda spec
- Accuracy of the occupancy profiles underlying the composites improves with deeper sequencing; sequencing depth requirements are not quantified

## Evidence
