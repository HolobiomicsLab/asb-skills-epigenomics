---
name: background-free-lambda-window-peak-detection
description: Use when you have qbed/ccf Calling Cards insertion data and no matching
  background (spontaneous insertions) control, and you need to call enriched TF binding
  peaks genome-wide.
when_to_use_negative:
- A paired background/qbed control dataset is available — then use the background-controlled
  MACCs or CCcaller variant instead of the lambda-window background-free mode.
- Input is already a peak set or feature table — use pp.filter_peaks, annotation,
  or differential peak analysis rather than re-calling peaks.
- The goal is motif calling or differential peak analysis on existing peaks (tl functions),
  not de novo peak detection from insertions.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0160
tools:
- name: Pycallingcards
  role: Python package providing the MACCs peak caller, qbed reading, preprocessing
    (clean_qbed, call_peaks, filter_peaks), and dataset loaders
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: MACCs
  role: Window-scan peak-calling algorithm within Pycallingcards; used here in its
    background-free lambda-window mode with TTAA-corrected Poisson testing
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: Scanpy
  role: Pycallingcards follows Scanpy structure and interoperates with it for downstream
    single-cell analysis of called peaks
  repo: https://scanpy.readthedocs.io/en/stable/
- name: bedtools
  role: Required dependency for annotating the called peaks
  repo: https://bedtools.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae070
    title: 'Pycallingcards: an integrated environment for visualizing, analyzing,
      and interpreting Calling Cards data'
  source_content_hash: sha256:4c9129541ff1e545ae47d27efea4b66e71a031890ad50a3ba46873f85cd631cc
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# background-free-lambda-window-peak-detection

## Summary

The MACCs peak-calling variant in Pycallingcards that identifies transcription factor binding sites from Calling Cards (CC) insertion data when no background control experiment is available, using a lambda-window strategy and Poisson significance testing with TTAA-site-density correction.

## When to use

Use when you have qbed/ccf Calling Cards insertion data and no matching background (spontaneous insertions) control, and you need to call enriched TF binding peaks genome-wide.

## When NOT to use

- A paired background/qbed control dataset is available — then use the background-controlled MACCs or CCcaller variant instead of the lambda-window background-free mode.
- Input is already a peak set or feature table — use pp.filter_peaks, annotation, or differential peak analysis rather than re-calling peaks.
- The goal is motif calling or differential peak analysis on existing peaks (tl functions), not de novo peak detection from insertions.

## Inputs

- Calling Cards insertion data in qbed or ccf format (bulk or single-cell)
- Reference genome TTAA-site annotation (for density correction)
- Window size and step size parameters for the genome scan

## Outputs

- Peak table with chromosome, start, end, center, and Poisson-test statistics (e.g. maccs_peaks.bed)
- Optionally an Anndata object of peaks after pp.make_Anndata and pp.filter_peaks

## How to apply

Load a calling cards qbed insertion dataset (e.g. the built-in mouse BRD4 data via pycallingcards.datasets.mouse_brd4_data) after optional preprocessing.clean_qbed. Run the MACCs peak caller (pp.call_peaks), which scans the genome with a fixed window and step size to detect windows enriched in experimental insertions, merges consecutively enriched windows into candidate peaks, and records each peak's center. Because no background control exists, invoke the background-free variant that estimates expected insertion rates via a lambda-window approach instead of a paired background sample. Significance of each candidate peak is assessed with a Poisson test, correcting for relative TTAA-site density so that genome-wide TTAA availability does not bias p-values. Write resulting peak coordinates (chromosome, start, end, center, statistics) to a BED file (e.g. maccs_peaks.bed) and optionally filter peaks (pp.filter_peaks) downstream.

## Related tools

- **Pycallingcards** (Python package providing the MACCs peak caller, qbed reading, preprocessing (clean_qbed, call_peaks, filter_peaks), and dataset loaders) — https://github.com/The-Mitra-Lab/pycallingcards
- **MACCs** (Window-scan peak-calling algorithm within Pycallingcards; used here in its background-free lambda-window mode with TTAA-corrected Poisson testing) — https://github.com/The-Mitra-Lab/pycallingcards
- **Scanpy** (Pycallingcards follows Scanpy structure and interoperates with it for downstream single-cell analysis of called peaks) — https://scanpy.readthedocs.io/en/stable/
- **bedtools** (Required dependency for annotating the called peaks) — https://bedtools.readthedocs.io/en/latest/

## Examples

```
import pycallingcards as cc; qbed = cc.datasets.mouse_brd4_data(); peaks = cc.pp.call_peaks(qbed, method='MACCs')
```

## Evaluation signals

- Output BED file contains chromosome, start, end, and center columns for every called peak (schema check).
- Peak coordinates form merged regions of consecutively enriched windows, so consecutive peaks should not overlap when using non-overlapping merge logic.
- Each reported peak carries a Poisson-test p-value computed against the lambda-window background estimate; extreme p-values should not arise from TTAA-density bias.
- Re-analysis of a published dataset (e.g. mouse cortex or glioblastoma CC data) should reproduce known cell-type-specific binding sites.
- Downstream pp.filter_peaks and annotation steps run without errors, confirming the peak output is well-formed.

## Limitations

- Poisson significance depends on TTAA-site-density correction; mis-specified reference TTAA annotation can bias peak calls.
- Designed for TTAA-based Calling Cards insertions; not applicable to unrelated transposon or ChIP peak-calling problems.
- Requires installing Pycallingcards plus bedtools/pybedtools for annotation; pip/GitHub install or the provided Docker image are the supported routes.

## Evidence

- [other] MACCs identifies candidate CC peaks by scanning the genome with a specified window and step size for regions enriched in experimental over background insertions, merging consecutively enriched windows, and finding the center of each candidate peak.: [evidence span withheld — anchor status: not_found]
- [other] Test experimental versus background insertions for each candidate peak using a Poisson significance test, correcting by relative TTAA-site density.: [evidence span withheld — anchor status: not_found]
- [abstract] Pycallingcards introduces two innovative peak callers, CCcaller and MACCs, enhancing the accuracy and speed of pinpointing TF binding sites from CC data.: "Pycallingcards introduces two innovative peak callers, CCcaller and MACCs, enhancing the accuracy and speed of pinpointing TF binding sites from CC data."
- [readme] **Preprocessing (pp)** helps to preprocess data from qbed data to call peaks, make annotation, make Anndata object and filter peaks.: "**Preprocessing (pp)** helps to preprocess data from qbed data to call peaks, make annotation, make Anndata object and filter peaks."
