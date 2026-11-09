---
name: genomic-interval-overlap-comparison
description: Use when you have two BED-format peak lists for different histone marks
  or features and need to partition their union into overlapping vs. non-overlapping
  classes (e.g. H3K4me3+/H3K27me3– bivalent analysis).
when_to_use_negative:
- Input is raw sequencing reads (FASTQ/BAM) rather than already-called peak lists
  — peak calling (e.g. with MACS) must happen first; matchpeaks compares peaks, not
  reads
- You need three or more sets intersected simultaneously or set-level summary graphics
  rather than pairwise interval matching — the SeqCode website's Venn/UpSet chart
  applications are the better fit per the README
- You need genome-wide bigWig profile generation — the article states bigWig output
  'should be considered in future versions' and is not supported
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: SeqCode
  role: Provides the `matchpeaks` command-line application that compares two sets
    of peaks/regions and computes their degree of genomic overlap, yielding the active/bivalent/silent
    partition
  repo: https://github.com/eblancoga/seqcode
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-021-98889-7
    title: Productive visualization of high-throughput sequencing data using the SeqCode
      open portable platform
  source_content_hash: sha256:bda4a092d4b50ba1122bb362204214d65cc06afe94b7ed790732e76a66f1c9aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eblancoga/seqcode
  tools:
  - SeqCode
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# genomic-interval-overlap-comparison

## Summary

Classify two sets of ChIP-seq peaks/regions (e.g. mESC H3K4me3 and H3K27me3 peaks from Beringer et al., GEO GSM2098958 and GSM2098952) into mutually exclusive overlap classes (active, bivalent, silent) using the SeqCode `matchpeaks` command to compute their degree of genomic overlap.

## When to use

Use when you have two BED-format peak lists for different histone marks or features and need to partition their union into overlapping vs. non-overlapping classes (e.g. H3K4me3+/H3K27me3– bivalent analysis).

## When NOT to use

- Input is raw sequencing reads (FASTQ/BAM) rather than already-called peak lists — peak calling (e.g. with MACS) must happen first; matchpeaks compares peaks, not reads
- You need three or more sets intersected simultaneously or set-level summary graphics rather than pairwise interval matching — the SeqCode website's Venn/UpSet chart applications are the better fit per the README
- You need genome-wide bigWig profile generation — the article states bigWig output 'should be considered in future versions' and is not supported

## Inputs

- H3K4me3 ChIP-seq peak list in BED format (chromosome, initial position, ending position, optional name), e.g. GSM2098958 mESC peaks
- H3K27me3 ChIP-seq peak list in BED format (chromosome, initial position, ending position, optional name), e.g. GSM2098952 mESC peaks

## Outputs

- Overlap/partition results from matchpeaks: assignment of each peak to active (H3K4me3+/H3K27me3–), bivalent (H3K4me3+/H3K27me3+), or silent (H3K4me3–/H3K27me3+) classes
- Tabular file of one row per peak with coordinates and class label, e.g. bivalent_peak_classes.tsv

## How to apply

Retrieve the two peak sets as BED files (chromosome, initial position, ending position, optional name); in the reference case these are the mESC H3K4me3 (GSM2098958) and H3K27me3 (GSM2098952) peaks from Beringer et al. Run the SeqCode command `bin/matchpeaks`, which is designed to compare two sets of peaks or genomic regions and determine the degree of overlap between them; build SeqCode binaries first with `make all` and use `-h`/`-v` to inspect options and progress. Partition the union of peaks into the three mutually exclusive classes of Fig. 5b: H3K4me3-only (active), overlapping in both marks (bivalent), and H3K27me3-only (silent). Rationale: matching on genomic coordinates is a pure interval-arithmetic step that needs the peak lists only — no read-level reprocessing — and SeqCode's modular command-line architecture outputs the overlap results into a results folder suitable for pipeline integration. Write one row per peak with its coordinates and class label (e.g. bivalent_peak_classes.tsv).

## Related tools

- **SeqCode** (Provides the `matchpeaks` command-line application that compares two sets of peaks/regions and computes their degree of genomic overlap, yielding the active/bivalent/silent partition) — https://github.com/eblancoga/seqcode

## Examples

```
./tests/test_matchpeaks.pl -v
```

## Evaluation signals

- The union of peaks partitions into exactly three mutually exclusive classes matching Fig. 5b: active (H3K4me3-only), bivalent (both marks), and silent (H3K27me3-only); every peak appears in exactly one class
- Overlapping intervals share chromosome and coordinate range in both input BED files, and the reported degree of overlap from matchpeaks is consistent with interval arithmetic on the input coordinates
- The Perl self-test `tests/test_matchpeaks.pl` completes with successful termination, confirming the binaries were correctly generated via `make all`
- Output rows carry the original BED fields (chromosome, initial position, ending position) plus a class label, so coordinates can be traced back to the source GEO peak lists (GSM2098958/GSM2098952)

## Limitations

- matchpeaks is a pairwise comparison; the three-class active/bivalent/silent partition must be assembled by the user from the pairwise overlap output
- Graphical outputs depend on R being available, since SeqCode images are 'internally generated using R'
- SeqCode commands generally require chromosome size and RefSeq annotation files for the genome being analyzed, though pure peak-to-peak matching needs the BED lists themselves
- The web-tool mode is 'intended to be a teaching resource' with file-size restrictions; large peak sets should be run via the standalone command line

## Evidence

- [readme] matchpeaks use case: "Compare two sets of peaks or genomic regions to determine the degree of overlap between them"
- [results] three-class partition: [evidence span withheld — anchor status: not_found]
- [methods] BED input format: [evidence span withheld — anchor status: not_found]
- [readme] self-test availability: "tests/:
Perl scripts to test the SeqCode binaries were successfully generated with real examples."
