---
name: bulk-vs-single-cell-qbed-preprocessing
description: Use when starting from raw bulk or single-cell calling cards qbed/ccf
  files (e.g. Brd4 bulk GSE156821 or SP1 scCC HCT116/K562 data) and you need cleaned
  insertions, called peaks, and a filtered Anndata object before downstream motif
  or differential analysis.
when_to_use_negative:
- Input is already a called, annotated peak table or Anndata object — jump to pp.tools
  (motif calling, differential peaks) or plotting instead
- Data are not calling cards insertions (e.g. ChIP-seq peaks or RNA-seq counts); this
  skill's qbed-specific cleaning and peak calling does not apply
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0769
tools:
- name: Pycallingcards
  role: Python package providing the pp preprocessing module (clean_qbed, call_peaks,
    annotation, make_Anndata, filter_peaks) for qbed data
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: CCcaller
  role: Peak caller available in pp.call_peaks for pinpointing TF binding sites from
    cleaned insertions
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: MACCs
  role: Alternative peak caller available in pp.call_peaks, benchmarked alongside
    CCcaller
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: bedtools
  role: Required dependency (with pybedtools) for the pp.annotation step of the preprocessed
    peaks
  repo: https://bedtools.readthedocs.io/en/latest/
- name: Scanpy
  role: Interoperable single-cell framework; Pycallingcards' Annadata output and structure
    follow Scanpy conventions for scCC analysis
  repo: https://scanpy.readthedocs.io/en/stable/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae070
    title: 'Pycallingcards: an integrated environment for visualizing, analyzing,
      and interpreting Calling Cards data'
  source_content_hash: sha256:3bf94ee3e891b7a4f8b4e905d6c83bb5d9c74200ba2c087b3adc4a7e7dff4bc6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/The-Mitra-Lab/pycallingcards
  tools:
  - CCcaller
  - MACCs
  - Pycallingcards
  - Python
  - Scanpy
  - bedtools
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# bulk-vs-single-cell-qbed-preprocessing

## Summary

A Pycallingcards-based preprocessing skill for cleaning and peak-calling both bulk and single-cell calling cards qbed data, using the pp (preprocessing) module's clean_qbed, call_peaks, annotation, make_Anndata, and filter_peaks functions.

## When to use

Use when starting from raw bulk or single-cell calling cards qbed/ccf files (e.g. Brd4 bulk GSE156821 or SP1 scCC HCT116/K562 data) and you need cleaned insertions, called peaks, and a filtered Anndata object before downstream motif or differential analysis.

## When NOT to use

- Input is already a called, annotated peak table or Anndata object — jump to pp.tools (motif calling, differential peaks) or plotting instead
- Data are not calling cards insertions (e.g. ChIP-seq peaks or RNA-seq counts); this skill's qbed-specific cleaning and peak calling does not apply

## Inputs

- qbed insertion files (bulk and single-cell calling cards), e.g. from GEO accession GSE156821
- Built-in datasets such as datasets.SP1_K562HCT116_data (SP1 single-cell K562/HCT116 qbed)
- Optional peak background qbed for with-background peak calling
- Genome annotation files used by pp.annotation (requires bedtools/pybedtools)

## Outputs

- Cleaned qbed insertion tables
- Called peak BED-format tables per caller (CCcaller / MACCs)
- Annotated peak tables with nearest genes/features
- Annadata (AnnData) objects of peaks/insertions, optionally peak-filtered

## How to apply

Load qbed data either from GEO accessions (e.g. GSE156821 for bulk Brd4) or from built-in datasets such as pycallingcards.datasets.SP1_K562HCT116_data, then clean insertions with pycallingcards.preprocessing.clean_qbed to remove low-quality/multi-mapping insertions. Call peaks with pycallingcards.preprocessing.call_peaks, choosing the caller (CCcaller or MACCs, the package's two innovative peak callers) and whether to use a background model — bulk no-background and single-cell with-background settings are the two benchmark configurations. Annotate the resulting peaks via pp.annotation (which requires bedtools/pybedtools installed) and build an Annadata object with pp.make_Anndata; for single-cell data combine insertion matrices with Scanpy-compatible AnnData objects. Optionally filter peaks with pp.filter_peaks to retain statistically supported binding sites before differential peak analysis or comparison against ChIP-seq signal.

## Related tools

- **Pycallingcards** (Python package providing the pp preprocessing module (clean_qbed, call_peaks, annotation, make_Anndata, filter_peaks) for qbed data) — https://github.com/The-Mitra-Lab/pycallingcards
- **CCcaller** (Peak caller available in pp.call_peaks for pinpointing TF binding sites from cleaned insertions) — https://github.com/The-Mitra-Lab/pycallingcards
- **MACCs** (Alternative peak caller available in pp.call_peaks, benchmarked alongside CCcaller) — https://github.com/The-Mitra-Lab/pycallingcards
- **bedtools** (Required dependency (with pybedtools) for the pp.annotation step of the preprocessed peaks) — https://bedtools.readthedocs.io/en/latest/
- **Scanpy** (Interoperable single-cell framework; Pycallingcards' Annadata output and structure follow Scanpy conventions for scCC analysis) — https://scanpy.readthedocs.io/en/stable/

## Examples

```
import pycallingcards as cc; qbed = cc.datasets.SP1_K562HCT116_data(); qbed = cc.pp.clean_qbed(qbed); peaks = cc.pp.call_peaks(qbed, method='CCcaller')
```

## Evaluation signals

- clean_qbed reduces the insertion count relative to the raw qbed file while retaining valid CC insertions
- call_peaks with each caller yields a peak table per dataset (bulk no-background and single-cell with-background), with peaks in BED-like coordinate form
- The Annadata object from pp.make_Anndata contains insertion counts per peak/cell and survives pp.filter_peaks without dropping the intended peak set
- Downstream average ChIP-seq signal under peaks (e.g. via plotting.calculate_signal) is computable, and pairwise peak-overlap percentages between callers reproduce the expected concordance ranking (CCcaller ~ MACCs most concordant)

## Limitations

- bedtools/pybedtools must be installed for annotation, or the pp pipeline breaks
- Caller choice and background setting materially change peak sets — ccf_tools, Blockify, and Blockify_original produce more discordant peaks than CCcaller/MACCs, so cross-caller comparisons must state settings
- Pycallingcards supports calling cards qbed/ccf data specifically (bulk and single-cell); it is not a general ChIP-seq peak caller

## Evidence

- [intro] Preprocessing (pp) helps to preprocess data from qbed data to call peaks, make annotation, make Anndata object and filter peaks.: "**Preprocessing (pp)** helps to preprocess data from qbed data to call peaks, make annotation, make Anndata object and filter peaks."
- [abstract] Pycallingcards introduces two innovative peak callers, CCcaller and MACCs, enhancing the accuracy and speed of pinpointing TF binding sites from CC data.: "Pycallingcards introduces two innovative peak callers, CCcaller and MACCs, enhancing the accuracy and speed of pinpointing TF binding sites from CC data."
- [methods] In order to complete the annotation, bedtools (includes pybedtools) also needs to be installed.: "In order to complete the annotation, [bedtools](https://bedtools.readthedocs.io/en/latest/) (includes [pybedtools](https://daler.github.io/pybedtools/)) also needs to be installed."
- [methods] it can be downloaded from [GEO](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE156821).: "it can be downloaded from [GEO](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE156821)."
- [readme] For single-cell calling cards anaysis, Pycallingcards interacts with Scanpy and the main structure of Pycallingcards also follows the Scanpy.: "For single-cell calling cards anaysis, Pycallingcards interacts with [Scanpy](https://scanpy.readthedocs.io/en/stable/) and the main structure of Pycallingcards also follows the Scanpy."
