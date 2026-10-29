---
name: poisson-enrichment-peak-testing
description: Use when Calling Cards qbed/ccf insertions must be tested for significant
  TF binding enrichment — e.g. after greedy candidate-peak extension, to decide which
  candidates become CC peaks.
when_to_use_negative:
- Input is already a called peak set — use downstream steps like preprocessing.annotation
  or preprocessing.filter_peaks instead
- The goal is merging or splitting existing peaks rather than testing candidate peaks
  for significance — preprocessing.combine_peaks or preprocessing.separate_peaks is
  the appropriate operation
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: pycallingcards
  role: Python module providing the preprocessing (pp) layer — including call_peaks
    — that performs Poisson enrichment testing of candidate peaks from qbed data
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: CCcaller
  role: Greedy peak caller that builds candidate peaks in a single pass over sorted
    insertions, then promotes them to CC peaks via Poisson enrichment as in the MACCs
    method
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: MACCs
  role: Innovative peak caller whose candidate-peak promotion logic (Poisson enrichment
    over background insertions) is reused for CCcaller peaks
  repo: https://github.com/The-Mitra-Lab/pycallingcards
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae070
    title: 'Pycallingcards: an integrated environment for visualizing, analyzing,
      and interpreting Calling Cards data'
  source_content_hash: sha256:da600f1d70539fcd2c81879852f325a4405aac701d5f0efc48f71bee5b66ad9e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/The-Mitra-Lab/pycallingcards
  tools:
  - CCcaller
  - MACCs
  - Python
  - pycallingcards
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# poisson-enrichment-peak-testing

## Summary

A peak-calling significance-testing skill from Pycallingcards in which candidate TF binding peaks assembled from Calling Cards insertions (via the CCcaller greedy pass or MACCs) are promoted to bona fide peaks using Poisson enrichment p-values of observed insertions against the background insertion rate.

## When to use

Use when Calling Cards qbed/ccf insertions must be tested for significant TF binding enrichment — e.g. after greedy candidate-peak extension, to decide which candidates become CC peaks.

## When NOT to use

- Input is already a called peak set — use downstream steps like preprocessing.annotation or preprocessing.filter_peaks instead
- The goal is merging or splitting existing peaks rather than testing candidate peaks for significance — preprocessing.combine_peaks or preprocessing.separate_peaks is the appropriate operation

## Inputs

- qbed or ccf Calling Cards insertion files (e.g. Brd4 K562 qbed data from GEO GSE148448)
- user-specified maximum insertion-to-peak distance in base pairs (CCcaller greedy extension)
- Poisson enrichment significance threshold for peak promotion
- background insertion rate estimate

## Outputs

- Promoted CC peaks with statistics in BED format (e.g. cccaller_peaks.bed with chromosome, start, end, statistics)
- Peak objects consumable by pycallingcards preprocessing steps (annotation, make_Anndata, filter_peaks)

## How to apply

Load insertions from qbed/ccf data (e.g. the built-in Brd4 K562 dataset from GEO GSE148448 via cc.datasets) and, for CCcaller, start a candidate peak at the first insertion on each chromosome and greedily extend it while the next nearest insertion falls within the user-specified maximum base-pair distance — a single ordered pass over all insertions. For MACCs-style calling, group nearby insertions into candidate peaks by windowing. For each candidate peak, compute the Poisson enrichment p-value of the observed insertion count against the background insertion rate and promote peaks passing the significance threshold using pycallingcards' preprocessing.call_peaks logic. Write promoted peaks (chromosome, start, end, statistics) to a BED file such as cccaller_peaks.bed; the single-pass design makes CCcaller computationally efficient because every insertion is only considered once.

## Related tools

- **pycallingcards** (Python module providing the preprocessing (pp) layer — including call_peaks — that performs Poisson enrichment testing of candidate peaks from qbed data) — https://github.com/The-Mitra-Lab/pycallingcards
- **CCcaller** (Greedy peak caller that builds candidate peaks in a single pass over sorted insertions, then promotes them to CC peaks via Poisson enrichment as in the MACCs method) — https://github.com/The-Mitra-Lab/pycallingcards
- **MACCs** (Innovative peak caller whose candidate-peak promotion logic (Poisson enrichment over background insertions) is reused for CCcaller peaks) — https://github.com/The-Mitra-Lab/pycallingcards

## Examples

```
import pycallingcards as cc; qbed = cc.datasets.Brd4_K562_data(); peaks = cc.pp.call_peaks(qbed, method='cccaller', reference='hg38'); cc.rd.save_peak(peaks, filename='cccaller_peaks.bed')
```

## Evaluation signals

- Promoted peaks file (e.g. cccaller_peaks.bed) exists with chromosome, start, end and statistics columns
- Each promoted peak has a Poisson enrichment p-value below the significance threshold over the background insertion rate
- CCcaller completed in a single ordered pass over sorted insertions — no insertion visited more than once
- Every CC peak boundary satisfies the greedy extension rule: the next nearest insertion beyond it exceeds the user-specified maximum distance
- Peak set is consistent when re-run through preprocessing.call_peaks on the same qbed input

## Limitations

- Peak promotion is exactly as described for the MACCs method, so quality depends on a valid background insertion rate; mis-specified background can shift significance
- Candidate peak extent is bounded by the user-specified maximum insertion distance, which must be chosen per dataset
- Annotation downstream additionally requires bedtools/pybedtools to be installed

## Evidence

- [other] CCcaller uses a greedy algorithm: it starts from the first insertion on each chromosome and extends the candidate peak while the next nearest insertion is within a user-specified number of base pairs: [evidence span withheld — anchor status: not_found]
- [other] candidate peaks are then promoted to CC peaks exactly as described for the MACCs method (Poisson enrichment over background insertions): [evidence span withheld — anchor status: not_found]
- [readme] **Preprocessing (pp)** helps to preprocess data from qbed data to call peaks, make annotation, make Anndata object and filter peaks.: "**Preprocessing (pp)** helps to preprocess data from qbed data to call peaks, make annotation, make Anndata object and filter peaks."
- [abstract] Pycallingcards introduces two innovative peak callers, CCcaller and MACCs, enhancing the accuracy and speed of pinpointing TF binding sites from CC data.: "Pycallingcards introduces two innovative peak callers, CCcaller and MACCs, enhancing the accuracy and speed of pinpointing TF binding sites from CC data."
