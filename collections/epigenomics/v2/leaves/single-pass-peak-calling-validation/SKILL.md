---
name: single-pass-peak-calling-validation
description: Use when a bulk Calling Cards qbed dataset (e.g. Brd4 K562 from GSE148448)
  needs TF binding site peak calling with a fast, deterministic greedy algorithm whose
  enrichment logic matches the MACCs method.
when_to_use_negative:
- Input is not raw insertion coordinates (e.g. already a called-peak BED or an Anndata
  object) — the greedy pass operates on per-insertion qbed data
- Analysis requires single-cell-aware peak calling or differential binding; CCcaller
  as described here is a bulk, per-chromosome greedy caller
- The user needs ChIP-seq-style signal profiles rather than insertion-enrichment peaks
  from Calling Cards data
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: pycallingcards
  role: Python package providing the datasets, qbed reading (rd), and preprocessing
    (pp) machinery in which CCcaller's greedy peak calling is implemented and executed
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: CCcaller
  role: the greedy single-pass peak caller that grows candidate peaks from sorted
    insertions and promotes them via Poisson enrichment over background insertions
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: MACCs
  role: the companion peak caller whose promotion logic (Poisson enrichment over background
    insertions) CCcaller reuses for candidate-to-CC-peak promotion
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: Python
  role: implementation language of the pycallingcards module and the greedy peak-calling
    workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae070
    title: 'Pycallingcards: an integrated environment for visualizing, analyzing,
      and interpreting Calling Cards data'
  source_content_hash: sha256:14cf4b35a29c84a5232a5bc404d4c5629d59679e0b17e5863b526d26ea17a96f
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
yes_rate: 0.7272727272727273
n_claims: 11
---

# single-pass-peak-calling-validation

## Summary

Implements the CCcaller greedy peak-calling algorithm from Pycallingcards: candidate peaks are grown in a single ordered pass over chromosome-sorted qbed insertions and promoted to CC peaks via Poisson enrichment testing over background insertions, providing a computationally efficient route to TF binding site identification.

## When to use

Use when a bulk Calling Cards qbed dataset (e.g. Brd4 K562 from GSE148448) needs TF binding site peak calling with a fast, deterministic greedy algorithm whose enrichment logic matches the MACCs method.

## When NOT to use

- Input is not raw insertion coordinates (e.g. already a called-peak BED or an Anndata object) — the greedy pass operates on per-insertion qbed data
- Analysis requires single-cell-aware peak calling or differential binding; CCcaller as described here is a bulk, per-chromosome greedy caller
- The user needs ChIP-seq-style signal profiles rather than insertion-enrichment peaks from Calling Cards data

## Inputs

- qbed-format insertion dataframe (chromosome, start, end, etc.) from Calling Cards sequencing data
- user-specified maximum distance (base pairs) between consecutive insertions for peak extension
- Poisson enrichment significance threshold for promoting candidate peaks

## Outputs

- BED file of promoted CC peaks with chromosome, start, end and peak statistics (e.g. cccaller_peaks.bed)
- peak set suitable for downstream annotation, motif calling, and differential peak analysis in pycallingcards

## How to apply

Load the Calling Cards insertions from a qbed-format dataframe (built-in datasets such as cc.datasets with Brd4 K562 data, originally GSE148448) using pycallingcards. Group insertions by chromosome and start a candidate peak at the first insertion on each chromosome. Greedily extend each candidate peak by appending the next nearest insertion while it falls within a user-specified maximum distance of the current peak boundary, traversing all insertions in a single ordered pass — this one-pass traversal is what makes CCcaller computationally efficient. For each candidate peak, compute a Poisson enrichment p-value of observed insertions against the background insertion rate and promote peaks passing the significance threshold using the same logic described for the MACCs method (preprocessing.call_peaks-style calling). Write promoted peaks (chromosome, start, end, statistics) to a BED file such as cccaller_peaks.bed and confirm the algorithm completed in a single pass over the sorted insertions.

## Related tools

- **pycallingcards** (Python package providing the datasets, qbed reading (rd), and preprocessing (pp) machinery in which CCcaller's greedy peak calling is implemented and executed) — https://github.com/The-Mitra-Lab/pycallingcards
- **CCcaller** (the greedy single-pass peak caller that grows candidate peaks from sorted insertions and promotes them via Poisson enrichment over background insertions) — https://github.com/The-Mitra-Lab/pycallingcards
- **MACCs** (the companion peak caller whose promotion logic (Poisson enrichment over background insertions) CCcaller reuses for candidate-to-CC-peak promotion) — https://github.com/The-Mitra-Lab/pycallingcards
- **Python** (implementation language of the pycallingcards module and the greedy peak-calling workflow)

## Examples

```
import pycallingcards as cc; qbed = cc.datasets.brd4_data(); peaks = cc.pp.call_peaks(qbed, refgen='hg38'); cc.rd.save_bed(peaks, 'cccaller_peaks.bed')
```

## Evaluation signals

- Every insertion on each chromosome is visited exactly once in a single ordered pass (efficiency invariant of the greedy algorithm)
- Candidate peaks only extend while the next nearest insertion is within the user-specified maximum base-pair distance
- Promoted CC peaks match the promotion outcome of the MACCs Poisson enrichment procedure (preprocessing.call_peaks-style logic)
- Output BED file has valid chromosome/start/end coordinates with statistics, and peaks are non-overlapping-consistent with greedy boundaries
- Called peaks can be carried through pycallingcards annotation (preprocessing.annotation) and filter_peaks without format errors

## Limitations

- Peak extension behavior depends on a user-specified maximum distance, so results are sensitive to that parameter choice
- CCcaller's promotion logic mirrors MACCs; accuracy gains of MACCs over simpler callers are not claimed for the greedy pass itself
- Only pycallingcards-provided bulk datasets (e.g. GSE148448 Brd4 K562) are documented; other species/platforms need compatible qbed input

## Evidence

- [other] CCcaller uses a greedy algorithm: it starts from the first insertion on each chromosome and extends the candidate peak while the next nearest insertion is within a user-specified number of base pairs: [evidence span withheld — anchor status: not_found]
- [abstract] Pycallingcards introduces two innovative peak callers, CCcaller and MACCs, enhancing the accuracy and speed of pinpointing TF binding sites from CC data.: "Pycallingcards introduces two innovative peak callers, CCcaller and MACCs, enhancing the accuracy and speed of pinpointing TF binding sites from CC data."
- [intro] **Preprocessing (pp)** helps to preprocess data from qbed data to call peaks, make annotation, make Anndata object and filter peaks.: "**Preprocessing (pp)** helps to preprocess data from qbed data to call peaks, make annotation, make Anndata object and filter peaks."
- [methods] it can be downloaded from [GEO](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE148448).: "it can be downloaded from [GEO](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE148448)."
