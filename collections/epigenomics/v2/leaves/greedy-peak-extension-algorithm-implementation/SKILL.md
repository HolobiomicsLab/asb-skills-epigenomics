---
name: greedy-peak-extension-algorithm-implementation
description: Use when calling TF binding sites from Calling Cards qbed/ccf insertion
  data (e.g. Brd4 K562, GSE148448) and a fast, single-pass greedy extension peak caller
  with Poisson significance promotion is needed.
when_to_use_negative:
- Input is already a called peak set or Anndata object rather than raw qbed insertions
  — use downstream filtering steps like preprocessing.filter_peaks instead
- The analysis goal is differential peak analysis, motif calling, or peak–gene pairing,
  which belong to the tools (tl) module rather than peak calling
- The data are not Calling Cards insertions (e.g. bulk ChIP-seq signal tracks), since
  the greedy insertion-extension logic assumes discrete insertion coordinates
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0749
tools:
- name: pycallingcards
  role: Python package whose preprocessing (pp) module implements peak calling, annotation,
    Anndata construction and peak filtering on qbed data
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: CCcaller
  role: Greedy peak caller whose single-pass extension algorithm is being implemented
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: MACCs
  role: Provides the peak-promotion logic (Poisson enrichment over background insertions)
    that CCcaller reuses for promoting candidate peaks to CC peaks
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: Python
  role: Implementation language for the greedy extension and Poisson enrichment computation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae070
    title: 'Pycallingcards: an integrated environment for visualizing, analyzing,
      and interpreting Calling Cards data'
  source_content_hash: sha256:d1e136467cddc930e05bfa2b45cea74e66ca7c81a5198f03a0c951c24b417ca1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/The-Mitra-Lab/pycallingcards
  tools:
  - CCcaller
  - MACCs
  - Python
  - pycallingcards
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# greedy-peak-extension-algorithm-implementation

## Summary

Implements the CCcaller greedy peak-calling algorithm from Pycallingcards: candidate peaks are grown by single-pass extension over sorted qbed insertions and promoted to CC peaks via Poisson enrichment testing, mirroring the MACCs promotion logic.

## When to use

Use when calling TF binding sites from Calling Cards qbed/ccf insertion data (e.g. Brd4 K562, GSE148448) and a fast, single-pass greedy extension peak caller with Poisson significance promotion is needed.

## When NOT to use

- Input is already a called peak set or Anndata object rather than raw qbed insertions — use downstream filtering steps like preprocessing.filter_peaks instead
- The analysis goal is differential peak analysis, motif calling, or peak–gene pairing, which belong to the tools (tl) module rather than peak calling
- The data are not Calling Cards insertions (e.g. bulk ChIP-seq signal tracks), since the greedy insertion-extension logic assumes discrete insertion coordinates

## Inputs

- qbed/ccf file of Calling Cards transposon insertions (chromosome, start, end, read info)
- built-in Pycallingcards datasets (e.g. datasets.mouse_brd4_data, datasets.SP1_K562HCT116_data) or GEO data such as GSE148448
- user-specified maximum distance in base pairs between adjacent insertions for peak extension

## Outputs

- CC peaks bed file (e.g. cccaller_peaks.bed) with chromosome, start, end, and peak statistics
- candidate peak regions with Poisson enrichment p-values

## How to apply

Load the qbed insertion data (e.g. via pycallingcards.datasets, such as the built-in Brd4 K562 data from GEO accession GSE148448) into a dataframe and group insertions by chromosome. Start a candidate peak at the first insertion on each chromosome, then greedily extend it by appending the next nearest insertion while it lies within a user-specified maximum base-pair distance of the current peak boundary, traversing each insertion exactly once in a single ordered pass. For each candidate peak, compute the Poisson enrichment p-value of observed insertions against the background insertion rate and promote it to a CC peak using the same promotion logic as the MACCs method (preprocessing.call_peaks-style calling). Write promoted peaks (chromosome, start, end, statistics) to a bed file such as cccaller_peaks.bed. The greedy single-pass design is what makes CCcaller computationally efficient compared with multi-pass peak refinement.

## Related tools

- **pycallingcards** (Python package whose preprocessing (pp) module implements peak calling, annotation, Anndata construction and peak filtering on qbed data) — https://github.com/The-Mitra-Lab/pycallingcards
- **CCcaller** (Greedy peak caller whose single-pass extension algorithm is being implemented) — https://github.com/The-Mitra-Lab/pycallingcards
- **MACCs** (Provides the peak-promotion logic (Poisson enrichment over background insertions) that CCcaller reuses for promoting candidate peaks to CC peaks) — https://github.com/The-Mitra-Lab/pycallingcards
- **Python** (Implementation language for the greedy extension and Poisson enrichment computation)

## Examples

```
import pycallingcards as cc; qbed = cc.datasets.SP1_K562HCT116_data(); peaks = cc.pp.call_peaks(qbed, method="cccaller", reference="hg38"); cc.rd.save_peak(peaks, filename="cccaller_peaks.bed")
```

## Evaluation signals

- Every insertion on each chromosome is visited exactly once in the sorted single pass (no re-scanning of insertions)
- Each output CC peak passes the Poisson enrichment significance threshold against the background insertion rate
- No two adjacent insertions within the user-specified maximum distance are split across different peaks
- Output bed file contains the expected columns (chromosome, start, end, statistics) and peak coordinates agree with the union of extended insertion runs
- Called peaks are consistent with peaks obtained via preprocessing.call_peaks on the same qbed input

## Limitations

- Peak quality depends on the user-specified maximum distance parameter for extension; the article does not prescribe a universal value
- CCcaller promotes candidate peaks exactly as the MACCs method does, so results inherit the assumptions of the Poisson background model over background insertions
- Single-cell Calling Cards workflows additionally require Anndata construction and interaction with Scanpy, which is outside the scope of this peak-calling step

## Evidence

- [abstract] Pycallingcards introduces two innovative peak callers, CCcaller and MACCs, enhancing the accuracy and speed of pinpointing TF binding sites from CC data.: "Pycallingcards introduces two innovative peak callers, CCcaller and MACCs, enhancing the accuracy and speed of pinpointing TF binding sites from CC data."
- [intro] Preprocessing (pp) helps to preprocess data from qbed data to call peaks, make annotation, make Anndata object and filter peaks.: "**Preprocessing (pp)** helps to preprocess data from qbed data to call peaks, make annotation, make Anndata object and filter peaks."
- [methods] it can be downloaded from GEO (GSE148448).: "it can be downloaded from [GEO](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE148448)."
- [methods] Reading (rd) includes several functions to read and save qbed/ccf and peak data.: "**Reading (rd)** includes several functions to read and save qbed/ccf and peak data."
- [readme] pip install pycallingcards: "pip install pycallingcards"
