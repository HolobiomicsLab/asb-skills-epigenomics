---
name: single-cell-qbed-data-processing
description: Use when the input is raw single-cell calling cards qbed/ccf insertion
  data and the goal is cleaned insertions, called binding peaks, genomic annotation,
  and a cell-by-peak AnnData/Mudata object ready for differential analysis.
when_to_use_negative:
- Input is already a called/annotated peak table or cell-by-peak feature matrix —
  skip preprocessing and go directly to tools (tl) differential peak analysis.
- Analysis targets bulk calling cards data with a different pipeline rather than single-cell
  qbed preprocessing.
- Peak calling is intended with external callers (e.g., MACS-style ChIP-seq callers)
  rather than the CC-specific CCcaller/MACCs callers designed for CC insertion distributions.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0203
tools:
- name: Pycallingcards
  role: core Python module providing rd, pp, tl, pl modules for qbed reading, cleaning,
    peak calling, annotation, AnnData/Mudata construction, and plotting
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: CCcaller
  role: peak caller used within preprocessing.call_peaks to pinpoint TF binding sites
    from cleaned single-cell CC insertions (e.g., maxbetween=1100)
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: Scanpy
  role: single-cell analysis framework whose structure pycallingcards follows; consumes
    the produced AnnData objects
  repo: https://scanpy.readthedocs.io/en/stable/
- name: Mudata
  role: combines the peak AnnData and gene-expression AnnData into one Mudata object
    for joint analysis
- name: bedtools
  role: backend (with pybedtools) required for preprocessing.annotation of called
    peaks
  repo: https://bedtools.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae070
    title: 'Pycallingcards: an integrated environment for visualizing, analyzing,
      and interpreting Calling Cards data'
  source_content_hash: sha256:f6a49579df017beb068f1084c323777ecd31a9020fb2d9772513f82225d4b81e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/The-Mitra-Lab/pycallingcards
  tools:
  - CCcaller
  - Mudata
  - Pycallingcards
  - Python
  - Scanpy
  - bedtools
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# single-cell-qbed-data-processing

## Summary

A Pycallingcards-based workflow that takes raw single-cell calling cards qbed insertion records (e.g., Brd4 GBM data from GEO GSE148448, Moudgil et al., Cell 2020), cleans them, calls and annotates peaks, and builds cell-by-peak AnnData objects for downstream differential binding and peak-gene pairing analysis.

## When to use

Use when the input is raw single-cell calling cards qbed/ccf insertion data and the goal is cleaned insertions, called binding peaks, genomic annotation, and a cell-by-peak AnnData/Mudata object ready for differential analysis.

## When NOT to use

- Input is already a called/annotated peak table or cell-by-peak feature matrix — skip preprocessing and go directly to tools (tl) differential peak analysis.
- Analysis targets bulk calling cards data with a different pipeline rather than single-cell qbed preprocessing.
- Peak calling is intended with external callers (e.g., MACS-style ChIP-seq callers) rather than the CC-specific CCcaller/MACCs callers designed for CC insertion distributions.

## Inputs

- single-cell calling cards qbed/ccf insertion files (e.g., Brd4 qbed datasets from GEO GSE148448)
- paired single-cell gene expression data (AnnData) for Mudata integration
- reference genomic annotation for peak annotation (bedtools/pybedtools backend)

## Outputs

- cleaned qbed insertion records
- called peak coordinates per condition (CCcaller/MACCs output)
- annotated peaks (genomic feature assignments)
- cell-by-peak AnnData object; combined peak+expression Mudata object

## How to apply

Load the qbed datasets with pycallingcards reading (rd) functions and clean the insertion records with preprocessing.clean_qbed. Combine peaks across experimental conditions with preprocessing.combine_peaks, then call peaks on the joint dataset with preprocessing.call_peaks using the CCcaller peak caller (e.g., maxbetween=1100 in the GBM Brd4 reanalysis). Annotate called peaks against genomic features with preprocessing.annotation (bedtools/pybedtools backend), then build cell-by-peak AnnData matrices with preprocessing.make_Anndata and optionally filter peaks with preprocessing.filter_peaks; combine peak and gene-expression AnnData objects into a Mudata object for joint single-cell analysis. The Scanpy-like structure (datasets, rd, pp, tl, pl) means outputs are directly usable by Scanpy-compatible tools for differential peak testing and visualization.

## Related tools

- **Pycallingcards** (core Python module providing rd, pp, tl, pl modules for qbed reading, cleaning, peak calling, annotation, AnnData/Mudata construction, and plotting) — https://github.com/The-Mitra-Lab/pycallingcards
- **CCcaller** (peak caller used within preprocessing.call_peaks to pinpoint TF binding sites from cleaned single-cell CC insertions (e.g., maxbetween=1100)) — https://github.com/The-Mitra-Lab/pycallingcards
- **Scanpy** (single-cell analysis framework whose structure pycallingcards follows; consumes the produced AnnData objects) — https://scanpy.readthedocs.io/en/stable/
- **Mudata** (combines the peak AnnData and gene-expression AnnData into one Mudata object for joint analysis)
- **bedtools** (backend (with pybedtools) required for preprocessing.annotation of called peaks) — https://bedtools.readthedocs.io/en/latest/

## Examples

```
import pycallingcards as cc; qbed = cc.rd.read_qbed('GBM_brd4.qbed'); qbed_clean = cc.pp.clean_qbed(qbed); peaks = cc.pp.call_peaks(qbed_clean, reference='mm10', maxbetween=1100, method='CCcaller')
```

## Evaluation signals

- preprocessing.clean_qbed runs without dropping all records and insertion counts per condition remain consistent with the source qbed files
- combined peak calling with CCcaller produces a non-empty peak set covering both male and female conditions (joint peak call before condition-level separation)
- preprocessing.annotation assigns genomic features to called peaks and completes without bedtools/pybedtools errors
- resulting AnnData has cells x peaks dimensions consistent with the input qbed cell barcodes and called peak number
- the peak AnnData and expression AnnData combine into a valid Mudata object with matched cell observations

## Limitations

- bedtools (including pybedtools) must be installed for annotation to work; annotation fails without it
- CC-specific peak calling (CCcaller/MACCs) is required — generic ChIP-seq peak callers are not designed for CC insertion data
- Insertion records may require cleaning (clean_qbed) before peak calling; uncleaned qbed can degrade peak accuracy

## Evidence

- [intro] Preprocessing (pp) helps to preprocess data from qbed data to call peaks, make annotation, make Anndata object and filter peaks.: "**Preprocessing (pp)** helps to preprocess data from qbed data to call peaks, make annotation, make Anndata object and filter peaks."
- [abstract] Two innovative peak callers CCcaller and MACCs enhance accuracy and speed of pinpointing TF binding sites.: "Pycallingcards introduces two innovative peak callers, CCcaller and MACCs, enhancing the accuracy and speed of pinpointing TF binding sites from CC data."
- [methods] In order to complete the annotation, bedtools (includes pybedtools) also needs to be installed.: "In order to complete the annotation, [bedtools](https://bedtools.readthedocs.io/en/latest/) (includes [pybedtools](https://daler.github.io/pybedtools/)) also needs to be installed."
- [methods] Two Anndata objects can combine into one Mudata object.: "Two Anndata objects can combine into one Mudata object."
- [methods] The GBM CC dataset can be downloaded from GEO GSE148448.: "it can be downloaded from [GEO](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE148448)."
- [readme] README: Pycallingcards is composed of five different part: datasets, reading (rd), preprocessing (pp), tools (tl) and plotting (pl).: "Pycallingcards is composed of five different part: datasets, reading (rd), preprocessing (pp), tools (tl) and plotting (pl)."
