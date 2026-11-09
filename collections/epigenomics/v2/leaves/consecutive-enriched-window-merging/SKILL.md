---
name: consecutive-enriched-window-merging
description: Use when peak-calling Calling Cards (CC) insertions from a qbed dataset
  and you need candidate TF binding regions formed by merging windows that are consecutively
  enriched in experimental over background insertions, as in the MACCs algorithm.
when_to_use_negative:
- Input is already a called-peak BED/peak file rather than raw qbed insertions — use
  annotation or filter_peaks instead of re-calling
- Analysis goal is differential peak analysis or motif calling on existing peaks (Tools
  (tl) stage), not candidate-region discovery
- Data is gene-expression or ChIP-seq signal being compared against CC peaks rather
  than CC insertion data itself
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: Pycallingcards
  role: Python package providing the MACCs peak caller, qbed reading (rd), and preprocessing
    (pp) pipeline in which window-scan merging runs
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: MACCs
  role: The peak caller implementing the window scan, merging of consecutively enriched
    windows, and TTAA-corrected Poisson testing
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: bedtools
  role: Required dependency for completing annotation of the called peaks
  repo: https://bedtools.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae070
    title: 'Pycallingcards: an integrated environment for visualizing, analyzing,
      and interpreting Calling Cards data'
  source_content_hash: sha256:4200f37db5d875a7942d7cfcb704e94e7aa61088948cf943760c6385f4eda4df
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/The-Mitra-Lab/pycallingcards
  tools:
  - MACCs
  - Pycallingcards
  - Python
  - bedtools
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# consecutive-enriched-window-merging

## Summary

The window-scan candidate-peak stage of the MACCs peak caller in Pycallingcards: scan a qbed insertion dataset with a fixed window and step size, mark windows enriched in experimental over background insertions, and merge consecutively enriched windows into candidate CC peak regions whose centers are then Poisson-tested with TTAA-site density correction.

## When to use

Use when peak-calling Calling Cards (CC) insertions from a qbed dataset and you need candidate TF binding regions formed by merging windows that are consecutively enriched in experimental over background insertions, as in the MACCs algorithm.

## When NOT to use

- Input is already a called-peak BED/peak file rather than raw qbed insertions — use annotation or filter_peaks instead of re-calling
- Analysis goal is differential peak analysis or motif calling on existing peaks (Tools (tl) stage), not candidate-region discovery
- Data is gene-expression or ChIP-seq signal being compared against CC peaks rather than CC insertion data itself

## Inputs

- qbed file of Calling Cards insertions (bulk or single-cell, e.g. built-in mouse cortex BRD4 data)
- background/control insertion data (optional; triggers lambda-window variant when absent)
- window size and step size parameters for the genome scan

## Outputs

- candidate peak regions (chromosome, start, end, center) from merged consecutively enriched windows
- Poisson test statistics with TTAA-site density correction per peak
- BED file of called peaks (e.g. maccs_peaks.bed)

## How to apply

Load a calling cards qbed insertion dataset (e.g. pycallingcards.datasets.mouse_brd4_data or mouse cortex BRD4 data) and scan the genome with a specified window and step size, computing insertion enrichment of experimental over background insertions per window. Windows that pass the enrichment criterion are merged when they occur consecutively, producing candidate peak regions; the center of each merged region is recorded as the candidate peak's center. Each candidate peak is then assessed with a Poisson significance test comparing experimental versus background insertions, corrected by relative TTAA-site density; when no background control exists, use the MACCs background-free lambda-window variant. Rationale: merging consecutive enriched windows reduces fragmentation of contiguous insertion clusters before statistical filtering, which is why MACCs enhances accuracy and speed of pinpointing TF binding sites. Write final peaks (chromosome, start, end, center, statistics) to a BED file such as maccs_peaks.bed.

## Related tools

- **Pycallingcards** (Python package providing the MACCs peak caller, qbed reading (rd), and preprocessing (pp) pipeline in which window-scan merging runs) — https://github.com/The-Mitra-Lab/pycallingcards
- **MACCs** (The peak caller implementing the window scan, merging of consecutively enriched windows, and TTAA-corrected Poisson testing) — https://github.com/The-Mitra-Lab/pycallingcards
- **bedtools** (Required dependency for completing annotation of the called peaks) — https://bedtools.readthedocs.io/en/latest/

## Examples

```
import pycallingcards as cc; qbed = cc.datasets.mouse_brd4_data(); peaks = cc.pp.call_peaks(qbed, method="MACCs", reference="mm10"); cc.rd.save_peak(peaks, filename="maccs_peaks.bed")
```

## Evaluation signals

- Every candidate peak's genomic interval is a contiguous span of merged windows that each passed the enrichment criterion — no peak contains a gap between windows
- Each peak has a recorded center within its start–end coordinates and Poisson p-value with TTAA correction
- Output BED file contains the required columns: chromosome, start, end, center, statistics
- Downstream filter_peaks on the resulting Anndata object should retain a plausible fraction of candidate peaks (poorly significant candidates removed)
- Called peaks overlap known TF binding sites/motifs when compared against ChIP-seq data, as Pycallingcards supports comparative analysis with ChIP-seq datasets

## Limitations

- Without a background control dataset, the method must fall back on the background-free lambda-window variant, changing the statistical basis
- Results depend on the chosen window and step size; windows too large or too coarse blur peak boundaries
- TTAA-site density correction assumes insertion bias toward TTAA sites; validity depends on the transposon system used

## Evidence

- [other] MACCs identifies candidate CC peaks by scanning the genome with a specified window and step size for regions enriched in experimental over background insertions, merging consecutively enriched windows, and finding the center of each candidate peak.: [evidence span withheld — anchor status: not_found]
- [abstract] Pycallingcards introduces two innovative peak callers, CCcaller and MACCs, enhancing the accuracy and speed of pinpointing TF binding sites from CC data.: "two innovative peak callers, CCcaller and MACCs, enhancing the accuracy and speed of pinpointing TF binding sites from CC data"
- [readme] **Preprocessing (pp)** helps to preprocess data from qbed data to call peaks, make annotation, make Anndata object and filter peaks.: "helps to preprocess data from qbed data to call peaks, make annotation, make Anndata object and filter peaks"
- [methods] **Reading (rd)** includes several functions to read and save qbed/ccf and peak data.: "several functions to read and save qbed/ccf and peak data"
