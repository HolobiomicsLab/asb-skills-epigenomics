---
name: cross-tool-performance-comparison
description: Use when choosing a CC peak caller for large calling cards datasets,
  or when reproducing Fig. 2f-g style runtime/memory scaling benchmarks across tools
  on downsampled qbed data.
when_to_use_negative:
- The goal is peak-calling accuracy or binding-site quality rather than computational
  performance comparison.
- The dataset is a small single-cell qbed where all callers finish quickly and runtime
  differences are not informative.
- Only one peak caller is available or licensed, so no cross-tool comparison can be
  made on identical subsets.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0769
- http://edamontology.org/topic_3168
tools:
- name: Pycallingcards
  role: Provides read_qbed, preprocessing.down_sample, preprocessing.call_peaks, and
    plotting used to build the dilution series and benchmark CCcaller and MACCs
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: CCcaller
  role: Pycallingcards peak caller being benchmarked; the fastest method, evaluated
    via pp.call_peaks
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: MACCs
  role: Pycallingcards peak caller being benchmarked; slower baseline evaluated via
    pp.call_peaks
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: Blockify
  role: External baseline peak caller run on identical downsampled subsets under identical
    conditions
- name: ccf_tools
  role: External baseline peak caller run on identical downsampled subsets under identical
    conditions
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae070
    title: 'Pycallingcards: an integrated environment for visualizing, analyzing,
      and interpreting Calling Cards data'
  source_content_hash: sha256:6fd7852aa0b1863606cc2d3a08e8b5fcd466b1b8f3419e45f11c40aa04fbf97e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/The-Mitra-Lab/pycallingcards
  tools:
  - Blockify
  - CCcaller
  - MACCs
  - Pycallingcards
  - Python
  - ccf_tools
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0769
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# cross-tool-performance-comparison

## Summary

Benchmark wall-clock runtime and peak memory (RSS) of multiple peak callers — CCcaller, MACCs, Blockify, Blockify_original, and ccf_tools — across a dilution series of transposon insertion counts, quantifying how each method's resource use scales with dataset size.

## When to use

Use when choosing a CC peak caller for large calling cards datasets, or when reproducing Fig. 2f-g style runtime/memory scaling benchmarks across tools on downsampled qbed data.

## When NOT to use

- The goal is peak-calling accuracy or binding-site quality rather than computational performance comparison.
- The dataset is a small single-cell qbed where all callers finish quickly and runtime differences are not informative.
- Only one peak caller is available or licensed, so no cross-tool comparison can be made on identical subsets.

## Inputs

- qbed file of transposon insertions (e.g., PBase Brd4 K562 background-free, >20M insertions)
- with-background qbed (SP1/Brd4 K562-HCT116) or bundled datasets.SP1_K562HCT116_data / datasets.mouse_brd4_data
- identical downsampled insertion subsets shared across all five callers

## Outputs

- runtime_memory_benchmark.csv with wall-clock runtime and peak memory (RSS) per caller per insertion count
- runtime vs. total insertion number and memory usage plots per caller (Fig. 2f-g analog)

## How to apply

Load a large qbed (e.g., the PBase Brd4 K562 background-free set with >20M insertions, or the SP1/Brd4 K562-HCT116 with-background set) via pycallingcards.reading.read_qbed or the bundled loaders (datasets.SP1_K562HCT116_data, datasets.mouse_brd4_data). Generate a dilution series of decreasing insertion counts with pycallingcards.preprocessing.down_sample so scaling versus total insertion number can be measured on identical subsets. Run CCcaller and MACCs through pycallingcards.preprocessing.call_peaks on every downsampled subset, and run the external baselines Blockify, Blockify_original, and ccf_tools on the identical subsets under identical conditions, timing each run and recording peak RSS. Write all caller/insertion-count combinations to runtime_memory_benchmark.csv and plot runtime and memory versus total insertion number per caller with matplotlib/pycallingcards plotting to reproduce the Fig. 2f-g analog.

## Related tools

- **Pycallingcards** (Provides read_qbed, preprocessing.down_sample, preprocessing.call_peaks, and plotting used to build the dilution series and benchmark CCcaller and MACCs) — https://github.com/The-Mitra-Lab/pycallingcards
- **CCcaller** (Pycallingcards peak caller being benchmarked; the fastest method, evaluated via pp.call_peaks) — https://github.com/The-Mitra-Lab/pycallingcards
- **MACCs** (Pycallingcards peak caller being benchmarked; slower baseline evaluated via pp.call_peaks) — https://github.com/The-Mitra-Lab/pycallingcards
- **Blockify** (External baseline peak caller run on identical downsampled subsets under identical conditions)
- **ccf_tools** (External baseline peak caller run on identical downsampled subsets under identical conditions)

## Examples

```
import pycallingcards as cc; qbed = cc.datasets.SP1_K562HCT116_data(); subs = cc.pp.down_sample(qbed); t0=time(); peaks = cc.pp.call_peaks(subs, method='CCcaller'); print(time()-t0)
```

## Evaluation signals

- CCcaller completes in <5 minutes on datasets with >5 million insertions.
- CCcaller shows ~7-10x speedup over MACCs and ccf_tools on >5M-insertion datasets.
- CCcaller shows ~2- to 3-fold faster runtime than Blockify on >5M-insertion datasets.
- runtime_memory_benchmark.csv contains a runtime and peak-RSS entry for every caller x insertion-count combination.
- All five callers were run on identical downsampled subsets under identical conditions (same input subsets, same machine).

## Limitations

- Speed comparisons are only informative for large datasets (>5M insertions); on small inputs differences may not reproduce the reported envelope.
- The benchmark requires the external callers (Blockify, Blockify_original, ccf_tools) to be installed and runnable on the same subsets; ccf_tools/Blockify read qbed/ccf formats so the reading (rd) module's qbed/ccf support is needed.
- External baseline callers are not part of the pip-installable pycallingcards package and may require separate environments or the pycallingcards docker image.

## Evidence

- [results] CCcaller runs significantly faster than the other peak calling methods, running 7–10× faster than MACCs and ccf_tools and 2- to 3-fold faster than Blockify when processing datasets with more than five million insertions.: [evidence span withheld — anchor status: not_found]
- [abstract] Pycallingcards introduces two innovative peak callers, CCcaller and MACCs, enhancing the accuracy and speed of pinpointing TF binding sites from CC data.: "Pycallingcards introduces two innovative peak callers, CCcaller and MACCs, enhancing the accuracy and speed of pinpointing TF binding sites from CC data."
- [intro] Preprocessing (pp) helps to preprocess data from qbed data to call peaks, make annotation, make Anndata object and filter peaks.: "**Preprocessing (pp)** helps to preprocess data from qbed data to call peaks, make annotation, make Anndata object and filter peaks."
- [methods] Reading (rd) includes several functions to read and save qbed/ccf and peak data.: "**Reading (rd)** includes several functions to read and save qbed/ccf and peak data."
- [methods] down_sample: "preprocessing.down_sample"
