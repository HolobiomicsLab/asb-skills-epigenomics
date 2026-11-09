---
name: histone-modification-peak-set-processing
description: Use when you have two BED-format ChIP-seq peak lists (chromosome, start,
  end) for different histone marks and need to partition their union into mark-specific
  and overlapping classes (e.g. active/bivalent/silent promoters).
when_to_use_negative:
- Input is raw sequencing reads rather than called peak lists — peak calling (e.g.
  MACS) must be performed first
- The goal is signal visualization (meta-plots, heatmaps, TSS maps) rather than overlap
  classification of two peak sets
- Only one histone mark or ChIP-seq peak set is available, since class assignment
  requires comparing two sets
edam_operation: http://edamontology.org/operation_0282
tools:
- name: SeqCode
  role: Provides the `bin/matchpeaks` command used to compare the H3K4me3 and H3K27me3
    BED peak lists and compute their degree of genomic overlap for classification
  repo: https://github.com/eblancoga/seqcode
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-021-98889-7
    title: Productive visualization of high-throughput sequencing data using the SeqCode
      open portable platform
  source_content_hash: sha256:9ba0a841ac8ceb4e31ac3cdd5427bdf89dbe554875009a1cb307c98ebc8d6cda
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eblancoga/seqcode
  tools:
  - SeqCode
  edam_operation: http://edamontology.org/operation_0282
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# histone-modification-peak-set-processing

## Summary

Classifies ChIP-seq peak sets of two histone marks into mutually exclusive overlap classes using SeqCode's matchpeaks command: for mESC H3K4me3 and H3K27me3 peaks this yields active (H3K4me3+/H3K27me3–), bivalent (H3K4me3+/H3K27me3+), and silent (H3K4me3–/H3K27me3+) classes.

## When to use

Use when you have two BED-format ChIP-seq peak lists (chromosome, start, end) for different histone marks and need to partition their union into mark-specific and overlapping classes (e.g. active/bivalent/silent promoters).

## When NOT to use

- Input is raw sequencing reads rather than called peak lists — peak calling (e.g. MACS) must be performed first
- The goal is signal visualization (meta-plots, heatmaps, TSS maps) rather than overlap classification of two peak sets
- Only one histone mark or ChIP-seq peak set is available, since class assignment requires comparing two sets

## Inputs

- BED-format peak list for mark 1 (e.g. H3K4me3 peaks from mESC, GEO GSM2098958): chromosome, initial position, ending position, optional name
- BED-format peak list for mark 2 (e.g. H3K27me3 peaks from mESC, GEO GSM2098952): chromosome, initial position, ending position, optional name

## Outputs

- bivalent_peak_classes.tsv with one row per peak: genomic coordinates and class label (active / bivalent / silent)

## How to apply

Retrieve the two ChIP-seq peak sets (e.g. mESC H3K4me3 and H3K27me3 from Beringer et al., GEO GSM2098958 and GSM2098952) as BED files with columns chromosome, initial position, ending position, and optional name. Run SeqCode's `bin/matchpeaks` to compare the two peak lists and compute their degree of genomic overlap, following the README workflow in which SeqCode 'Compare[s] two sets of peaks or genomic regions to determine the degree of overlap between them'. Partition the union of peaks into three mutually exclusive classes as in Fig. 5b: H3K4me3-only (active), peaks present in both marks (bivalent), and H3K27me3-only (silent). Decision point: peaks are assigned to exactly one class, so any overlap rule (e.g. any base-pair intersection) must be fixed before partitioning. Write one row per peak with its coordinates and class label to the output TSV (bivalent_peak_classes.tsv). Correctness is confirmed by the classes being non-overlapping and jointly covering the union of both input peak lists.

## Related tools

- **SeqCode** (Provides the `bin/matchpeaks` command used to compare the H3K4me3 and H3K27me3 BED peak lists and compute their degree of genomic overlap for classification) — https://github.com/eblancoga/seqcode

## Examples

```
./tests/test_matchpeaks.pl -v
```

## Evaluation signals

- The three classes (active, bivalent, silent) are mutually exclusive: no peak appears in more than one class
- The union of the three classes covers every peak in both input BED files (no peaks dropped)
- Each output row in bivalent_peak_classes.tsv carries valid coordinates (chromosome, initial position, ending position) plus exactly one class label
- Class assignment matches the matchpeaks overlap output: H3K4me3-only → active, overlap in both marks → bivalent, H3K27me3-only → silent
- Running the SeqCode self-test `./tests/test_matchpeaks.pl` terminates successfully, confirming the binary is correctly built via `make all`

## Limitations

- The SeqCode README test suite must run successfully after `make all` before trusting binaries; pthread warnings during SAMtools compilation can be safely skipped
- The workflow as described classifies peaks by set membership/overlap only; it does not quantify signal intensity within classes
- matchpeaks overlap semantics (any-base intersection vs. fractional overlap) must be fixed by the user; the source cards specify 'degree of genomic overlap' without a numeric cutoff

## Evidence

- [readme] matchpeaks overlap role: "Compare two sets of peaks or genomic regions to determine the degree of overlap between them"
- [results] peak lists as matchpeaks input: [evidence span withheld — anchor status: not_found]
- [readme] matchpeaks test availability: "The tests/ folder contain a series of Perl scripts to test the correct functioning of each SeqCode command:"
- [results] three-class partition from peak overlap: [evidence span withheld — anchor status: not_found]
