---
name: differential-gene-expression-stratification
description: Use when RNA-seq expression changes across an environmental perturbation
  (e.g., 0 min vs 60 min cadmium in yeast, GSE153609) must be binned into ranked gene
  groups to subset downstream chromatin analyses such as +1 nucleosome dyad composites.
when_to_use_negative:
- The analysis goal is a genome-wide continuous ranking or statistical differential-expression
  table rather than discrete top-100 gene groups for downstream stratified compositing
- Only one experimental condition/timepoint is available, so no expression change
  can be computed to rank genes into upmost/downmost/constant groups
- Genes are already stratified by a prior differential-expression step and only chromatin
  compositing remains
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: RoboCOP
  role: RoboCOP_MNase consumes the MNase-seq data for 0 min and 60 min cadmium treatment
    to compute timepoint-specific nucleosome dyad probability profiles; the stratified
    gene groups (upmost, downmost, constant) are used to subset those profiles into
    composite +1 dyad shift plots, adapting dynacop_figures.ipynb
  repo: https://github.com/HarteminkLab/RoboCOP
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkab553
    title: 'RoboCOP: jointly computing chromatin occupancy profiles for numerous factors
      from chromatin accessibility data'
  source_content_hash: sha256:79c5851fb8fbe1ac3a64185d91b849ab3fcd834bac06b5bb9dc7e90cb013e610
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HarteminkLab/RoboCOP
  tools:
  - RoboCOP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# differential-gene-expression-stratification

## Summary

Stratifies yeast genes into discrete transcriptional-response groups (upmost: top 100 most up-regulated, downmost: top 100 most down-regulated, constant: 100 unchanged) from RNA-seq expression changes under cadmium stress, so that chromatin dynamics can be composited per group from RoboCOP_MNase nucleosome occupancy profiles.

## When to use

Use when RNA-seq expression changes across an environmental perturbation (e.g., 0 min vs 60 min cadmium in yeast, GSE153609) must be binned into ranked gene groups to subset downstream chromatin analyses such as +1 nucleosome dyad composites.

## When NOT to use

- The analysis goal is a genome-wide continuous ranking or statistical differential-expression table rather than discrete top-100 gene groups for downstream stratified compositing
- Only one experimental condition/timepoint is available, so no expression change can be computed to rank genes into upmost/downmost/constant groups
- Genes are already stratified by a prior differential-expression step and only chromatin compositing remains

## Inputs

- RNA-seq expression data under cadmium stress (0 min and 60 min, GEO accession GSE153609)
- Per-gene expression change values (e.g., fold change / log2 ratio 60 min vs 0 min) for Saccharomyces cerevisiae genes
- RoboCOP_MNase timepoint-specific nucleosome dyad probability profiles (computed from 0 min and 60 min MNase-seq) for compositing per group

## Outputs

- Three gene-group lists: upmost (top 100 most up-regulated), downmost (top 100 most down-regulated), and constant (100 unchanged) genes
- Composite probability distributions of the +1 nucleosome dyad per gene group and timepoint (Figure 6A-C shift plots)

## How to apply

Download the RNA-seq data for yeast from GEO under accession GSE153609 (0 min and 60 min cadmium treatment). Compute per-gene expression change between the two timepoints and rank genes by log change. Define the upmost group as the top 100 most up-regulated genes, the downmost group as the top 100 most down-regulated genes, and the constant group as 100 genes with unchanged expression. These fixed top-100 groupings are the paper's own design choice (no adjusted-p-value threshold is stated), so keep them explicit when reporting. Feed each group, together with timepoint-specific RoboCOP_MNase dyad probability profiles (run independently on 0 min and 60 min MNase-seq), into composite +1 nucleosome dyad distributions aligned across promoters to generate Figure 6A-C style shift plots via dynacop_figures.ipynb.

## Related tools

- **RoboCOP** (RoboCOP_MNase consumes the MNase-seq data for 0 min and 60 min cadmium treatment to compute timepoint-specific nucleosome dyad probability profiles; the stratified gene groups (upmost, downmost, constant) are used to subset those profiles into composite +1 dyad shift plots, adapting dynacop_figures.ipynb) — https://github.com/HarteminkLab/RoboCOP

## Evaluation signals

- Each output group contains exactly 100 genes (upmost, downmost, constant) as defined by the paper's workflow
- The constant group's composite +1 dyad probability shows no appreciable shift between 0 min and 60 min, while the upmost group shifts downstream (expanding the NDR) and the downmost group shifts toward the NDR (contraction)
- Resulting composites reproduce the reported Figure 6A-C pattern when re-running with dynacop_figures.ipynb
- RoboCOP_MNase must be run independently on the 0 min and 60 min MNase-seq datasets before compositing; a single pooled run invalidates the comparison
- Ranking of expression change is computed solely from the RNA-seq data at GSE153609; verifying group membership against the expression table reproduces the same top-100 gene sets

## Limitations

- The top-100 group sizes are a fixed design choice from the task definition; no statistical significance cutoff for differential expression is specified in the article context
- Stratification quality depends on downstream RoboCOP_MNase profiles; the article notes markedly better performance with MNase-seq than ATAC-seq, and deeper sequencing improves occupancy profile accuracy
- Running RoboCOP requires compiling librobocop.so via 'bash gccCompile' and registering its absolute path as the cshared entry in config.ini; build or path errors silently block profile computation

## Evidence

- [other] Classify genes into upmost (top 100 most up-regulated), downmost (top 100 most down-regulated), and constant (100 unchanged) groups using the RNA-seq expression changes under cadmium stress.: [evidence span withheld — anchor status: not_found]
- [other] the RoboCOP_MNase composite +1 nucleosome dyad probability for the 100 most up-regulated genes shifts downstream: [evidence span withheld — anchor status: not_found]
