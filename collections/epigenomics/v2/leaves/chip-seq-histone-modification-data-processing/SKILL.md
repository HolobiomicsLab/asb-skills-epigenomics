---
name: chip-seq-histone-modification-data-processing
description: Use when raw ChIP-Seq BAM files for histone marks (e.g. wgEncodeBroadHistoneK562H3k4me3StdAlnRep1.bam)
  must be converted into binned enrichment profiles over peak regions before alignment/clustering
  with DGW.
when_to_use_negative:
- Input is already binned enrichment data or a distance/feature matrix rather than
  raw aligned BAM reads
- The data are not ChIP-Seq enrichment measurements within peak regions identified
  by a peak finder such as MACS (e.g. RNA-seq or bulk genomic coverage)
- Regions lack any point-of-interest entries or fail the min-pileup threshold (no
  bin with more than 10 reads) and should simply be ignored rather than processed
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0203
tools:
- name: MACS2
  role: Peak caller that identifies enriched regions in histone mark BAM files relative
    to an input control sample
- name: MACS
  role: Peak finder whose called peak regions define the ChIP-Seq enrichment measurements
    used as input to DGW
- name: DGW
  role: Downstream consumer of the binned peak-region profiles for DTW-based alignment
    and clustering
  repo: github.com/lukauskas/dgw
- name: Python
  role: Language in which the DGW preprocessing and clustering package is implemented
    (requires Python 2.7)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-016-1306-0
    title: 'DGW: an exploratory data analysis tool for clustering and visualisation
      of epigenomic marks'
  source_content_hash: sha256:a8a6066859065741819ac53a24f0a826bce4ff39dcef31bc3d8f4d48d96431d8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DGW
  - MACS
  - MACS2
  - Python
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# chip-seq-histone-modification-data-processing

## Summary

Preprocess ChIP-Seq histone modification data (e.g. ENCODE H3K4me3/H3K9ac K562 BAM files) into binned peak-region enrichment tracks suitable for DGW alignment and clustering, by peak calling with MACS2, merging enriched regions, extending reads, and binning coverage into non-overlapping 50 bp windows.

## When to use

Use when raw ChIP-Seq BAM files for histone marks (e.g. wgEncodeBroadHistoneK562H3k4me3StdAlnRep1.bam) must be converted into binned enrichment profiles over peak regions before alignment/clustering with DGW.

## When NOT to use

- Input is already binned enrichment data or a distance/feature matrix rather than raw aligned BAM reads
- The data are not ChIP-Seq enrichment measurements within peak regions identified by a peak finder such as MACS (e.g. RNA-seq or bulk genomic coverage)
- Regions lack any point-of-interest entries or fail the min-pileup threshold (no bin with more than 10 reads) and should simply be ignored rather than processed

## Inputs

- Aligned ChIP-Seq fragment BAM files (e.g. wgEncodeBroadHistoneK562H3k4me3StdAlnRep1.bam, wgEncodeBroadHistoneK562H3k9acStdAlnRep1.bam)
- Input control BAM file (e.g. wgEncodeBroadHistoneK562ControlStdAlnRep1.bam)
- Gene annotation table from UCSC (knownGenes)

## Outputs

- MACS2-called enriched peak regions per histone mark
- Merged enriched region set (regions called for either mark)
- Binned coverage profiles over peak regions in non-overlapping 50 bp windows, filtered for peak length (5–1000 bins) and minimum counts (≥10)

## How to apply

Process aligned fragment BAM files with the MACS2 peak caller against an input control sample to identify enriched regions for each histone mark, then merge the two sets by considering every region called for either mark. Extend single-end reads to their estimated fragment lengths, and bin coverage within the merged peak regions into non-overlapping windows spanning 50 bp. Filter out unexpressed regions that have no bin with more than 10 reads (adjustable via the -min-pileup constraint), and restrict the analysed set of peaks to those longer than 5 and shorter than 1000 bins with at least 10 counts. These binned profiles are the DGW input for subsequent DTW-based clustering, as done for the ENCODE K562 H3K4me3 and H3K9ac datasets using the UCSC knownGenes table for annotation.

## Related tools

- **MACS2** (Peak caller that identifies enriched regions in histone mark BAM files relative to an input control sample)
- **MACS** (Peak finder whose called peak regions define the ChIP-Seq enrichment measurements used as input to DGW)
- **DGW** (Downstream consumer of the binned peak-region profiles for DTW-based alignment and clustering) — github.com/lukauskas/dgw
- **Python** (Language in which the DGW preprocessing and clustering package is implemented (requires Python 2.7))

## Examples

```
git clone git://github.com/sauliusl/dgw.git; pip install -e git://github.com/sauliusl/mlpy.git#egg=mlpy; pip install -e .
```

## Evaluation signals

- Each retained peak region has length between 5 and 1000 non-overlapping 50 bp bins
- Every retained peak has at least 10 counts and at least one bin with more than 10 reads
- Enriched regions are merged so that any region called for either histone mark is included once
- Binned profiles feed DGW and yield dendrograms in which enriched peaks cluster by shape, as demonstrated on the ENCODE K562 H3K4me3/H3K9ac data
- MACS2 calls are made against an input control sample, not de novo without control

## Limitations

- Regions with no bin exceeding 10 reads are ignored, so low-signal peaks are silently dropped
- Single-end reads must be extended to estimated fragment lengths, which depends on fragment-length estimation accuracy
- DGW requires Python 2.7, and the related fastcluster/mlpy forks must be installed from source (pip install -e git://github.com/sauliusl/mlpy.git#egg=mlpy)

## Evidence

- [methods] MACS2 peak calling against input control: "Aligned fragments (BAM files) of both epigenomic marks were processed with the MACS2 peak caller [10] to identify regions which showed enrichment relative to a input control sample"
- [methods] Binning coverage into 50 bp windows: "coverage within peak regions are binned into non-overlapping windows spanning 50 bp"
- [methods] Read extension: "Single-end reads are extended to the estimated fragment lengths"
- [methods] Merging called regions: "we then merged the two sets by considering every region called for either mark"
- [methods] Peak length and count filters: "restricted the analysed set of peaks to those that had a length larger than 5 and smaller than 1000 bins"
- [methods] Min-pileup filter: "the unexpressed regions that have no bin with more than 10 reads in it (`-min-pileup` constraint to change) will be ignored"
- [readme] DGW Python package: "This is the repository for the open source DGW library developped by Saulius Lukauskas. Please consult the documentation on https://dgw.readthedocs.org/"
