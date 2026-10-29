---
name: peak-caller-scaling-visualization
description: Use when comparing computational cost (wall-clock runtime, peak RSS memory)
  of calling-cards peak callers across a dilution series of qbed datasets with increasing
  insertion counts (e.g., >5M insertions).
when_to_use_negative:
- The goal is peak-calling accuracy or binding-site discovery rather than computational
  performance scaling.
- The input is single-cell CC data requiring annotation, Anndata construction, differential
  peak analysis, or motif calling rather than bulk runtime benchmarking.
- The dataset has far fewer than five million insertions, where the reported speed
  differences (CCcaller 7-10x faster than MACCs/ccf_tools, 2-3x faster than Blockify)
  do not manifest.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: Pycallingcards
  role: Python framework providing reading.read_qbed, preprocessing.down_sample, preprocessing.call_peaks,
    and plotting for the benchmark workflow
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: CCcaller
  role: Pycallingcards peak caller invoked via call_peaks; the fast method whose scaling
    (7-10x faster than MACCs/ccf_tools, 2-3x faster than Blockify on >5M insertions)
    is the expected outcome
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: MACCs
  role: Pycallingcards peak caller invoked via call_peaks; slow baseline in the runtime/memory
    comparison
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: Python
  role: Implementation language; timing and memory (RSS) measurement and matplotlib
    visualization
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae070
    title: 'Pycallingcards: an integrated environment for visualizing, analyzing,
      and interpreting Calling Cards data'
  source_content_hash: sha256:2be4d10061c7a17cc60bfb46745948b4172e07edbdf34c343b31c3ebb271aad7
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# peak-caller-scaling-visualization

## Summary

Benchmarks how runtime and memory of CC peak calling methods (CCcaller, MACCs, Blockify, Blockify_original, ccf_tools) scale with total transposon insertions, visualizing scaling curves analogous to the Pycallingcards Fig. 2f-g runtime/memory comparison.

## When to use

Use when comparing computational cost (wall-clock runtime, peak RSS memory) of calling-cards peak callers across a dilution series of qbed datasets with increasing insertion counts (e.g., >5M insertions).

## When NOT to use

- The goal is peak-calling accuracy or binding-site discovery rather than computational performance scaling.
- The input is single-cell CC data requiring annotation, Anndata construction, differential peak analysis, or motif calling rather than bulk runtime benchmarking.
- The dataset has far fewer than five million insertions, where the reported speed differences (CCcaller 7-10x faster than MACCs/ccf_tools, 2-3x faster than Blockify) do not manifest.

## Inputs

- qbed file of calling cards insertions (e.g., PBase Brd4 K562 background-free qbed >20M insertions, or SP1/Brd4 K562-HCT116 with-background qbed)
- bundled pycallingcards datasets (datasets.SP1_K562HCT116_data, datasets.mouse_brd4_data)
- downsampled qbed subsets spanning a range of total insertion counts

## Outputs

- runtime_memory_benchmark.csv with wall-clock runtime and peak memory (RSS) for every caller/insertion-count combination
- runtime-versus-insertion-number and memory-usage plots per caller (Fig. 2f-g analog), e.g., PNG/PDF via matplotlib or pycallingcards plotting

## How to apply

Load a large qbed (e.g., the PBase Brd4 K562 background-free qbed with >20M insertions, or the SP1/Brd4 K562-HCT116 with-background qbed) via pycallingcards.reading.read_qbed or the bundled loaders (datasets.SP1_K562HCT116_data, datasets.mouse_brd4_data). Generate a dilution series of decreasing insertion counts with pycallingcards.preprocessing.down_sample so scaling versus total insertion number can be measured. Run the Pycallingcards callers CCcaller and MACCs through pycallingcards.preprocessing.call_peaks on every downsampled subset, timing each run, and run the external baselines Blockify, Blockify_original, and ccf_tools on identical subsets under identical conditions. Record wall-clock runtime and peak memory (RSS) per caller/insertion-count combination into runtime_memory_benchmark.csv, then plot runtime versus total insertions and memory per caller with matplotlib/pycallingcards plotting to reproduce the Fig. 2f-g style scaling curves.

## Related tools

- **Pycallingcards** (Python framework providing reading.read_qbed, preprocessing.down_sample, preprocessing.call_peaks, and plotting for the benchmark workflow) — https://github.com/The-Mitra-Lab/pycallingcards
- **CCcaller** (Pycallingcards peak caller invoked via call_peaks; the fast method whose scaling (7-10x faster than MACCs/ccf_tools, 2-3x faster than Blockify on >5M insertions) is the expected outcome) — https://github.com/The-Mitra-Lab/pycallingcards
- **MACCs** (Pycallingcards peak caller invoked via call_peaks; slow baseline in the runtime/memory comparison) — https://github.com/The-Mitra-Lab/pycallingcards
- **Python** (Implementation language; timing and memory (RSS) measurement and matplotlib visualization)

## Examples

```
import pycallingcards as cc; qbed = cc.datasets.SP1_K562HCT116_data(); sub = cc.pp.down_sample(qbed); peaks = cc.pp.call_peaks(sub, method='CCcaller')
```

## Evaluation signals

- CCcaller completes in <5 minutes on datasets with >5 million insertions
- Measured speedup is approximately 7-10x over MACCs and ccf_tools and 2-3x over Blockify on >5M-insertion datasets, matching the reported Fig. 2f-g performance envelope
- runtime_memory_benchmark.csv contains a runtime and RSS entry for every caller/insertion-count combination with no missing cells
- Runtime curves are monotonic with total insertion number across the down_sample dilution series
- All external callers (Blockify, Blockify_original, ccf_tools) ran on identical downsampled subsets under identical conditions

## Limitations

- The reported speedup envelope (7-10x vs MACCs/ccf_tools, 2-3x vs Blockify) applies to datasets with more than five million insertions; smaller datasets may not show the same separation.
- Baseline callers Blockify, Blockify_original, and ccf_tools are external to Pycallingcards and must be installed and run separately under identical conditions for a fair comparison.

## Evidence

- [other] CCcaller runs significantly faster than the other peak calling methods, running 7-10x faster than MACCs and ccf_tools and 2- to 3-fold faster than Blockify: [evidence span withheld — anchor status: not_found]
- [abstract] Pycallingcards introduces two innovative peak callers, CCcaller and MACCs, enhancing the accuracy and speed of pinpointing TF binding sites from CC data.: "Pycallingcards introduces two innovative peak callers, CCcaller and MACCs, enhancing the accuracy and speed of pinpointing TF binding sites from CC data."
- [methods] **Preprocessing (pp)** helps to preprocess data from qbed/ccf data to call peaks, make annotation, make Anndata object and filter peaks.: "**Preprocessing (pp)** helps to preprocess data from qbed/ccf data to call peaks, make annotation, make Anndata object and filter peaks."
- [readme] **Reading (rd)** includes several functions to read and save qbed and peak data.: "**Reading (rd)** includes several functions to read and save qbed and peak data."
- [readme] **Plotting (pl)** proveides an allround plottting system.: "**Plotting (pl)** proveides an allround plottting system. It could plot genome areas, link with [WashU Epigenome Browser](http://epigenomegateway.wustl.edu/browser/)"
