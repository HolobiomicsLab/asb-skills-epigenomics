---
name: runtime-memory-profiling
description: Use when comparing computational cost of peak callers on calling cards
  data, or when validating that CCcaller delivers ~7-10x speedup over MACCs/ccf_tools
  on >5M insertion datasets.
when_to_use_negative:
- The goal is accuracy of called peaks rather than computational cost — use peak quality/motif
  evaluation instead of timing benchmarks.
- The dataset has fewer than ~5M insertions, where the reported speedup envelope (7-10x
  over MACCs/ccf_tools) is not expected to be observable.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0769
- http://edamontology.org/topic_3168
tools:
- name: Pycallingcards
  role: loads qbed data and generates the down-sampling dilution series (rd.read_qbed,
    pp.down_sample) and runs CCcaller/MACCs via pp.call_peaks
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: CCcaller
  role: primary Pycallingcards peak caller being benchmarked for runtime and memory
    scaling
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: MACCs
  role: second Pycallingcards peak caller benchmarked as the slower in-package comparator
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: Python
  role: execution environment for timing loops, RSS capture, and matplotlib benchmark
    plotting
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae070
    title: 'Pycallingcards: an integrated environment for visualizing, analyzing,
      and interpreting Calling Cards data'
  source_content_hash: sha256:8ed630e85df70ef54b66f282210d6c72fec0c3505ea27140610c2ed1f4b42414
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
  - http://edamontology.org/topic_0769
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# runtime-memory-profiling

## Summary

Benchmarks CC peak callers (CCcaller, MACCs, Blockify, Blockify_original, ccf_tools) by measuring wall-clock runtime and peak memory (RSS) across a down-sampling series of qbed insertion counts, to characterize computational scaling as in Fig. 2f-g.

## When to use

Use when comparing computational cost of peak callers on calling cards data, or when validating that CCcaller delivers ~7-10x speedup over MACCs/ccf_tools on >5M insertion datasets.

## When NOT to use

- The goal is accuracy of called peaks rather than computational cost — use peak quality/motif evaluation instead of timing benchmarks.
- The dataset has fewer than ~5M insertions, where the reported speedup envelope (7-10x over MACCs/ccf_tools) is not expected to be observable.

## Inputs

- qbed file with >20M transposon insertions (e.g., PBase Brd4 K562 background-free) or bundled datasets (datasets.SP1_K562HCT116_data, datasets.mouse_brd4_data)
- down-sampled qbed subsets at decreasing total insertion counts from pp.down_sample

## Outputs

- runtime_memory_benchmark.csv with wall-clock runtime and peak memory (RSS) for each caller x insertion-count combination
- runtime vs. total insertion number plot and per-caller memory usage plot (Fig. 2f-g analog) via matplotlib/pycallingcards plotting

## How to apply

Load a large qbed (e.g., PBase Brd4 K562 background-free, >20M insertions, or SP1/Brd4 K562-HCT116 with-background data) via pycallingcards.reading.read_qbed or the bundled datasets.SP1_K562HCT116_data / datasets.mouse_brd4_data loaders. Generate a dilution series of decreasing insertion counts with pycallingcards.preprocessing.down_sample so scaling versus total insertion number can be measured. Run CCcaller and MACCs through pycallingcards.preprocessing.call_peaks on every downsampled subset, timing each run, and run the external baselines (Blockify, Blockify_original, ccf_tools) on identical subsets under identical conditions. Record wall-clock runtime and peak RSS for every caller/insertion-count combination (e.g., into runtime_memory_benchmark.csv), then plot runtime versus total insertions and memory per caller as a Fig. 2f-g analog.

## Related tools

- **Pycallingcards** (loads qbed data and generates the down-sampling dilution series (rd.read_qbed, pp.down_sample) and runs CCcaller/MACCs via pp.call_peaks) — https://github.com/The-Mitra-Lab/pycallingcards
- **CCcaller** (primary Pycallingcards peak caller being benchmarked for runtime and memory scaling) — https://github.com/The-Mitra-Lab/pycallingcards
- **MACCs** (second Pycallingcards peak caller benchmarked as the slower in-package comparator) — https://github.com/The-Mitra-Lab/pycallingcards
- **Python** (execution environment for timing loops, RSS capture, and matplotlib benchmark plotting)

## Examples

```
import pycallingcards as cc; qbed = cc.datasets.SP1_K562HCT116_data(); ds = cc.pp.down_sample(qbed); peaks = cc.pp.call_peaks(ds, method='CCcaller')
```

## Evaluation signals

- CCcaller completes in <5 minutes on datasets with >5M insertions
- CCcaller shows ~7-10x speedup over MACCs and ccf_tools on large datasets
- CCcaller shows ~2- to 3-fold speedup over Blockify on >5M insertion datasets
- Every caller/insertion-count combination in the dilution series has a recorded runtime and peak RSS row in runtime_memory_benchmark.csv
- Runtime curves for each caller are monotonic (or near-monotonic) with total insertion number, consistent with the reported Fig. 2f-g performance envelope

## Limitations

- External baselines (Blockify, Blockify_original, ccf_tools) must run on identical subsets under identical conditions, or comparisons are invalid.
- Speedup ratios are calibrated for datasets with more than five million insertions; scaling relationships may differ on smaller datasets.

## Evidence

- [results] CCcaller runs significantly faster than the other peak calling methods, running 7–10× faster than MACCs and ccf_tools and 2- to 3-fold faster than Blockify when processing datasets with more than five million insertions.: [evidence span withheld — anchor status: not_found]
- [methods] Generate a dilution series of decreasing insertion counts from each qbed with pycallingcards.preprocessing.down_sample so that scaling versus total insertion number can be measured.: "preprocessing.down_sample"
- [methods] down_sample  [section=methods; evidence='preprocessing.down_sample']: "Pycallingcards is composed of five different part: datasets, reading (rd), preprocessing (pp), tools (tl) and plotting (pl)."
- [readme] pip install "git+https://github.com/The-Mitra-Lab/pycallingcards.git" --upgrade: "pip install "git+https://github.com/The-Mitra-Lab/pycallingcards.git" --upgrade"
