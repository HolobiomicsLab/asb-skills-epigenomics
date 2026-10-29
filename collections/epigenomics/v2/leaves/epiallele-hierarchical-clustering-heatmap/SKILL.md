---
name: epiallele-hierarchical-clustering-heatmap
description: Use when you have epialleles annotated by MethCores_Combinator (Module
  2) from bisulfite-sequenced single-molecule CpG configurations and need to visualize,
  cluster, and significance-test epiallele families across samples or cell populations.
when_to_use_negative:
- Input epialleles have not yet passed Module 1/Module 2 significance annotation (chi-square
  P ≤ 10−9); the analysis step assumes pre-annotated epialleles
- Data are single-CpG methylation calls or unphased bulk methylation levels (e.g.,
  from EBA or methclone) rather than single-molecule phased CpG configurations from
  ampliMethProfiler binary epihaplotype files
- The goal is to identify significant mCpG combinations de novo from raw reads rather
  than to cluster, visualize, and index already-annotated epialleles
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3295
tools:
- name: MethCoresProfiler
  role: R-based tool whose Module 3 (MethCores_Analyst) performs the hierarchical
    clustering, heatmap rendering, chi-square comparison to a random control, PCA,
    and index computation
  repo: https://github.com/84AP/MethCoresProfiler/
- name: R
  role: Language in which MethCoresProfiler and all three modules are written; requires
    packages such as ComplexHeatmap, entropy, factoextra, and vegan
  repo: https://github.com/84AP/MethCoresProfiler/
- name: ampliMethProfiler
  role: Upstream tool producing the binary epihaplotype files of single-molecule mCpG
    configurations that feed the MethCoresProfiler pipeline
  repo: https://sourceforge.net/projects/amplimethprofiler/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqaa096
    title: Tracing and tracking epiallele families in complex DNA populations
  source_content_hash: sha256:fb0b0346b94d213691a6dbfede839c3024b0a7d2269ad7fa80e1eeb6ce553829
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/84AP/MethCoresProfiler
  repo_url: https://github.com/84AP/MethCoresProfiler
  tools:
  - MethCoresProfiler
  - PEAR
  - PRINSEQ
  - R
  - ampliMethProfiler
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# epiallele-hierarchical-clustering-heatmap

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates hierarchical clustering heatmaps of epiallele structure and frequency using the MethCoresProfiler MethCores_Analyst (Module 3), testing each epiallele against a random theoretical control and computing MethCores, Clonality, and E (stability/entanglement) indices per sample.

## When to use

Use when you have epialleles annotated by MethCores_Combinator (Module 2) from bisulfite-sequenced single-molecule CpG configurations and need to visualize, cluster, and significance-test epiallele families across samples or cell populations.

## When NOT to use

- Input epialleles have not yet passed Module 1/Module 2 significance annotation (chi-square P ≤ 10−9); the analysis step assumes pre-annotated epialleles
- Data are single-CpG methylation calls or unphased bulk methylation levels (e.g., from EBA or methclone) rather than single-molecule phased CpG configurations from ampliMethProfiler binary epihaplotype files
- The goal is to identify significant mCpG combinations de novo from raw reads rather than to cluster, visualize, and index already-annotated epialleles

## Inputs

- Epiallele tables annotated by MethCores_Combinator (Module 2)
- Tab-delimited list of target regions
- Random control (R) generated automatically by Module 1 with the same number of CpGs and depth
- R environment with dependencies installed via Install_Dependencies.R

## Outputs

- Hierarchical clustering heatmap of epiallele structure and frequency with significant epialleles marked (red = high frequency, blue = low frequency, white = neutral)
- PCA of significant epialleles
- ComposCore / Tab_Epialleles-style tables of significant mCpG combinations and epiallele frequencies
- Per-sample MethCores Index, Clonality Index, and E (stability/entanglement) Index

## How to apply

Run Module 3 (3_MethCores_Analysis.R) on the epialleles annotated by MethCores_Combinator, passing a list of regions and the module's filtering parameters. The module compares each epiallele's frequency by chi-square test to a random control (R) generated automatically by Module 1 with the same number of CpGs and depth, then renders a complex heatmap (red = high frequency, blue = low frequency, white = neutral) marking individually significant epialleles. It also performs PCA of significant epialleles, extracts the most frequent mCpGs shared by significant epialleles on a decreasing percentage scale starting from 0.9 (minimum 2-mCpGs), and falls back to annotating the highest-frequency epiallele as a stable core if none are significant. Outputs three indices per sample: the MethCores Index (core frequency in the population), the Clonality Index (core frequency normalized to average methylation), and the E Index (in-phase CpGs / stability / entanglement, measuring coupling of at least 2 mCpGs). Requires R with the vegan, ComplexHeatmap, entropy, factoextra, and other dependencies listed in the package README.

## Related tools

- **MethCoresProfiler** (R-based tool whose Module 3 (MethCores_Analyst) performs the hierarchical clustering, heatmap rendering, chi-square comparison to a random control, PCA, and index computation) — https://github.com/84AP/MethCoresProfiler/
- **R** (Language in which MethCoresProfiler and all three modules are written; requires packages such as ComplexHeatmap, entropy, factoextra, and vegan) — https://github.com/84AP/MethCoresProfiler/
- **ampliMethProfiler** (Upstream tool producing the binary epihaplotype files of single-molecule mCpG configurations that feed the MethCoresProfiler pipeline) — https://sourceforge.net/projects/amplimethprofiler/

## Examples

```
Rscript Run_MethCoresProfiler_setup.R  # with MethCoresProfiler="/path-to-MethCoresProfiler-master/", Data="testData/", input="out/" set inside
```

## Evaluation signals

- Heatmap renders with the documented color semantics: red = high frequency, blue = low frequency, white = neutral, with individually significant epialleles marked
- Each epiallele's significance is supported by a chi-square comparison against the Module-1 random control with the same number of CpGs and depth
- Per-sample output includes all three indices: MethCores Index, Clonality Index (normalized to average population methylation), and E (entanglement) Index
- Most frequent shared mCpGs are extracted on the decreasing percentage scale starting from 0.9 with a minimum of 2 mCpGs; if no significant epialleles exist, the highest-frequency epiallele is annotated as the stable core
- In the published validation, astrocytes and MOEs share a common precursor core distinct from the neuron/whole-cerebellum epiallele family, reproducible as in Figure 3H

## Limitations

- If no significant epialleles are found at this stage, the module falls back to annotating the single highest-frequency epiallele as the stable signature/core, which may not reflect family structure
- Significance testing depends on the random control generated by Module 1; subtle low-frequency (e.g., 10% amplified) epialleles can escape detection by Modules 1 and 2 upstream
- Large dependency burden: the package requires dozens of R packages (ComplexHeatmap, factoextra, entropy, vegan, igraph, etc.) best installed via Install_Dependencies.R, pacman, or docker

## Evidence
