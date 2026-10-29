---
name: chipseq-peak-overlap-classification
description: Use when two BED-format peak lists from ChIP-seq experiments (e.g. H3K4me3
  and H3K27me3 in mESC) must be partitioned into overlap-based classes such as active,
  bivalent, and silent regulatory states.
when_to_use_negative:
- Input is still raw sequencing data (FASTQ/BAM) — peaks must first be called (e.g.
  with MACS) before matchpeaks can compare them
- The two peak sets are already annotated as a feature table without genomic coordinates
  — matchpeaks operates on BED coordinate lists
- You need genome-wide signal profiles in bigWig format — the article notes this is
  deferred to future versions of SeqCode
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: SeqCode
  role: matchpeaks command compares the two peak lists and computes their degree of
    genomic overlap; genomeDistribution annotates resulting classes against RefSeq
    features
  repo: https://github.com/eblancoga/seqcode
- name: MACS
  role: upstream peak caller that produces the BED peak lists fed into the overlap
    classification
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41598-021-98889-7
    title: Productive visualization of high-throughput sequencing data using the SeqCode
      open portable platform
  source_content_hash: sha256:3dad50f852bce6b76a71699f277e4a6b89c1f2b88a7c1ef82375d86fb3b92dd4
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# chipseq-peak-overlap-classification

## Summary

Classify ChIP-seq peaks from two histone marks into mutually exclusive overlap classes (e.g. active H3K4me3-only, bivalent H3K4me3+/H3K27me3+, silent H3K27me3-only) using the SeqCode matchpeaks command, as demonstrated for mESC H3K4me3/H3K27me3 peaks from Beringer et al.

## When to use

Use when two BED-format peak lists from ChIP-seq experiments (e.g. H3K4me3 and H3K27me3 in mESC) must be partitioned into overlap-based classes such as active, bivalent, and silent regulatory states.

## When NOT to use

- Input is still raw sequencing data (FASTQ/BAM) — peaks must first be called (e.g. with MACS) before matchpeaks can compare them
- The two peak sets are already annotated as a feature table without genomic coordinates — matchpeaks operates on BED coordinate lists
- You need genome-wide signal profiles in bigWig format — the article notes this is deferred to future versions of SeqCode

## Inputs

- BED-format peak list for mark 1 (e.g. H3K4me3 mESC peaks: chromosome, initial position, ending position, optional name)
- BED-format peak list for mark 2 (e.g. H3K27me3 mESC peaks in the same assembly)
- Chromosome size file (chromInfo.txt) for the reference assembly (e.g. mm9 from UCSC)
- RefSeq gene transcript annotation (refGene.txt) for genomic-feature annotation of classes

## Outputs

- One row per peak with coordinates and class label (active / bivalent / silent), e.g. bivalent_peak_classes.tsv
- Per-class peak lists (H3K4me3-only, overlapping, H3K27me3-only) usable as genomeDistribution inputs
- Optional genomic-distribution pie charts produced by the R scripts emitted by SeqCode genomeDistribution

## How to apply

Retrieve the two ChIP-seq peak sets (e.g. GEO GSM2098958 and GSM2098952 from Beringer et al.) as BED files (chromosome, initial position, ending position, optional name), with peaks produced by MACS peak calling as in the SeqCode paper. Run the SeqCode application `bin/matchpeaks`, which compares two sets of peaks or genomic regions to determine their degree of genomic overlap. Partition the union of peaks into three mutually exclusive classes: H3K4me3-only (active), overlapping in both marks (bivalent), and H3K27me3-only (silent), mirroring Fig. 5b of the paper. Optionally annotate each class with SeqCode genomeDistribution using chromInfo.txt and refGene.txt for the mouse mm9 assembly to confirm bivalent peaks are promoter-enriched. Write one row per peak with coordinates and class label (e.g. bivalent_peak_classes.tsv) for downstream use.

## Related tools

- **SeqCode** (matchpeaks command compares the two peak lists and computes their degree of genomic overlap; genomeDistribution annotates resulting classes against RefSeq features) — https://github.com/eblancoga/seqcode
- **MACS** (upstream peak caller that produces the BED peak lists fed into the overlap classification)

## Examples

```
./tests/test_matchpeaks.pl -v
```

## Evaluation signals

- The three output classes are mutually exclusive and exhaustive over the union of input peaks (no peak appears in two classes)
- Class assignment matches mark status per peak: active = H3K4me3+/H3K27me3–, bivalent = H3K4me3+/H3K27me3+, silent = H3K4me3–/H3K27me3+
- genomeDistribution on the bivalent class shows preferential promoter enrichment with only a minor intergenic fraction, reproducing the Fig. 5b pattern
- SeqCode built successfully via `make all` and the bundled test `./tests/test_matchpeaks.pl` completes with [DONE] termination
- Output TSV rows retain valid BED coordinates (chromosome, initial position, ending position) consistent with the input peak lists

## Limitations

- Most SeqCode programs require genome-specific files — the list of chromosomes (and their sizes) and the RefSeq gene transcript annotations — so classification is assembly-dependent (mm9/mm10 in the article)
- Peak-calling quality upstream (MACS) directly determines class membership; matchpeaks does not rescore raw signal
- No formal changelog is provided for the software, so version-dependent behavior of matchpeaks is hard to track

## Evidence

- [readme] Compare two sets of peaks or genomic regions to determine the degree of overlap between them: "Compare two sets of peaks or genomic regions to determine the degree of overlap between them"
- [methods] MACS12 was used to perform the peak calling of the ChIP-seq experiments.: "MACS12 was used to perform the peak calling of the ChIP-seq experiments."
- [readme] Most programs require the user to provide two files for the particular genome that is being analyzed: the list of chromosomes (and their sizes) and the RefSeq gene transcript annotations.: "Most programs require the user to provide two files for the particular genome that is being analyzed: the list of chromosomes (and their sizes) and the RefSeq gene transcript annotations."
- [readme] The tests/ folder contain a series of Perl scripts to test the correct functioning of each SeqCode command: "The tests/ folder contain a series of Perl scripts to test the correct functioning of each SeqCode command"
