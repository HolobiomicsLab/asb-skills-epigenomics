---
name: nearest-gene-peak-annotation
description: Use when you have called Brd4/TF binding peaks (e.g., ~902 peaks from
  the mouse cortex scCC dataset) and need to assign nearest genes so peaks can later
  be paired with gene expression via cc.tl.pair_peak_gene_sc.
when_to_use_negative:
- Input is raw qbed/ccf insertion data that has not yet been through peak calling
  with cc.pp.call_peaks
- You need binding-expression statistics but have no annotated peaks — annotation
  alone does not produce peak-gene correlations
- Analysis targets ChIP-seq or other peak types outside the Pycallingcards qbed/CC
  data model
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0204
- http://edamontology.org/topic_3168
tools:
- name: pycallingcards
  role: Provides cc.pp.annotation to attach nearest genes to called peaks and cc.pp.make_Anndata
    to build the cell-by-peak matrix
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: bedtools
  role: Backend (with pybedtools) used by the annotation step to compute nearest genes
    from genome annotation
  repo: https://bedtools.readthedocs.io/en/latest/
- name: Mudata
  role: Combines the annotated peak AnnData with the gene-expression AnnData so peak-gene
    pairing can be run
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae070
    title: 'Pycallingcards: an integrated environment for visualizing, analyzing,
      and interpreting Calling Cards data'
  source_content_hash: sha256:65fe7c8fc7e99ea63b00e86d23eea55b646659c7fc693dfbdd4933ac29a7dec3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/The-Mitra-Lab/pycallingcards
  tools:
  - Mudata
  - Scanpy
  - bedtools
  - pycallingcards
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0204
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# nearest-gene-peak-annotation

## Summary

Annotates called Calling Cards (qbed-derived) peaks with their nearest genes using Pycallingcards' preprocessing.annotation step, which is backed by bedtools/pybedtools. Nearest-gene annotation is the prerequisite for pairing cell-type-specific TF binding peaks with nearby gene expression.

## When to use

Use when you have called Brd4/TF binding peaks (e.g., ~902 peaks from the mouse cortex scCC dataset) and need to assign nearest genes so peaks can later be paired with gene expression via cc.tl.pair_peak_gene_sc.

## When NOT to use

- Input is raw qbed/ccf insertion data that has not yet been through peak calling with cc.pp.call_peaks
- You need binding-expression statistics but have no annotated peaks — annotation alone does not produce peak-gene correlations
- Analysis targets ChIP-seq or other peak types outside the Pycallingcards qbed/CC data model

## Inputs

- Called peak data (bed-like dataframe/BED from cc.pp.call_peaks on qbed insertions, e.g., 902 Brd4 peaks from GSE148448 mouse cortex)
- Genome reference annotation (e.g., mouse mm10 gene annotation) used by bedtools/pybedtools for nearest-gene assignment

## Outputs

- Peak table annotated with nearest genes (e.g., two nearest genes per peak)
- Annotated peaks suitable for cc.pp.make_Anndata and downstream MuData peak-gene pairing

## How to apply

After calling peaks from qbed insertions with CCcaller via cc.pp.call_peaks, run cc.pp.annotation on the peak dataframe against a matched genome reference to attach nearest genes (in the mouse cortex reanalysis each peak was annotated with its two nearest genes, e.g., Mef2c for peak chr13:83141353-83148478). The annotation step relies on bedtools/pybedtools being installed, so the environment must include them. Annotated peaks then feed into cc.pp.make_Anndata to build a cell-by-peak AnnData object, which can be combined with the scRNA-seq AnnData into a MuData object so differential peak analysis (cc.tl.rank_peak_groups) and peak-gene pairing (cc.tl.pair_peak_gene_sc) can recover binding-expression relationships. Correctness is judged by whether reported peaks recover their expected neighboring genes (e.g., E130114P18Rik near chr4:97575305-97588788).

## Related tools

- **pycallingcards** (Provides cc.pp.annotation to attach nearest genes to called peaks and cc.pp.make_Anndata to build the cell-by-peak matrix) — https://github.com/The-Mitra-Lab/pycallingcards
- **bedtools** (Backend (with pybedtools) used by the annotation step to compute nearest genes from genome annotation) — https://bedtools.readthedocs.io/en/latest/
- **Mudata** (Combines the annotated peak AnnData with the gene-expression AnnData so peak-gene pairing can be run)

## Examples

```
import pycallingcards as cc; peaks = cc.pp.annotation(peaks, reference='mm10')
```

## Evaluation signals

- Annotated peak count matches the expected called set (e.g., ~902 peaks for the mouse cortex scCC reanalysis)
- Known peaks recover their expected neighboring genes: Mef2c for chr13:83141353-83148478 and E130114P18Rik for chr4:97575305-97588788
- Peak annotation output is a valid input to cc.pp.make_Anndata (cell-by-peak AnnData builds without error)
- Downstream cc.tl.pair_peak_gene_sc returns the reported cell-type-specific pairs (Mef2c in excitatory neurons, E130114P18Rik in astrocytes/neuroblasts)

## Limitations

- Nearest-gene assignment assumes the closest gene is the regulatory target, which can miss distal enhancer-like relationships; correlations must be validated with pair_peak_gene_sc
- bedtools/pybedtools is a hard dependency — annotation fails if it is not installed
- Annotation is reference-genome specific; mismatched reference annotation will produce wrong gene assignments

## Evidence

- [methods] In order to complete the annotation, bedtools (includes pybedtools) also needs to be installed.: [evidence span withheld — anchor status: not_found]
- [readme] Preprocessing (pp) helps to preprocess data from qbed data to call peaks, make annotation, make Anndata object and filter peaks.: "**Preprocessing (pp)** helps to preprocess data from qbed data to call peaks, make annotation, make Anndata object and filter peaks."
- [methods] Tools (tl) calls motif of the peaks, completes differential peaks and pair differential peaks with gene expression,: "**Tools (tl)** calls motif of the peaks, completes differential peaks and pair differential peaks with gene expression,"
- [methods] pair analysis is completed to find out potential hypothesis for the relationship between transcription factor bindings and gene expression: "pair analysis is completed to find out potential hypothesis for the relationship between transcription factor bindings and gene expression"
