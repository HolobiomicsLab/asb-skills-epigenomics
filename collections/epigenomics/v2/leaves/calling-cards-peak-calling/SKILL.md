---
name: calling-cards-peak-calling
description: Use when input is bulk or single-cell Calling Cards qbed/ccf transposon-insertion
  data (e.g. Brd4 or SP1 CC datasets from GEO) and TF binding sites must be identified
  as peaks before annotation, differential analysis, or peak-gene pairing.
when_to_use_negative:
- Input is ChIP-seq, RNA-seq, or already-processed peak/gene feature data — this skill
  is specific to qbed/ccf Calling Cards insertions, not other peak-source assays
- The goal is downstream analysis (motif calling, differential peaks, peak-gene pairing)
  on existing peaks — use tools (tl) functions instead of re-calling peaks
- Peak caller runtime is irrelevant and a non-CC baseline caller (e.g. Blockify) is
  already the lab standard — benchmarking context is where CCcaller/MACCs advantages
  were demonstrated
tools:
- name: Pycallingcards
  role: Core Python package providing preprocessing.clean_qbed, preprocessing.combine_peaks,
    and preprocessing.call_peaks for qbed-to-peak workflows
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: CCcaller
  role: New Pycallingcards peak caller; fastest and most concordant in benchmarks
    (e.g. maxbetween=1100 for GBM single-cell data)
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: MACCs
  role: Alternative Pycallingcards peak caller producing peaks highly concordant with
    CCcaller
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: bedtools
  role: Backend (with pybedtools) used for genomic interval operations such as peak
    annotation after calling
  repo: https://bedtools.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_001
  - task_006
  - task_002
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae070
    title: 'Pycallingcards: an integrated environment for visualizing, analyzing,
      and interpreting Calling Cards data'
  source_content_hash: sha256:91264f9619b89f777d59f70aab085b3a7724b2bbc550c236d65fb61bcfd25bb8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/The-Mitra-Lab/pycallingcards
  tools:
  - CCcaller
  - HOMER
  - MACCs
  - Mudata
  - Pycallingcards
  - Python
  - Scanpy
  - bedtools
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# calling-cards-peak-calling

## Summary

Call TF binding-site peaks from Calling Cards qbed/ccf insertion data using the Pycallingcards package's CCcaller and MACCs peak callers, the two new callers introduced by Guo et al. (2024) to improve accuracy and speed of pinpointing TF binding sites over legacy methods like Blockify.

## When to use

Use when input is bulk or single-cell Calling Cards qbed/ccf transposon-insertion data (e.g. Brd4 or SP1 CC datasets from GEO) and TF binding sites must be identified as peaks before annotation, differential analysis, or peak-gene pairing.

## When NOT to use

- Input is ChIP-seq, RNA-seq, or already-processed peak/gene feature data — this skill is specific to qbed/ccf Calling Cards insertions, not other peak-source assays
- The goal is downstream analysis (motif calling, differential peaks, peak-gene pairing) on existing peaks — use tools (tl) functions instead of re-calling peaks
- Peak caller runtime is irrelevant and a non-CC baseline caller (e.g. Blockify) is already the lab standard — benchmarking context is where CCcaller/MACCs advantages were demonstrated

## Inputs

- Calling Cards qbed file (genomic insertion coordinates) for bulk or single-cell CC data
- Calling Cards ccf file (alternative insertion format readable by pycallingcards.reading)
- Bundled datasets (e.g. datasets.SP1_K562HCT116_data, datasets.mouse_brd4_data, datasets.mousecortex_data, datasets.SP1_Cre_data)
- GEO-hosted CC datasets such as GSE156821, GSE148448, GSE128493, GSE134427

## Outputs

- Peak BED-style table of called TF binding sites per condition/caller
- Cell-by-peak AnnData object (after preprocessing.make_Anndata) for downstream differential analysis
- Benchmark outputs such as peak_calling_comparison.csv (peak overlap %, ChIP-seq signal) or runtime_memory_benchmark.csv

## How to apply

Load the qbed insertion table (e.g. via pycallingcards.datasets.SP1_K562HCT116_data, datasets.mouse_brd4_data, or pycallingcards.reading.read_qbed on GEO records such as GSE156821/GSE148448) and clean insertions with pycallingcards.preprocessing.clean_qbed. Call peaks with pycallingcards.preprocessing.call_peaks, choosing the CCcaller or MACCs method; for single-cell with-background data, first merge conditions with preprocessing.combine_peaks (e.g. combining male and female GBM Brd4 data with CCcaller at maxbetween=1100). For runtime/scaling comparisons, generate dilution series with preprocessing.down_sample and benchmark wall-clock and memory per caller. The rationale is benchmark-backed: CCcaller and MACCs produce highly concordant peak sets with stronger ChIP-seq (H3K27ac/SP1) signal enrichment than Blockify/Blockify_original/ccf_tools, and CCcaller runs 7-10x faster than MACCs and ccf_tools on datasets exceeding five million insertions. Downstream, peaks feed preprocessing.annotation (bedtools/pybedtools backend) and preprocessing.make_Anndata to build cell-by-peak matrices.

## Related tools

- **Pycallingcards** (Core Python package providing preprocessing.clean_qbed, preprocessing.combine_peaks, and preprocessing.call_peaks for qbed-to-peak workflows) — https://github.com/The-Mitra-Lab/pycallingcards
- **CCcaller** (New Pycallingcards peak caller; fastest and most concordant in benchmarks (e.g. maxbetween=1100 for GBM single-cell data)) — https://github.com/The-Mitra-Lab/pycallingcards
- **MACCs** (Alternative Pycallingcards peak caller producing peaks highly concordant with CCcaller) — https://github.com/The-Mitra-Lab/pycallingcards
- **bedtools** (Backend (with pybedtools) used for genomic interval operations such as peak annotation after calling) — https://bedtools.readthedocs.io/en/latest/

## Examples

```
import pycallingcards as cc; qbed = cc.pp.clean_qbed(cc.datasets.SP1_K562HCT116_data()); peaks = cc.pp.call_peaks(qbed, method='CCcaller')
```

## Evaluation signals

- Peaks called with CCcaller and MACCs on the same qbed show high pairwise overlap concordance, whereas Blockify/Blockify_original/ccf_tools show more discordant sets (Fig. 2c-e pattern)
- Average H3K27ac and SP1 ChIP-seq signal under called peaks (via pycallingcards.plotting.calculate_signal / bedtools overlap) is higher for CCcaller/MACCs peaks than for baseline callers
- CCcaller completes in under 5 minutes on datasets with more than 5 million insertions, showing ~7-10x speedup over MACCs/ccf_tools and ~2-3x over Blockify
- Output peak set table contains a row per peak caller per dataset with average H3K27ac/SP1 signal and pairwise overlap-percentage fields
- clean_qbed was applied before call_peaks so insertion records are deduplicated/cleaned prior to peak calling

## Limitations

- Peak-caller accuracy and speed advantages were demonstrated by benchmark against Blockify, Blockify_original, and ccf_tools on HyPBase Brd4 bulk HCT116 and SP1 scCC HCT116 data; other TFs/assays may behave differently
- For with-background single-cell data, conditions must first be merged (combine_peaks) and parameters tuned (e.g. maxbetween=1100), so default-parameter behavior does not transfer uniformly between bulk and single-cell settings
- bedtools/pybedtools must be installed separately for the annotation step; the peak-calling skill alone does not annotate peaks

## Evidence

- [abstract] Pycallingcards introduces two innovative peak callers, CCcaller and MACCs, enhancing the accuracy and speed of pinpointing TF binding sites from CC data.: "Pycallingcards introduces two innovative peak callers, CCcaller and MACCs, enhancing the accuracy and speed of pinpointing TF binding sites from CC data."
- [methods] Preprocessing helps to preprocess data from qbed/ccf data to call peaks, make annotation, make Anndata object and filter peaks.: "**Preprocessing (pp)** helps to preprocess data from qbed/ccf data to call peaks, make annotation, make Anndata object and filter peaks."
- [readme] Calling cards is a sequencing technology to assay TF binding which could be done in vitro and in vivo at both bulk and single-cell level.: "Calling cards is a sequencing technology to assay TF binding which could be done in vitro and in vivo at both bulk and single-cell level."
- [methods] In order to complete the annotation, bedtools (includes pybedtools) also needs to be installed.: "In order to complete the annotation, [bedtools](https://bedtools.readthedocs.io/en/latest/) (includes [pybedtools](https://daler.github.io/pybedtools/)) also needs to be installed."
