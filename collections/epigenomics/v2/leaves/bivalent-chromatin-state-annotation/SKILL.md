---
name: bivalent-chromatin-state-annotation
description: Use when two ChIP-seq peak lists (e.g. BED files of H3K4me3 and H3K27me3
  peaks from Beringer et al., GSM2098958 and GSM2098952) must be partitioned into
  mutually exclusive active, bivalent, and silent site classes via overlap.
when_to_use_negative:
- Input consists of raw sequencing reads or BigWig signal tracks rather than already-called
  peak lists — peak calling (MACS) must be done first, and SeqCode defers BigWig generation
  to future versions.
- The two peak sets come from different genome assemblies (mm9 vs mm10) without prior
  coordinate liftover, since overlap comparison requires consistent coordinates.
- The goal is graphical visualization of signal around peaks (meta-plots, heatmaps)
  rather than set-overlap classification of peak coordinates.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0091
tools:
- name: SeqCode
  role: Provides the matchpeaks command that compares the H3K4me3 and H3K27me3 peak
    lists and determines their degree of genomic overlap; installed via `make all`
    into bin/.
  repo: https://github.com/eblancoga/seqcode
- name: MACS
  role: Used upstream to perform the peak calling of the ChIP-seq experiments whose
    peaks are then classified.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-021-98889-7
    title: Productive visualization of high-throughput sequencing data using the SeqCode
      open portable platform
  source_content_hash: sha256:9c640665df97bfac7d73f3c10b456bdd3f61541c216faef72d61e14a5d066efb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eblancoga/seqcode
  tools:
  - MACS
  - SeqCode
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# bivalent-chromatin-state-annotation

## Summary

Classifies mESC H3K4me3 and H3K27me3 ChIP-seq peak sets into active (H3K4me3+/H3K27me3–), bivalent (H3K4me3+/H3K27me3+), and silent (H3K4me3–/H3K27me3+) classes by computing genomic overlap between the two peak lists with SeqCode's matchpeaks command.

## When to use

Use when two ChIP-seq peak lists (e.g. BED files of H3K4me3 and H3K27me3 peaks from Beringer et al., GSM2098958 and GSM2098952) must be partitioned into mutually exclusive active, bivalent, and silent site classes via overlap.

## When NOT to use

- Input consists of raw sequencing reads or BigWig signal tracks rather than already-called peak lists — peak calling (MACS) must be done first, and SeqCode defers BigWig generation to future versions.
- The two peak sets come from different genome assemblies (mm9 vs mm10) without prior coordinate liftover, since overlap comparison requires consistent coordinates.
- The goal is graphical visualization of signal around peaks (meta-plots, heatmaps) rather than set-overlap classification of peak coordinates.

## Inputs

- H3K4me3 mESC ChIP-seq peak list in BED format (GEO GSM2098958, Beringer et al.)
- H3K27me3 mESC ChIP-seq peak list in BED format (GEO GSM2098952, Beringer et al.)
- SeqCode binary suite built via Makefile (bin/ folder)

## Outputs

- bivalent_peak_classes.tsv: one row per peak with chromosome, start, end coordinates and class label (active / bivalent / silent)
- matchpeaks output folder with overlap statistics between the two peak sets

## How to apply

Retrieve the mESC H3K4me3 and H3K27me3 ChIP-seq peak lists in BED format (chromosome, initial position, ending position, optional name); per the article's methods, peaks were called with MACS and raw data obtained from NCBI GEO. Compile SeqCode from source with `make all` to generate binaries in bin/. Run `bin/matchpeaks` to compare the two peak sets and compute their degree of genomic overlap (see tests/test_matchpeaks.pl for a worked example). Partition the union of peaks into the three mutually exclusive classes matching Fig. 5b: H3K4me3-only (active), overlapping in both marks (bivalent), and H3K27me3-only (silent). Write one row per peak with coordinates and class label to bivalent_peak_classes.tsv.

## Related tools

- **SeqCode** (Provides the matchpeaks command that compares the H3K4me3 and H3K27me3 peak lists and determines their degree of genomic overlap; installed via `make all` into bin/.) — https://github.com/eblancoga/seqcode
- **MACS** (Used upstream to perform the peak calling of the ChIP-seq experiments whose peaks are then classified.)

## Examples

```
make all; ./tests/test_matchpeaks.pl -v; bin/matchpeaks <ChromInfo.txt> <H3K4me3_peaks.bed> <H3K27me3_peaks.bed> bivalent_classes
```

## Evaluation signals

- Every peak in the union set receives exactly one class label — the three classes (active, bivalent, silent) are mutually exclusive.
- Peaks assigned 'bivalent' are present in both the H3K4me3 and H3K27me3 BED files; 'active' peaks appear only in H3K4me3; 'silent' peaks only in H3K27me3.
- bivalent_peak_classes.tsv parses as tab-separated rows with valid BED-style coordinates (chromosome, initial position, ending position) and a class label column.
- The distribution of class counts is consistent with the three classes of Fig. 5b of the source article.
- Running tests/test_matchpeaks.pl completes successfully, confirming the matchpeaks binary was correctly generated by the Makefile.

## Limitations

- The three-class scheme only captures H3K4me3/H3K27me3 combinations; other histone marks or combinations are not covered.
- SeqCode commands require genome-specific chromosome lists and RefSeq annotations; peaks must be called and mapped on the same assembly.
- The SeqCode web version restricts input sizes; classification of large peak sets should use the standalone command line, which has no limits on input file size.

## Evidence

- [readme] Compare two sets of peaks or genomic regions to determine the degree of overlap between them: "Compare two sets of peaks or genomic regions to determine the degree of overlap between them"
- [methods] MACS used for peak calling: "MACS12 was used to perform the peak calling of the ChIP-seq experiments."
- [methods] Raw data retrieved from GEO: "Raw data of the examples introduced throughout the article (Table 4) were retrieved from the NCBI GEO repository74."
