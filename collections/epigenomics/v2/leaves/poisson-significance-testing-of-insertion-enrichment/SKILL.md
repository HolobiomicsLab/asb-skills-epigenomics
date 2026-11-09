---
name: poisson-significance-testing-of-insertion-enrichment
description: Use when bulk or single-cell Calling Cards (CC) qbed insertion data must
  be converted into statistically significant TF binding-site peaks and no ready-made
  peak set is available.
when_to_use_negative:
- Input is already a called peak set or a filtered Anndata peak matrix rather than
  raw qbed insertions
- The goal is motif calling, differential peak analysis, or pairing peaks with RNA-seq
  gene expression (use pycallingcards tools (tl) instead of peak calling)
- Data are ChIP-seq or CUT&RUN signals, not Calling Cards transposase insertion sites
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: Pycallingcards
  role: Python module providing the qbed readers, datasets, and preprocessing (pp)
    functions that wrap MACCs peak calling
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: MACCs
  role: window-scan peak caller within Pycallingcards that performs the Poisson significance
    testing of insertion enrichment
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: bedtools
  role: required dependency for annotating the called peaks
  repo: https://bedtools.readthedocs.io/en/latest/
- name: Scanpy
  role: interface for single-cell Calling Cards analysis; Pycallingcards follows Scanpy's
    object structure (Anndata)
  repo: https://scanpy.readthedocs.io/en/stable/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae070
    title: 'Pycallingcards: an integrated environment for visualizing, analyzing,
      and interpreting Calling Cards data'
  source_content_hash: sha256:5aa1f600b7d20e71e499146766c15b443d5b5227c799ac29017f842baceee847
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/The-Mitra-Lab/pycallingcards
  tools:
  - MACCs
  - Pycallingcards
  - Python
  - Scanpy
  - bedtools
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# poisson-significance-testing-of-insertion-enrichment

## Summary

Implements the MACCs peak-calling approach from Pycallingcards: scanning a Calling Cards qbed genome-wide for windows enriched in experimental over background insertions, then assessing each candidate peak with a Poisson significance test corrected by relative TTAA-site density.

## When to use

Use when bulk or single-cell Calling Cards (CC) qbed insertion data must be converted into statistically significant TF binding-site peaks and no ready-made peak set is available.

## When NOT to use

- Input is already a called peak set or a filtered Anndata peak matrix rather than raw qbed insertions
- The goal is motif calling, differential peak analysis, or pairing peaks with RNA-seq gene expression (use pycallingcards tools (tl) instead of peak calling)
- Data are ChIP-seq or CUT&RUN signals, not Calling Cards transposase insertion sites

## Inputs

- qbed Calling Cards insertion file (bulk or single-cell, e.g. mouse cortex BRD4 data from pycallingcards.datasets.mouse_brd4_data)
- optional background/control insertion dataset (qbed)
- genome reference/build identifier (e.g. mm10) for TTAA-site density correction

## Outputs

- peak table with chromosome, start, end, center, and Poisson-test statistics
- peak file written as maccs_peaks.bed
- optionally an Anndata object after preprocessing.make_Anndata for downstream filtering

## How to apply

Load the qbed insertion dataset (e.g. the built-in mouse cortex BRD4 data via pycallingcards datasets) and scan the genome with a specified window and step size, computing experimental-vs-background insertion enrichment per window. Merge consecutively enriched windows into candidate peak regions and find each peak's center, mirroring MACCs' window-scan strategy. For each candidate, test experimental versus background insertions with a Poisson significance test, correcting by relative TTAA-site density to account for TTAA-dependent insertion bias. When no background control exists, implement the background-free lambda-window variant instead. Write the final coordinates and statistics (chromosome, start, end, center) to maccs_peaks.bed; Pycallingcards' preprocessing (pp) module wraps call_peaks for this purpose.

## Related tools

- **Pycallingcards** (Python module providing the qbed readers, datasets, and preprocessing (pp) functions that wrap MACCs peak calling) — https://github.com/The-Mitra-Lab/pycallingcards
- **MACCs** (window-scan peak caller within Pycallingcards that performs the Poisson significance testing of insertion enrichment) — https://github.com/The-Mitra-Lab/pycallingcards
- **bedtools** (required dependency for annotating the called peaks) — https://bedtools.readthedocs.io/en/latest/
- **Scanpy** (interface for single-cell Calling Cards analysis; Pycallingcards follows Scanpy's object structure (Anndata)) — https://scanpy.readthedocs.io/en/stable/

## Examples

```
import pycallingcards as cc; qbed = cc.datasets.mouse_brd4_data(); peaks = cc.pp.call_peaks(qbed, method = 'MACCs', reference = 'mm10')
```

## Evaluation signals

- Called peaks carry all four coordinate fields (chromosome, start, end, center) plus Poisson-test statistics and are written to maccs_peaks.bed
- Peak regions correspond to merged runs of consecutively enriched windows — no overlapping unmerged enriched windows remain in the output
- P-value computation uses a TTAA-corrected Poisson model; swapping in the lambda-window background-free variant should only be needed when no background control is supplied
- Re-running on a built-in dataset (e.g. datasets.mouse_brd4_data) yields peaks consistent with the published mouse cortex/glioblastoma reanalyses in Pycallingcards
- Downstream preprocessing.filter_peaks and make_Anndata accept the peaks without coordinate parsing errors

## Limitations

- Requires TTAA-site density correction; without it, Poisson significance is biased by the TTAA-preference of the calling cards transposase system
- Window and step size parameters must be chosen; scanning performance and peak granularity depend on these settings
- Pycallingcards additionally requires bedtools/pybedtools to be installed for annotation of the called peaks

## Evidence

- [abstract] Pycallingcards introduces two innovative peak callers, CCcaller and MACCs, enhancing the accuracy and speed of pinpointing TF binding sites from CC data.: "Pycallingcards introduces two innovative peak callers, CCcaller and MACCs, enhancing the accuracy and speed of pinpointing TF binding sites from CC data."
- [methods] Pycallingcards is a python package designed for both single-cell and bulk calling cards data analysis.: "Pycallingcards is a python package designed for both single-cell and bulk calling cards data analysis."
- [readme] Preprocessing (pp) helps to preprocess data from qbed data to call peaks, make annotation, make Anndata object and filter peaks.: "**Preprocessing (pp)** helps to preprocess data from qbed data to call peaks, make annotation, make Anndata object and filter peaks."
- [methods] In order to complete the annotation, bedtools (includes pybedtools) also needs to be installed.: "In order to complete the annotation, [bedtools](https://bedtools.readthedocs.io/en/latest/) (includes [pybedtools](https://daler.github.io/pybedtools/)) also needs to be installed."
