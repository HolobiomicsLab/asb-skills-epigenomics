---
name: qbed-peak-calling-with-cccaller
description: Use when the input is qbed/ccf calling cards insertion data (e.g., Brd4
  scCC insertions from GEO) and TF binding sites must be called as peaks for annotation,
  AnnData construction, or peak-gene pairing.
when_to_use_negative:
- Input is already a peak file or feature-count matrix — use downstream tools (rank_peak_groups,
  pair_peak_gene_sc) instead of re-calling peaks
- Data are standard scRNA-seq or ChIP-seq reads rather than calling cards qbed/ccf
  insertions — use Scanpy or ChIP-seq peak callers instead
- Only motif or visualization analysis is needed and peaks already exist
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0602
tools:
- name: pycallingcards
  role: Python module providing reading (read_qbed), preprocessing (clean_qbed, call_peaks,
    annotation, make_Anndata, filter_peaks) for the whole peak-calling workflow
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: CCcaller
  role: Primary peak caller that pinpoints TF binding sites from qbed/ccf insertion
    data
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: bedtools
  role: Backend (with pybedtools) used to complete peak annotation with nearest genes
  repo: https://bedtools.readthedocs.io/en/latest/
- name: Scanpy
  role: Pycallingcards follows the Scanpy structure and interacts with it for single-cell
    calling cards analysis
  repo: https://scanpy.readthedocs.io/en/stable/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae070
    title: 'Pycallingcards: an integrated environment for visualizing, analyzing,
      and interpreting Calling Cards data'
  source_content_hash: sha256:c48e3924919e8f581f24dfcf0d8acca849b466ad4b07e420a5067e3e9c004fe0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/The-Mitra-Lab/pycallingcards
  tools:
  - CCcaller
  - Mudata
  - Scanpy
  - bedtools
  - pycallingcards
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# qbed-peak-calling-with-cccaller

## Summary

Calls transcription factor binding peaks from single-cell or bulk calling cards qbed/ccf insertion data using the CCcaller peak caller within Pycallingcards, then annotates and formats peaks for downstream binding-expression analysis.

## When to use

Use when the input is qbed/ccf calling cards insertion data (e.g., Brd4 scCC insertions from GEO) and TF binding sites must be called as peaks for annotation, AnnData construction, or peak-gene pairing.

## When NOT to use

- Input is already a peak file or feature-count matrix — use downstream tools (rank_peak_groups, pair_peak_gene_sc) instead of re-calling peaks
- Data are standard scRNA-seq or ChIP-seq reads rather than calling cards qbed/ccf insertions — use Scanpy or ChIP-seq peak callers instead
- Only motif or visualization analysis is needed and peaks already exist

## Inputs

- qbed/ccf calling cards insertion file (e.g., from GEO accession GSE148448 or via pycallingcards.datasets.mousecortex_data)
- reference genome annotation for nearest-gene annotation via bedtools/pybedtools
- optional group/cell-type labels defining peak-calling groups

## Outputs

- bed-format peak file of called TF binding sites (e.g., ~902 Brd4 peaks in the mouse cortex reanalysis)
- annotated peak table with nearest gene(s)
- cell-by-peak AnnData object suitable for combination with gene expression into a MuData object

## How to apply

Load the qbed insertion data with pycallingcards.reading.read_qbed (e.g., the mouse cortex Brd4 scCC dataset GSE148448), optionally cleaning it via preprocessing.clean_qbed. Call peaks with cc.pp.call_peaks using the CCcaller peak caller, which the article describes as one of two innovative peak callers enhancing accuracy and speed of pinpointing TF binding sites; for grouped/single-cell data call peaks per group and combine or separate with cc.pp.combine_peaks/cc.pp.separate_peaks as needed. Annotate each called peak with its nearest genes via cc.pp.annotation (bedtools/pybedtools must be installed). Convert the peaks to a cell-by-peak count object with cc.pp.make_Anndata and optionally apply cc.pp.filter_peaks to remove low-quality peaks. Validation: the called peak set should match the expected count (e.g., ~902 peaks for the mouse cortex Brd4 reanalysis) and coordinates should be bed-format chrom/start-end.

## Related tools

- **pycallingcards** (Python module providing reading (read_qbed), preprocessing (clean_qbed, call_peaks, annotation, make_Anndata, filter_peaks) for the whole peak-calling workflow) — https://github.com/The-Mitra-Lab/pycallingcards
- **CCcaller** (Primary peak caller that pinpoints TF binding sites from qbed/ccf insertion data) — https://github.com/The-Mitra-Lab/pycallingcards
- **bedtools** (Backend (with pybedtools) used to complete peak annotation with nearest genes) — https://bedtools.readthedocs.io/en/latest/
- **Scanpy** (Pycallingcards follows the Scanpy structure and interacts with it for single-cell calling cards analysis) — https://scanpy.readthedocs.io/en/stable/

## Examples

```
import pycallingcards as cc; qbed = cc.rd.read_qbed('GSE148448_brd4.qbed'); peaks = cc.pp.call_peaks(qbed, peak_type='CCcaller'); cc.pp.annotation(peaks, reference='mm10')
```

## Evaluation signals

- Called peak count matches the reported reference set (e.g., ~902 peaks for the mouse cortex Brd4 scCC reanalysis)
- Peak coordinates are valid bed intervals; reported examples such as chr13:83141353-83148478 and chr4:97575305-97588788 are recovered
- Each peak carries nearest-gene annotations (e.g., Mef2c and E130114P18Rik for the reported examples)
- make_Anndata output is a cell-by-peak AnnData compatible with combining into a MuData object with the scRNA-seq AnnData
- filter_peaks removes low-support peaks without eliminating the reported cell-type-specific peaks

## Limitations

- bedtools (including pybedtools) must be installed for annotation; the workflow fails without this system dependency
- Peak calling is specific to calling cards insertion data (qbed/ccf), not general ChIP-seq or RNA-seq reads
- Peak counts depend on group definitions and filtering choices; reproducing a published peak set requires matching the group structure and preprocessing steps

## Evidence

- [abstract] Pycallingcards introduces two innovative peak callers, CCcaller and MACCs, enhancing the accuracy and speed of pinpointing TF binding sites from CC data.: "Pycallingcards introduces two innovative peak callers, CCcaller and MACCs, enhancing the accuracy and speed of pinpointing TF binding sites from CC data."
- [intro] Preprocessing (pp) helps to preprocess data from qbed data to call peaks, make annotation, make Anndata object and filter peaks.: "**Preprocessing (pp)** helps to preprocess data from qbed data to call peaks, make annotation, make Anndata object and filter peaks."
- [methods] In order to complete the annotation, bedtools (includes pybedtools) also needs to be installed.: "In order to complete the annotation, [bedtools](https://bedtools.readthedocs.io/en/latest/) (includes [pybedtools](https://daler.github.io/pybedtools/)) also needs to be installed."
- [methods] Two Anndata objects can combine into one Mudata object.: "Two Anndata objects can combine into one Mudata object."
- [methods] Reading (rd) includes several functions to read and save qbed/ccf and peak data.: "**Reading (rd)** includes several functions to read and save qbed/ccf and peak data."
