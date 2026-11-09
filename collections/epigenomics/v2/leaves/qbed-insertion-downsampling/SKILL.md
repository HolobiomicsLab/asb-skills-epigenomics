---
name: qbed-insertion-downsampling
description: Use when a qbed file (e.g. >20M PBase Brd4 K562 insertions) must be subsampled
  to decreasing insertion counts to measure how peak-caller runtime and memory scale
  with dataset size.
when_to_use_negative:
- Input is already a peak set or Anndata object rather than a raw qbed of insertions
- The goal is peak-accuracy (binding-site sensitivity/specificity) comparison rather
  than runtime/memory scaling, since downsampling changes effective depth
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0769
tools:
- name: Pycallingcards
  role: Provides reading.read_qbed, preprocessing.down_sample, and preprocessing.call_peaks
    used to load, dilute, and peak-call the qbed subsets
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: CCcaller
  role: Pycallingcards peak caller benchmarked on each downsampled subset; expected
    to run 7-10x faster than MACCs/ccf_tools and 2-3x faster than Blockify
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: MACCs
  role: Pycallingcards peak caller benchmarked on each downsampled subset as the slower
    baseline
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: Python
  role: Execution environment in which timing, RSS capture, CSV logging, and matplotlib
    plotting are scripted
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae070
    title: 'Pycallingcards: an integrated environment for visualizing, analyzing,
      and interpreting Calling Cards data'
  source_content_hash: sha256:8a98ad6d295ce670a2db5436eff4f030716c1e2810b4884a2b8dc209cd55182f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/The-Mitra-Lab/pycallingcards
  tools:
  - CCcaller
  - MACCs
  - Pycallingcards
  - Python
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# qbed-insertion-downsampling

## Summary

Generates dilution series of calling cards qbed datasets with pycallingcards.preprocessing.down_sample so that caller runtime and memory scaling versus total transposon insertion number can be benchmarked across CCcaller, MACCs, Blockify, Blockify_original, and ccf_tools.

## When to use

Use when a qbed file (e.g. >20M PBase Brd4 K562 insertions) must be subsampled to decreasing insertion counts to measure how peak-caller runtime and memory scale with dataset size.

## When NOT to use

- Input is already a peak set or Anndata object rather than a raw qbed of insertions
- The goal is peak-accuracy (binding-site sensitivity/specificity) comparison rather than runtime/memory scaling, since downsampling changes effective depth

## Inputs

- Bulk qbed file of transposon insertions (e.g. PBase Brd4 K562 background-free qbed, >20M insertions)
- SP1/Brd4 K562-HCT116 with-background qbed, or bundled pycallingcards datasets (datasets.SP1_K562HCT116_data, datasets.mouse_brd4_data)

## Outputs

- Dilution series of downsampled qbed insertion subsets at decreasing total insertion counts
- runtime_memory_benchmark.csv with wall-clock runtime and peak memory (RSS) for every caller/insertion-count combination
- Runtime-versus-insertion-count and memory-usage plots per caller (Fig. 2f-g analog)

## How to apply

Load a bulk qbed (e.g. PBase Brd4 K562 background-free qbed with >20M insertions, or the bundled datasets.SP1_K562HCT116_data / datasets.mouse_brd4_data) with pycallingcards.reading.read_qbed, then apply pycallingcards.preprocessing.down_sample to generate a series of subsets with decreasing total insertion counts. Run each peak caller (CCcaller and MACCs via pycallingcards.preprocessing.call_peaks, plus external Blockify, Blockify_original, and ccf_tools) on the identical downsampled subsets under identical conditions, timing each run. Record wall-clock runtime and peak memory (RSS) per caller/insertion-count combination into runtime_memory_benchmark.csv and plot runtime versus total insertions (Fig. 2f-g analog). The rationale is that downsampling isolates the effect of dataset size on caller performance while holding data content constant.

## Related tools

- **Pycallingcards** (Provides reading.read_qbed, preprocessing.down_sample, and preprocessing.call_peaks used to load, dilute, and peak-call the qbed subsets) — https://github.com/The-Mitra-Lab/pycallingcards
- **CCcaller** (Pycallingcards peak caller benchmarked on each downsampled subset; expected to run 7-10x faster than MACCs/ccf_tools and 2-3x faster than Blockify) — https://github.com/The-Mitra-Lab/pycallingcards
- **MACCs** (Pycallingcards peak caller benchmarked on each downsampled subset as the slower baseline) — https://github.com/The-Mitra-Lab/pycallingcards
- **Python** (Execution environment in which timing, RSS capture, CSV logging, and matplotlib plotting are scripted)

## Examples

```
import pycallingcards as cc; qbed = cc.rd.read_qbed('Brd4_K562.qbed'); subsets = [cc.pp.down_sample(qbed, n) for n in insertion_counts]
```

## Evaluation signals

- CCcaller completes in <5 minutes on subsets with >5M insertions
- CCcaller shows ~7-10x speedup over MACCs and ccf_tools and ~2-3x over Blockify on the large-insertion subsets, matching the reported Fig. 2f-g performance envelope
- Downsampled subsets show a monotonic decreasing total insertion count series derived from the original >20M-insertion qbed
- runtime_memory_benchmark.csv contains one runtime and peak-memory (RSS) record for every caller x insertion-count combination, including all five callers on identical subsets

## Limitations

- Runtime/memory conclusions are specific to the benchmarked datasets (e.g. >20M-insertion Brd4 K562 qbed) and hardware; smaller datasets may not resolve the 7-10x speedup regime
- With-background and background-free qbeds (SP1/Brd4 K562-HCT116 vs PBase Brd4 K562) may behave differently, so caller choice of input affects scaling curves
- External baselines (Blockify, Blockify_original, ccf_tools) must be run under identical conditions to the Pycallingcards callers for the comparison to be valid

## Evidence

- [other] Load the PBase Brd4 K562 background-free qbed (>20M insertions): [evidence span withheld — anchor status: not_found]
- [other] down_sample workflow step: [evidence span withheld — anchor status: not_found]
- [other] CCcaller speed finding: [evidence span withheld — anchor status: not_found]
- [intro] preprocessing module description: "**Preprocessing (pp)** helps to preprocess data from qbed data to call peaks, make annotation, make Anndata object and filter peaks."
- [readme] README package description: "Pycallingcards is composed of five different part: datasets, reading (rd), preprocessing (pp), tools (tl) and plotting (pl)."
