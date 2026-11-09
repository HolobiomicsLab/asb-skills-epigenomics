---
name: window-based-genome-scanning-peak-calling
description: Use when calling TF binding sites from Calling Cards qbed/ccf insertion
  data, especially when you need genome-wide window-scan peaks with Poisson significance
  tests and an optional background-free lambda-window variant.
when_to_use_negative:
- Input is already a peak set (e.g. existing BED peaks needing annotation, combination,
  or filtering — use combine_peaks/annotation/filter_peaks instead).
- The goal is differential peak analysis or peak-gene expression pairing, which are
  downstream tools (tl) steps applied after peaks are called.
- Input is ChIP-seq or RNA-seq data rather than Calling Cards insertion (qbed/ccf)
  data.
tools:
- name: MACCs
  role: The window-scan peak caller implemented by this skill; one of Pycallingcards'
    two innovative peak callers for pinpointing TF binding sites from CC data.
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: Pycallingcards
  role: Python package providing the datasets (e.g. mouse_brd4_data), reading (rd)
    for qbed/ccf files, and preprocessing (pp) peak-calling functions the skill runs
    inside.
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: CCcaller
  role: Alternative Pycallingcards peak caller to MACCs; relevant as a comparison
    point for accuracy and speed when calling peaks from CC data.
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: bedtools
  role: Required dependency for completing peak annotation after peak calling in the
    Pycallingcards pipeline.
  repo: https://bedtools.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae070
    title: 'Pycallingcards: an integrated environment for visualizing, analyzing,
      and interpreting Calling Cards data'
  source_content_hash: sha256:2bb1491e14adeddbdf639722e46ecdcf6583de24cf7aa15ef0665e50a71c6d42
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/The-Mitra-Lab/pycallingcards
  tools:
  - CCcaller
  - MACCs
  - Pycallingcards
  - Python
  - bedtools
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# window-based-genome-scanning-peak-calling

## Summary

Implements the MACCs peak-calling algorithm from Pycallingcards, which identifies transcription factor binding sites from Calling Cards (CC) qbed insertion data by scanning the genome in windows for enrichment of experimental over background insertions and testing candidates with a TTAA-corrected Poisson model.

## When to use

Use when calling TF binding sites from Calling Cards qbed/ccf insertion data, especially when you need genome-wide window-scan peaks with Poisson significance tests and an optional background-free lambda-window variant.

## When NOT to use

- Input is already a peak set (e.g. existing BED peaks needing annotation, combination, or filtering — use combine_peaks/annotation/filter_peaks instead).
- The goal is differential peak analysis or peak-gene expression pairing, which are downstream tools (tl) steps applied after peaks are called.
- Input is ChIP-seq or RNA-seq data rather than Calling Cards insertion (qbed/ccf) data.

## Inputs

- qbed or ccf Calling Cards insertion file (experimental sample)
- background/control qbed insertion file (optional; if absent, use the lambda-window background-free variant)
- genome TTAA-site density reference for Poisson correction
- built-in pycallingcards dataset (e.g. datasets.mouse_brd4_data, datasets.SP1_K562HCT116_data)

## Outputs

- Peak BED file (maccs_peaks.bed) with chromosome, start, end, peak center, and Poisson test statistics per candidate peak
- Candidate peak regions from merged consecutively enriched windows

## How to apply

Load a calling cards qbed insertion dataset (e.g. the built-in mouse cortex BRD4 data via pycallingcards.datasets). Scan the genome with a fixed window and step size, computing enrichment of experimental insertions per window, then merge consecutively enriched windows into candidate peak regions and locate each peak's center. For each candidate, test experimental versus background insertions with a Poisson significance test, correcting by relative TTAA-site density. When no background control dataset exists, implement the background-free lambda-window variant instead. Write final peaks (chromosome, start, end, center, statistics) to maccs_peaks.bed. Rationale: MACCs is one of two peak callers (with CCcaller) that Pycallingcards introduces to improve accuracy and speed of pinpointing TF binding sites from CC data, and it operates within the package's preprocessing (pp) module on qbed/ccf inputs.

## Related tools

- **MACCs** (The window-scan peak caller implemented by this skill; one of Pycallingcards' two innovative peak callers for pinpointing TF binding sites from CC data.) — https://github.com/The-Mitra-Lab/pycallingcards
- **Pycallingcards** (Python package providing the datasets (e.g. mouse_brd4_data), reading (rd) for qbed/ccf files, and preprocessing (pp) peak-calling functions the skill runs inside.) — https://github.com/The-Mitra-Lab/pycallingcards
- **CCcaller** (Alternative Pycallingcards peak caller to MACCs; relevant as a comparison point for accuracy and speed when calling peaks from CC data.) — https://github.com/The-Mitra-Lab/pycallingcards
- **bedtools** (Required dependency for completing peak annotation after peak calling in the Pycallingcards pipeline.) — https://bedtools.readthedocs.io/en/latest/

## Examples

```
import pycallingcards as pyc; qbed = pyc.datasets.mouse_brd4_data; peaks = pyc.pp.call_peaks(qbed, method="MACCs")
```

## Evaluation signals

- Output BED contains the required columns: chromosome, start, end, center, and Poisson statistics for every peak.
- Each reported peak region corresponds to a merge of consecutive enriched windows, and no enriched windows were silently dropped between merges.
- Poisson p-values show TTAA-site density correction applied (same insertion counts in regions with different TTAA density yield different significance).
- The background-free lambda-window path is exercised when no background qbed is supplied, and results remain bounded/sane in low-insertion regions.
- Peaks from the built-in mouse cortex BRD4 qbed dataset overlap expected TF-binding-enriched regions and pass downstream filter_peaks/annotation steps.

## Limitations

- Requires choosing window and step size parameters; candidates are only found at enriched-window scale, so binding sites narrower or wider than the window may be missed or merged.
- The background-free lambda-window variant is a fallback when no background control is available and is less controlled than the background-based Poisson test.
- bedtools (and pybedtools) must be installed for downstream annotation steps of the peak-calling pipeline.

## Evidence

- [abstract] Pycallingcards introduces two innovative peak callers, CCcaller and MACCs, enhancing the accuracy and speed of pinpointing TF binding sites from CC data.: "Pycallingcards introduces two innovative peak callers, CCcaller and MACCs, enhancing the accuracy and speed of pinpointing TF binding sites from CC data."
- [methods] Preprocessing (pp) helps to preprocess data from qbed/ccf data to call peaks, make annotation, make Anndata object and filter peaks.: "**Preprocessing (pp)** helps to preprocess data from qbed/ccf data to call peaks, make annotation, make Anndata object and filter peaks."
- [readme] Preprocessing (pp) helps to preprocess data from qbed data to call peaks, make annotation, make Anndata object and filter peaks.: "**Preprocessing (pp)** helps to preprocess data from qbed data to call peaks, make annotation, make Anndata object and filter peaks."
- [methods] datasets.mouse_brd4_data: "datasets.mouse_brd4_data"
