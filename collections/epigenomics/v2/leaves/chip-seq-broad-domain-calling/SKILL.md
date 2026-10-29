---
name: chip-seq-broad-domain-calling
description: Use when the goal is to identify broad ChIP-seq enrichment domains spanning
  kb to Mb scales (e.g., H3K27me3 or H3K9me3) from mapped-read BED files, especially
  where whole-domain integrity (covering full gene bodies of repressed genes) matters
  more than sharp peak positions.
when_to_use_negative:
- The target marks produce sharp, punctate peaks rather than diffuse kb-to-Mb domains
  (RECOGNICER is designed for broad signal detection, not narrow peak calling)
- You need to resolve two domains separated by fine-scale gaps — RECOGNICER can falsely
  merge them into one coarse-scale domain, requiring manual monitoring or masked/featured
  regions
- Multiple RECOGNICER instances must run in parallel from the same directory (hard-coded
  temporary filenames will interfere); use separate directories instead
edam_operation: http://edamontology.org/operation_3563
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0749
tools:
- name: RECOGNICER
  role: 'Primary tool: recursive coarse-graining block transformation, domain retrieval
    with boundary determination, and Poisson-based significance determination for
    broad domain calling'
  repo: https://github.com/zanglab/recognicer
- name: SICER
  role: Comparator method for broad domain calling, run with default parameters in
    benchmarking repressed-gene-body coverage
- name: RSEG
  role: HMM-based comparator method for broad epigenomic domain identification, run
    with default parameters in benchmarking
- name: MUSIC
  role: Multiscale-decomposition comparator method for ChIP-seq peak calling, run
    with default parameters in benchmarking
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_005
  source_papers:
  - doi: 10.1007/s40484-020-0225-2
    title: 'RECOGNICER: A coarse‐graining approach for identifying broad domains from
      ChIP‐seq data'
  source_content_hash: sha256:7adc56e02d601678cc46f7e9a0e6cd7f50f21830a87fbcb76b57fa572153a58a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/zanglab/recognicer
  tools:
  - MUSIC
  - RECOGNICER
  - RSEG
  - SICER
  edam_operation: http://edamontology.org/operation_3563
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# chip-seq-broad-domain-calling

## Summary

Call multi-scale broad enrichment domains (kb-to-Mb, e.g., H3K27me3/H3K9me3) from ChIP-seq mapped reads using RECOGNICER's recursive coarse-graining block transformation, followed by candidate domain retrieval with boundary determination and Poisson-model significance testing with FDR control.

## When to use

Use when the goal is to identify broad ChIP-seq enrichment domains spanning kb to Mb scales (e.g., H3K27me3 or H3K9me3) from mapped-read BED files, especially where whole-domain integrity (covering full gene bodies of repressed genes) matters more than sharp peak positions.

## When NOT to use

- The target marks produce sharp, punctate peaks rather than diffuse kb-to-Mb domains (RECOGNICER is designed for broad signal detection, not narrow peak calling)
- You need to resolve two domains separated by fine-scale gaps — RECOGNICER can falsely merge them into one coarse-scale domain, requiring manual monitoring or masked/featured regions
- Multiple RECOGNICER instances must run in parallel from the same directory (hard-coded temporary filenames will interfere); use separate directories instead

## Inputs

- ChIP-seq mapped-read BED file for the ChIP (treatment) sample
- Matched control ChIP-seq mapped-read BED file
- Optional: matched gene expression profiles (microarray or RNA-seq) for functional validation of repressed genes

## Outputs

- $SAMPLE.cgsummary — candidate domains with chrom, start, end, ChIP read count, control read count, P-value, fold change, Q-value
- $SAMPLE-fdr0.01_broadPeak.bed — final significant domains in ENCODE broadPeak format (5th column = fold change, 7th column = ChIP read count)
- $SAMPLE-W200.bedgraph — read pile-up track in bedGraph format at 200 bp resolution for genome browser visualization
- $SAMPLE-nonredundant.bed — non-redundant reads filtered from the input ChIP dataset

## How to apply

Partition each chromosome into equal-sized non-overlapping windows (default 200 bp, roughly mononucleosome+linker) and shift each single-end read from its 5' end by half the DNA fragment size (default 150 bp). Apply the recursive block transformation with block transformation factor L=3 and phase optimization over all 3 possible phases, computing the auto-correlation curve and fitted characteristic length λ at each coarse-graining round; terminate when λ drops below 1 on the next coarse scale. Retrieve candidate domains with boundary determination, then assess significance by comparing ChIP and control read counts under a local Poisson model, applying the Benjamini-Hochberg procedure for FDR (default threshold 0.01 passed to RECOGNICER.sh). Run RECOGNICER via `sh RECOGNICER.sh ChIP.bed control.bed 0.01`, configuring SAMPLE, SPECIES (e.g., hg18/hg19/hg38), OUT_DIR, and optionally WINDOW_SIZE and FRAGMENT_SIZE; the method is robust to sequencing depth (total domain length and FRIP invariant from 17.23M down to 4M reads) and to fragment-size/window-size parameter choices.

## Related tools

- **RECOGNICER** (Primary tool: recursive coarse-graining block transformation, domain retrieval with boundary determination, and Poisson-based significance determination for broad domain calling) — https://github.com/zanglab/recognicer
- **SICER** (Comparator method for broad domain calling, run with default parameters in benchmarking repressed-gene-body coverage)
- **RSEG** (HMM-based comparator method for broad epigenomic domain identification, run with default parameters in benchmarking)
- **MUSIC** (Multiscale-decomposition comparator method for ChIP-seq peak calling, run with default parameters in benchmarking)

## Examples

```
sh RECOGNICER.sh ChIP.bed control.bed 0.01
```

## Evaluation signals

- Total aggregate length of identified significantly enriched domains remains approximately invariant when downsampling from 17.23M to 4M reads; FRIP score stays at the same level (depth invariance)
- Total domain length and total read count within domains remain approximately stable when varying FRAGMENT_SIZE (default 150) and WINDOW_SIZE (default 200 up to 1000) (parameter insensitivity)
- Output broadPeak file contains one row per significant domain with valid ENCODE broadPeak schema; score column (fold change) and signalValue column (ChIP read count) are populated
- In the 9-ENCODE-cell-type benchmark, RECOGNICER-identified H3K27me3 domains cover the entire gene bodies of more repressed genes than SICER, RSEG, and MUSIC in 8 of 9 cell types
- Auto-correlation characteristic length λ at each coarse-graining round behaves as expected (≈100 kb for synthetic data; growing with coarse-graining for real data) and termination occurs when λ drops below 1 on the next coarse scale

## Limitations

- RECOGNICER could falsely merge two domains separated by fine-scale gaps into one domain on the coarse scale; individual cases should be specifically monitored and other information (featured or masked regions) may need to be incorporated
- Parallel execution of multiple RECOGNICER instances from the same directory corrupts results due to hard-coded temporary file names — separate directories are required
- Implemented in python2 with a scipy prerequisite; only pre-configured genomes in GenomeData.py are supported unless customized (human hg18/hg19/hg38, mouse mm8/mm9/mm10, rat rn4, drosophila dm2/dm3, yeast sacCer1/pombe, Arabidopsis tair8)

## Evidence
