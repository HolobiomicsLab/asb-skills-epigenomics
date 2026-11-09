---
name: ttaa-density-relative-correction
description: Use when calling peaks from Calling Cards (qbed) insertion data with
  MACCs and computing Poisson p-values comparing experimental vs background insertions,
  to normalize for local TTAA-site density differences across candidate peaks.
when_to_use_negative:
- Data are not Calling Cards transposon insertions (e.g. ChIP-seq or ATAC-seq peaks),
  where TTAA-site integration bias does not apply
- Peak calling is being done with CCcaller rather than MACCs, or the analysis goal
  is downstream differential peak analysis rather than significance testing of candidate
  peaks
- The insertion data have not yet been scanned and merged into candidate peak regions
  by MACCs (apply the window scan first)
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: MACCs
  role: Peak caller whose Poisson significance test is TTAA-corrected; scans genome
    windows, merges enriched windows, and tests experimental vs background insertions
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: Pycallingcards
  role: Python module providing the MACCs implementation and qbed dataset loaders
    (preprocessing.call_peaks) used to run the corrected peak calling
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: Python
  role: Execution environment for running the Pycallingcards MACCs workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae070
    title: 'Pycallingcards: an integrated environment for visualizing, analyzing,
      and interpreting Calling Cards data'
  source_content_hash: sha256:3018f2ba2e4a2277aaa0888fe0b1ccb6b0488a62db2a2d3464ed41d934548b2e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/The-Mitra-Lab/pycallingcards
  tools:
  - MACCs
  - Pycallingcards
  - Python
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# ttaa-density-relative-correction

## Summary

A correction step in the MACCs peak-calling algorithm (Pycallingcards) that adjusts Poisson significance testing of Calling Cards insertions by the relative density of TTAA sites, accounting for the fact that the CC transposon integrates at TTAA sites and genomic regions differ in how many such target sites they contain.

## When to use

Use when calling peaks from Calling Cards (qbed) insertion data with MACCs and computing Poisson p-values comparing experimental vs background insertions, to normalize for local TTAA-site density differences across candidate peaks.

## When NOT to use

- Data are not Calling Cards transposon insertions (e.g. ChIP-seq or ATAC-seq peaks), where TTAA-site integration bias does not apply
- Peak calling is being done with CCcaller rather than MACCs, or the analysis goal is downstream differential peak analysis rather than significance testing of candidate peaks
- The insertion data have not yet been scanned and merged into candidate peak regions by MACCs (apply the window scan first)

## Inputs

- qbed insertion dataset from calling cards experiments (e.g. datasets.mouse_brd_data, mouse cortex BRD4 data)
- background control insertion dataset (when available)
- candidate peak regions from MACCs window scan (chromosome, start, end, center)
- reference genome TTAA-site locations/density

## Outputs

- TTAA-corrected Poisson p-values per candidate peak
- Significant TF binding site peaks in .bed format (chromosome, start, end, center, statistics), e.g. maccs_peaks.bed

## How to apply

After scanning the genome with MACCs using a fixed window and step size, merging consecutively enriched windows into candidate peaks, and finding each peak's center, perform a Poisson significance test comparing experimental insertions against background insertions for each candidate region. Before interpreting p-values, correct the test by the relative density of TTAA sites in each peak region, since Calling Cards insertions only occur at TTAA sites and regions richer in TTAA sites accumulate more insertions by chance. In Pycallingcards this correction is embedded in the pp.call_peaks MACCs workflow; write resulting peak coordinates and statistics (chromosome, start, end, center, p-value) to a .bed file such as maccs_peaks.bed. When no background control dataset exists, use the background-free lambda-window variant instead, and still apply the TTAA-corrected Poisson model.

## Related tools

- **MACCs** (Peak caller whose Poisson significance test is TTAA-corrected; scans genome windows, merges enriched windows, and tests experimental vs background insertions) — https://github.com/The-Mitra-Lab/pycallingcards
- **Pycallingcards** (Python module providing the MACCs implementation and qbed dataset loaders (preprocessing.call_peaks) used to run the corrected peak calling) — https://github.com/The-Mitra-Lab/pycallingcards
- **Python** (Execution environment for running the Pycallingcards MACCs workflow)

## Examples

```
import pycallingcards as cc; qbed = cc.datasets.mouse_brd_data(); peaks = cc.pp.call_peaks(qbed, method="MACCs", reference="mm10"); cc.rd.save_bed(peaks, "maccs_peaks.bed")
```

## Evaluation signals

- Each reported peak carries a Poisson p-value that reflects TTAA-site-corrected expected insertion counts, not raw counts alone
- Output .bed file contains the required columns: chromosome, start, end, center, and statistics
- Candidate peak regions correspond to consecutive enriched windows from the MACCs scan, with a defined center per peak
- With a background dataset supplied, experimental-vs-background enrichment per window precedes significance testing; without one, the lambda-window background-free variant is used instead
- Reanalysis of built-in datasets (e.g. datasets.mouse_brd_data) yields peaks consistent with known TF binding patterns reported in the Pycallingcards documentation

## Limitations

- The correction assumes Calling Cards insertions are restricted to TTAA sites; it does not apply to insertion technologies with different target-site preferences
- Without a matched background control dataset, the background-free lambda-window variant must be used, which is a fallback rather than the default model
- Results depend on the specified window and step size used during the MACCs genome scan

## Evidence

- [other] MACCs identifies candidate CC peaks by scanning the genome with a specified window and step size for regions enriched in experimental over background insertions, merging consecutively enriched windows, and finding the center of each candidate peak.: [evidence span withheld — anchor status: not_found]
- [abstract] Pycallingcards introduces two innovative peak callers, CCcaller and MACCs, enhancing the accuracy and speed of pinpointing TF binding sites from CC data.: "Pycallingcards introduces two innovative peak callers, CCcaller and MACCs, enhancing the accuracy and speed of pinpointing TF binding sites from CC data"
- [methods] **Preprocessing (pp)** helps to preprocess data from qbed/ccf data to call peaks, make annotation, make Anndata object and filter peaks.: "helps to preprocess data from qbed/ccf data to call peaks, make annotation, make Anndata object and filter peaks"
- [readme] **Pycallingcards** is a package for calling cards data analysis developed and maintained by [Mitra Lab](http://genetics.wustl.edu/rmlab/) at Washington University in St. Louis.: [evidence span withheld — anchor status: not_found]
