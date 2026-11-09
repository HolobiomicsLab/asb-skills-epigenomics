---
name: genomic-interval-processing
description: Use when the task requires comparing, merging, or signal-quantifying
  genomic intervals, e.g. calling CC peaks from qbed insertions, annotating peaks,
  or computing ChIP-seq signal under called peaks.
when_to_use_negative:
- Input is already a gene expression matrix or Anndata object with no interval-level
  operations needed (use Scanpy-style analysis instead).
- The data are raw sequencing reads (FASTQ) prior to insertion extraction — qbed/ccf
  parsing must happen first.
- Peak-calling parameters must be non-default or the caller is not CCcaller/MACCs/Blockify;
  the documented benchmark is for these five callers with default parameters.
edam_operation: http://edamontology.org/operation_0327
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: Pycallingcards
  role: Python module providing qbed reading (rd), peak calling, peak combination/separation,
    annotation, filtering (pp), and interval signal plotting (pl) for CC data
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: bedtools
  role: Interval overlap backend (via pybedtools) used to compute ChIP-seq signal
    under called peaks and annotate peaks
  repo: https://bedtools.readthedocs.io/en/latest/
- name: CCcaller
  role: 'Greedy insertion-based peak caller: single-pass candidate peak extension
    with Poisson enrichment promotion over background insertions'
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: MACCs
  role: Alternative Pycallingcards peak caller; candidate peaks are promoted to CC
    peaks per the MACCs method; highly concordant with CCcaller
  repo: https://github.com/The-Mitra-Lab/pycallingcards
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae070
    title: 'Pycallingcards: an integrated environment for visualizing, analyzing,
      and interpreting Calling Cards data'
  source_content_hash: sha256:917400c5d4f901dd1e0957cad50dae909153d8616dd14fd79ef2f85d497c0983
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/The-Mitra-Lab/pycallingcards
  tools:
  - CCcaller
  - MACCs
  - Pycallingcards
  - Python
  - bedtools
  edam_operation: http://edamontology.org/operation_0327
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 24
---

# genomic-interval-processing

## Summary

Calling Cards insertions (qbed) and called TF-binding peaks (BED-like intervals) are manipulated, compared, and annotated as genomic intervals — the core interval algebra (overlap, signal calculation, peak merging/separation) underlying Pycallingcards' peak-calling and ChIP-seq comparison workflows.

## When to use

Use when the task requires comparing, merging, or signal-quantifying genomic intervals, e.g. calling CC peaks from qbed insertions, annotating peaks, or computing ChIP-seq signal under called peaks.

## When NOT to use

- Input is already a gene expression matrix or Anndata object with no interval-level operations needed (use Scanpy-style analysis instead).
- The data are raw sequencing reads (FASTQ) prior to insertion extraction — qbed/ccf parsing must happen first.
- Peak-calling parameters must be non-default or the caller is not CCcaller/MACCs/Blockify; the documented benchmark is for these five callers with default parameters.

## Inputs

- qbed insertion files (Calling Cards data, bulk or single-cell; also ccf files)
- BED peak files (called peaks with chromosome, start, end, statistics)
- ChIP-seq BigWig signal files (e.g. H3K27ac, SP1) for signal-under-peak calculation
- peak-caller reference outputs (Blockify, Blockify_original, ccf_tools) for overlap comparison

## Outputs

- Called peak intervals (BED-format, e.g. cccaller_peaks.bed) with Poisson enrichment statistics
- Peak-overlap percentage matrices between peak callers (e.g. peak_calling_comparison.csv)
- Average ChIP-seq signal per peak set (H3K27ac / TF signal under peaks)
- Annotated Anndata objects with filtered peaks

## How to apply

Load insertion data as qbed and clean it with pycallingcards.preprocessing.clean_qbed, then call TF-binding site intervals with preprocessing.call_peaks using the CCcaller or MACCs callers (CCcaller greedily extends a candidate peak while the next nearest insertion falls within a user-specified maximum distance, then promotes candidates to CC peaks by Poisson enrichment over background insertions in a single ordered pass). Related interval operations include preprocessing.combine_peaks and preprocessing.separate_peaks to merge or split called peaks, and preprocessing.filter_peaks to filter peaks after building the Anndata object. For benchmarking, compute average H3K27ac or TF ChIP-seq signal under each caller's peaks using pycallingcards.plotting.calculate_signal with interval overlap via bedtools/pybedtools, and quantify pairwise peak-overlap percentages between callers. Output interval sets as BED files (e.g. cccaller_peaks.bed with chromosome, start, end, statistics).

## Related tools

- **Pycallingcards** (Python module providing qbed reading (rd), peak calling, peak combination/separation, annotation, filtering (pp), and interval signal plotting (pl) for CC data) — https://github.com/The-Mitra-Lab/pycallingcards
- **bedtools** (Interval overlap backend (via pybedtools) used to compute ChIP-seq signal under called peaks and annotate peaks) — https://bedtools.readthedocs.io/en/latest/
- **CCcaller** (Greedy insertion-based peak caller: single-pass candidate peak extension with Poisson enrichment promotion over background insertions) — https://github.com/The-Mitra-Lab/pycallingcards
- **MACCs** (Alternative Pycallingcards peak caller; candidate peaks are promoted to CC peaks per the MACCs method; highly concordant with CCcaller) — https://github.com/The-Mitra-Lab/pycallingcards

## Examples

```
import pycallingcards as cc; qbed = cc.datasets.SP1_K562HCT116_data(); peaks = cc.pp.call_peaks(qbed, method='CCcaller', reference='hg38'); cc.pl.calculate_signal(peaks, 'H3K27ac.bed')
```

## Evaluation signals

- Peak set files exist in BED format with chromosome, start, end and statistics columns for each caller/dataset.
- CCcaller runs in a single ordered pass over sorted insertions — every insertion is considered only once; peak boundaries never regress beyond the user-specified maximum distance.
- Output table (e.g. peak_calling_comparison.csv) has one row per peak caller per dataset with average H3K27ac/SP1 signal and pairwise overlap-percentage fields.
- Overlap percentages reproduce the relative caller ranking of Fig. 2c-e: CCcaller and MACCs show highly concordant peak sets, while ccf_tools, Blockify, and Blockify_original are more discordant.
- Downstream filtering (preprocessing.filter_peaks) applies cleanly to the Anndata object built from called peaks.

## Limitations

- Peak calling requires a user-specified maximum insertion distance for candidate peak extension; peak sets depend on this parameter and on the Poisson background rate.
- Benchmarks reported are for five callers (CCcaller, MACCs, Blockify, Blockify_original, ccf_tools) with default parameters on HyPBase Brd4 bulk CC HCT116 and SP1 scCC HCT116 data; other settings are untested here.
- Annotation requires bedtools (and pybedtools) to be installed, so interval operations fail if that dependency is missing.

## Evidence

- [abstract] Pycallingcards introduces two innovative peak callers, CCcaller and MACCs, enhancing the accuracy and speed of pinpointing TF binding sites from CC data.: "Pycallingcards introduces two innovative peak callers, CCcaller and MACCs, enhancing the accuracy and speed of pinpointing TF binding sites from CC data."
- [methods] Preprocessing (pp) helps to preprocess data from qbed/ccf data to call peaks, make annotation, make Anndata object and filter peaks.: "**Preprocessing (pp)** helps to preprocess data from qbed/ccf data to call peaks, make annotation, make Anndata object and filter peaks."
- [methods] In order to complete the annotation, bedtools (includes pybedtools) also needs to be installed.: "In order to complete the annotation, [bedtools](https://bedtools.readthedocs.io/en/latest/) (includes [pybedtools](https://daler.github.io/pybedtools/)) also needs to be installed."
- [readme] it could plot genome areas, link with WashU Epigenome Browser, show signal comparison with Chip-seq(BigWig file): "show signal comparison with Chip-seq(BigWig file)"
