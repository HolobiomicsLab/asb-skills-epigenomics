---
name: peak-caller-benchmarking
description: Use when multiple CC peak callers are available for qbed input and you
  must decide which caller to trust, or when reproducing the Pycallingcards Fig. 2c-e
  accuracy benchmark comparing callers on bulk and single-cell HCT116 datasets.
when_to_use_negative:
- Input is not qbed/ccf calling-cards insertion data (e.g. already a peak bed file
  or ChIP-seq peaks) — the skill benchmarks callers on raw insertion-level qbed input.
- The goal is single-caller optimization or parameter tuning rather than comparing
  multiple peak callers under their default parameters.
- No matched ChIP-seq (e.g. H3K27ac, SP1) reference signal is available to compute
  enrichment under called peaks, and no second caller exists for overlap comparison.
edam_operation: http://edamontology.org/operation_3563
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0769
tools:
- name: Pycallingcards
  role: Provides the data loaders (datasets.SP1_K562HCT116_data), preprocessing (clean_qbed),
    built-in peak callers (CCcaller, MACCs via call_peaks), and ChIP-seq signal calculation
    (plotting.calculate_signal)
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: CCcaller
  role: Benchmarked peak caller built into Pycallingcards; calls peaks on bulk and
    scCC qbed data with default parameters and shows high concordance with MACCs
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: MACCs
  role: Benchmarked peak caller built into Pycallingcards; second reference caller
    called with default parameters via call_peaks
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: bedtools
  role: Interval-overlap engine (with pybedtools) used to compute average ChIP-seq
    signal under peaks and pairwise peak-overlap percentages
  repo: https://bedtools.readthedocs.io/en/latest/
- name: Blockify
  role: External comparison peak caller (Pycallingcards implementation) run with default
    settings to obtain reference/discordant peak sets
- name: ccf_tools
  role: External comparison peak caller run with default settings on the same qbed
    inputs for the concordance benchmark
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae070
    title: 'Pycallingcards: an integrated environment for visualizing, analyzing,
      and interpreting Calling Cards data'
  source_content_hash: sha256:1559c50e524681777235ed7756830a5ef10a21ee294660d8b5c87b05d129e28b
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
  - bedtools
  - ccf_tools
  edam_operation: http://edamontology.org/operation_3563
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# peak-caller-benchmarking

## Summary

Benchmark calling-cards (CC) peak callers — CCcaller, MACCs, Blockify (Pycallingcards), Blockify_original, and ccf_tools — by comparing ChIP-seq signal enrichment under called peaks and pairwise peak-overlap concordance on HyPBase Brd4 bulk CC HCT116 and SP1 scCC HCT116 data.

## When to use

Use when multiple CC peak callers are available for qbed input and you must decide which caller to trust, or when reproducing the Pycallingcards Fig. 2c-e accuracy benchmark comparing callers on bulk and single-cell HCT116 datasets.

## When NOT to use

- Input is not qbed/ccf calling-cards insertion data (e.g. already a peak bed file or ChIP-seq peaks) — the skill benchmarks callers on raw insertion-level qbed input.
- The goal is single-caller optimization or parameter tuning rather than comparing multiple peak callers under their default parameters.
- No matched ChIP-seq (e.g. H3K27ac, SP1) reference signal is available to compute enrichment under called peaks, and no second caller exists for overlap comparison.

## Inputs

- SP1 single-cell calling cards qbed data (K562/HCT116) via pycallingcards.datasets.SP1_K562HCT116_data
- Brd4 bulk calling cards qbed data from GEO accession GSE156821
- H3K27ac and SP1 ChIP-seq bigWig/bed signal files for enrichment calculation
- Comparison caller implementations: Blockify, Blockify_original, ccf_tools
- bedtools/pybedtools installation for interval overlap

## Outputs

- Per-caller peak sets (peaks called with CCcaller, MACCs, Blockify, Blockify_original, ccf_tools) for each dataset
- Average H3K27ac and SP1 ChIP-seq signal under each caller's peaks
- Pairwise peak-overlap percentages between all caller pairs for bulk (no-background) and single-cell (with-background) datasets
- peak_calling_comparison.csv with a row per peak caller per dataset

## How to apply

Load the SP1 single-cell CC qbed data via pycallingcards.datasets.SP1_K562HCT116_data and the Brd4 bulk CC qbed data from GEO accession GSE156821, cleaning insertions with pycallingcards.preprocessing.clean_qbed. Call peaks on both cleaned qbed inputs with pycallingcards.preprocessing.call_peaks using the built-in CCcaller and MACCs callers under default parameters, and run the comparison callers Blockify, Blockify_original, and ccf_tools with default settings on the same inputs. Quantify accuracy two ways: average H3K27ac and SP1 ChIP-seq signal under each caller's peaks using pycallingcards.plotting.calculate_signal (interval overlap via bedtools/pybedtools), and pairwise peak-overlap percentages between all caller pairs for both the no-background bulk and with-background single-cell datasets. Write per-caller peak sets with ChIP-seq signal and overlap percentages to peak_calling_comparison.csv. The expected result is that CCcaller and MACCs produce highly concordant peak sets, while ccf_tools, Blockify, and Blockify_original yield more discordant sets.

## Related tools

- **Pycallingcards** (Provides the data loaders (datasets.SP1_K562HCT116_data), preprocessing (clean_qbed), built-in peak callers (CCcaller, MACCs via call_peaks), and ChIP-seq signal calculation (plotting.calculate_signal)) — https://github.com/The-Mitra-Lab/pycallingcards
- **CCcaller** (Benchmarked peak caller built into Pycallingcards; calls peaks on bulk and scCC qbed data with default parameters and shows high concordance with MACCs) — https://github.com/The-Mitra-Lab/pycallingcards
- **MACCs** (Benchmarked peak caller built into Pycallingcards; second reference caller called with default parameters via call_peaks) — https://github.com/The-Mitra-Lab/pycallingcards
- **bedtools** (Interval-overlap engine (with pybedtools) used to compute average ChIP-seq signal under peaks and pairwise peak-overlap percentages) — https://bedtools.readthedocs.io/en/latest/
- **Blockify** (External comparison peak caller (Pycallingcards implementation) run with default settings to obtain reference/discordant peak sets)
- **ccf_tools** (External comparison peak caller run with default settings on the same qbed inputs for the concordance benchmark)

## Examples

```
import pycallingcards as cc; qbed = cc.pp.clean_qbed(cc.datasets.SP1_K562HCT116_data()); peaks = cc.pp.call_peaks(qbed, method='CCcaller')
```

## Evaluation signals

- peak_calling_comparison.csv exists with a row per peak caller per dataset (bulk no-background and single-cell with-background).
- The table contains average H3K27ac/SP1 ChIP-seq signal fields and pairwise overlap-percentage fields for every caller pair.
- Overlap percentages reproduce the relative caller ranking shown in Fig. 2c-e (CCcaller vs MACCs most concordant; ccf_tools, Blockify, Blockify_original more discordant).
- Each caller's peaks were called from the same cleaned qbed input (after preprocessing.clean_qbed) so pairwise overlap comparisons are apples-to-apples.
- Average ChIP-seq signal under peaks is computed consistently via plotting.calculate_signal with the same H3K27ac/SP1 signal files across all callers.

## Limitations

- Benchmark uses default parameters for all callers; results may shift under tuned settings.
- External comparison callers (Blockify, Blockify_original, ccf_tools) are not native to Pycallingcards and may require separate installation or adapters.
- Overlapped/discordant results are specific to the benchmarked HCT116 datasets (HyPBase Brd4 bulk and SP1 scCC); concordance rankings may not generalize to other TFs, cell types, or with/with-background calling regimes.

## Evidence

- [abstract] Pycallingcards introduces two innovative peak callers, CCcaller and MACCs, enhancing the accuracy and speed of pinpointing TF binding sites from CC data.: "Pycallingcards introduces two innovative peak callers, CCcaller and MACCs, enhancing the accuracy and speed of pinpointing TF binding sites from CC data."
- [results] CCcaller and MACCs call very similar (highly concordant) sets of peaks, whereas ccf_tools, Blockify, and Blockify_original produce more discordant peak sets.: [evidence span withheld — anchor status: not_found]
- [methods] **Preprocessing (pp)** helps to preprocess data from qbed/ccf data to call peaks, make annotation, make Anndata object and filter peaks.: "**Preprocessing (pp)** helps to preprocess data from qbed/ccf data to call peaks, make annotation, make Anndata object and filter peaks."
- [methods] In order to complete the annotation, bedtools (includes pybedtools) also needs to be installed.: "In order to complete the annotation, [bedtools](https://bedtools.readthedocs.io/en/latest/) (includes [pybedtools](https://daler.github.io/pybedtools/)) also needs to be installed."
- [readme] **Plotting (pl)** ... show signal comparison with Chip-seq(BigWig file), display differential peaks: "show signal comparison with Chip-seq(BigWig file), display differential peaks, demonstrate potenial binding-gene expression relationships."
