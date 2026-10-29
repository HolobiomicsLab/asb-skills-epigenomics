---
name: supplementary-figure-reproduction
description: Use when a task asks to reproduce a specific supplementary figure from
  a published paper (e.g. Supplementary Figure S12) by re-computing the plotted metric
  for all methods on the same validation data.
when_to_use_negative:
- The goal is a qualitative visual re-plot from data already published in a table
  rather than re-running the underlying tool calls and metric computation.
- The figure compares methods on different validation sets — without an identical
  validation set the AUPR comparison is not valid and the skill should not proceed.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0749
tools:
- name: RoboCOP
  role: Multivariate state space model run with ATAC-seq alone and with MNase-seq
    + ATAC-seq to produce nucleosome occupancy predictions whose AUPR is compared
    in the figure
  repo: https://github.com/HarteminkLab/RoboCOP
- name: NucleoATAC
  role: Baseline method run on the same ATAC-seq data to obtain nucleosome-position
    calls for comparison against RoboCOP_ATAC
- name: DANPOS2
  role: Baseline method run on the same MNase-seq data to obtain nucleosome-position
    calls for comparison against RoboCOP_MNase+ATAC
- name: GCC
  role: Used via 'bash gccCompile' to compile the shared library librobocop.so needed
    to invoke the RoboCOP engine
  repo: https://github.com/HarteminkLab/RoboCOP
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkab553
    title: 'RoboCOP: jointly computing chromatin occupancy profiles for numerous factors
      from chromatin accessibility data'
  source_content_hash: sha256:b7bb2f18ccff4312edb2bd361a2fc219efcc226804e976b28deeccf4d37f1a5e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HarteminkLab/RoboCOP
  tools:
  - DANPOS2
  - GCC
  - NucleoATAC
  - RoboCOP
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# supplementary-figure-reproduction

## Summary

Reproduces a paper's supplementary figure by re-running the underlying analyses on the same public datasets, computing the reported metric (here nucleosome-position AUPR) for all compared methods on an identical validation set, and rendering a figure matching the original layout.

## When to use

Use when a task asks to reproduce a specific supplementary figure from a published paper (e.g. Supplementary Figure S12) by re-computing the plotted metric for all methods on the same validation data.

## When NOT to use

- The goal is a qualitative visual re-plot from data already published in a table rather than re-running the underlying tool calls and metric computation.
- The figure compares methods on different validation sets — without an identical validation set the AUPR comparison is not valid and the skill should not proceed.

## Inputs

- ATAC-seq data from GEO accession GSE66386
- MNase-seq data from GEO accession GSE153609
- Chemical-cleavage method nucleosome validation annotations
- RoboCOP configuration file (config.ini) with cshared path set
- RoboCOP shared library librobocop.so built via gccCompile

## Outputs

- nucleosome_aupr_scores.csv containing AUPR values for RoboCOP (ATAC-only and MNase+ATAC), DANPOS2, and NucleoATAC
- supplementary_figure_S12_aupr_comparison.png reproducing the Supplementary Figure S12 layout
- Nucleosome occupancy predictions from RoboCOP (ATAC-only and MNase+ATAC), NucleoATAC, and DANPOS2 on the shared input data

## How to apply

First recreate the software environment: clone the RoboCOP repository, create the conda environment from robocop-spec-file.txt, run 'bash gccCompile' in the robocop directory to build librobocop.so, and set the cshared entry in config.ini to the absolute path of that library. Run each compared method on the same raw inputs as the paper — RoboCOP with ATAC-seq alone (GSE66386), RoboCOP with MNase-seq (GSE153609) plus ATAC-seq, NucleoATAC on the ATAC-seq data, and DANPOS2 on the MNase-seq data. Evaluate all four methods' nucleosome-position AUPR against the chemical-cleavage validation annotations, ensuring the identical validation set is used across methods so the comparison is fair. Write the scores to nucleosome_aupr_scores.csv, then render the figure (supplementary_figure_S12_aupr_comparison.png) mirroring the paper's S12 layout: RoboCOP_ATAC vs NucleoATAC and RoboCOP_MNase+ATAC vs DANPOS2. Success is judged when all AUPR values are computed on the identical validation set and the rendered figure reproduces the S12 layout.

## Related tools

- **RoboCOP** (Multivariate state space model run with ATAC-seq alone and with MNase-seq + ATAC-seq to produce nucleosome occupancy predictions whose AUPR is compared in the figure) — https://github.com/HarteminkLab/RoboCOP
- **NucleoATAC** (Baseline method run on the same ATAC-seq data to obtain nucleosome-position calls for comparison against RoboCOP_ATAC)
- **DANPOS2** (Baseline method run on the same MNase-seq data to obtain nucleosome-position calls for comparison against RoboCOP_MNase+ATAC)
- **GCC** (Used via 'bash gccCompile' to compile the shared library librobocop.so needed to invoke the RoboCOP engine) — https://github.com/HarteminkLab/RoboCOP

## Examples

```
git clone https://github.com/HarteminkLab/RoboCOP.git; cd RoboCOP/robocop && bash gccCompile; ls "`pwd`/librobocop.so"
```

## Evaluation signals

- All four methods' AUPR values are computed on the identical chemical-cleavage validation set (same genomic coordinates for every method).
- nucleosome_aupr_scores.csv exists and contains AUPR entries for RoboCOP_ATAC, RoboCOP_MNase+ATAC, DANPOS2, and NucleoATAC.
- The rendered supplementary_figure_S12_aupr_comparison.png reproduces the S12 panel layout: RoboCOP_ATAC vs NucleoATAC and RoboCOP_MNase+ATAC vs DANPOS2.
- RoboCOP run with ATAC-seq alone yields more nucleosome calls than NucleoATAC, matching Supplementary Figure S12A.
- librobocop.so was successfully built and the cshared path in config.ini resolves to it before any RoboCOP run.

## Limitations

- NucleoATAC reports only a small fraction of genomic nucleosomes (~20%), so a naive comparison of call counts against tools that call more nucleosomes can be misleading without the AUPR metric.
- Markedly better RoboCOP performance is observed with MNase-seq than ATAC-seq due to genome-wide fragment distribution and MNase exonuclease activity reflecting DBF sizes; ATAC-only results should be interpreted with this in mind.
- No tool version numbers are stated in the article for RoboCOP, NucleoATAC, DANPOS2, MACS2, or FIMO, so exact version pinning cannot be verified from the source.
- The paper's figures are generated via notebooks such as dynacop_figures.ipynb in the repository; if the exact notebook for S12 is unavailable, figure layout must be approximated.

## Evidence

- [supplementary] RoboCOP run with ATAC-seq alone finds more nucleosomes than NucleoATAC, another method that incorporates ATAC-seq to find nucleosome positions (Supplementary Figure S12A).: [evidence span withheld — anchor status: not_found]
