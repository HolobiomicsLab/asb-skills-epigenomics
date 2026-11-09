---
name: refseq-transcript-annotation-matching
description: Use when you have BED-formatted peak coordinates (e.g. MACS peaks) and
  need to map them to gene identifiers via RefSeq transcript annotations, e.g. to
  build target-gene lists for pathway or GO enrichment.
when_to_use_negative:
- Input is already a gene list or expression matrix — gene matching operates on genomic
  coordinates and requires interval inputs
- Peak lists are in a raw SAM/BAM format rather than BED coordinates; use SeqCode
  peak/processing commands first
- Your annotation is not RefSeq-based (e.g. you need GENCODE/Ensembl transcript models);
  SeqCode relies on RefSeq annotations
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: SeqCode
  role: Provides the matchpeaksgenes command that matches BED peak coordinates to
    RefSeq genes within a user-defined promoter window (2500 bp of the TSS in the
    bivalent-gene workflow); requires chromInfo and refGene inputs
  repo: https://github.com/eblancoga/seqcode
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1038/s41598-021-98889-7
    title: Productive visualization of high-throughput sequencing data using the SeqCode
      open portable platform
  source_content_hash: sha256:b6037f379b98fcd6035d5b94953debd4f544476a82eb29b8384f5ade5cc0feb2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eblancoga/seqcode
  tools:
  - Enrichr
  - SeqCode
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# refseq-transcript-annotation-matching

## Summary

Assign ChIP-seq or other genomic intervals to RefSeq genes using the SeqCode matchpeaksgenes command, which pairs a chromosome-size file and a UCSC RefSeq refGene.txt annotation with a BED peak list to associate each peak with nearby genes. In the bivalent-chromatin example this links overlapping H3K4me3/H3K27me3 MACS peaks to genes within a 2500 bp promoter window of the TSS, feeding downstream GO enrichment.

## When to use

Use when you have BED-formatted peak coordinates (e.g. MACS peaks) and need to map them to gene identifiers via RefSeq transcript annotations, e.g. to build target-gene lists for pathway or GO enrichment.

## When NOT to use

- Input is already a gene list or expression matrix — gene matching operates on genomic coordinates and requires interval inputs
- Peak lists are in a raw SAM/BAM format rather than BED coordinates; use SeqCode peak/processing commands first
- Your annotation is not RefSeq-based (e.g. you need GENCODE/Ensembl transcript models); SeqCode relies on RefSeq annotations

## Inputs

- BED-formatted list of genomic intervals/peaks (e.g. MACS-called H3K4me3/H3K27me3 bivalent peaks)
- Chromosome size file (chromInfo.txt) for the genome assembly (e.g. mouse mm9 from UCSC)
- RefSeq gene transcript annotation file (refGene.txt) for the same assembly from the UCSC genome browser
- Promoter window parameter (e.g. 2500 bp around the TSS)

## Outputs

- Peak-to-gene association table (each peak with assigned RefSeq gene(s) within the promoter window)
- Collapsed non-redundant gene list of matched target genes (e.g. bivalent gene list for Enrichr submission)

## How to apply

Download two per-genome files from the UCSC genome browser: the chromosome size list (chromInfo.txt) and the RefSeq gene transcript annotation (refGene.txt) for the assembly being analyzed (e.g. mm9). Run SeqCode matchpeaksgenes with the chromInfo file, the refGene file, and the BED peak list, specifying the promoter window relative to the transcription start site — the source workflow uses 2500 bp around the TSS for bivalent H3K4me3/H3K27me3 peaks in mESCs. The command emits a peak-to-gene association table in an output folder; collapse these associations into a non-redundant list of target genes. Rationale: SeqCode's unit of information is the RefSeq annotation file, so all genomic-feature operations (including gene matching) are anchored to RefSeq transcript models, which the tool supports for every organism and assembly release available from the consortium. Sanity-check the results with the tests/test_matchpeaksgenes.pl script before running on production data.

## Related tools

- **SeqCode** (Provides the matchpeaksgenes command that matches BED peak coordinates to RefSeq genes within a user-defined promoter window (2500 bp of the TSS in the bivalent-gene workflow); requires chromInfo and refGene inputs) — https://github.com/eblancoga/seqcode

## Examples

```
./tests/test_matchpeaksgenes.pl -v
```

## Evaluation signals

- The gene association output file exists, is non-empty, and every associated gene's TSS falls within the specified promoter window (2500 bp in the source workflow) of an assigned peak
- All matched genes are RefSeq gene identifiers consistent with the refGene.txt assembly used (e.g. mm9 gene symbols)
- The tests/test_matchpeaksgenes.pl script completes successfully on the bundled test inputs before applying to production peaks
- Downstream check: submitting the collapsed gene list to Enrichr against GO Biological Process yields development-related top terms for the bivalent peak set, as in the source task
- Collapsing peak-to-gene associations produces a non-redundant gene list (no duplicate gene entries)

## Limitations

- SeqCode relies on RefSeq reference annotations, so matching quality depends on the completeness of the refGene.txt file retrieved from the UCSC genome browser for the chosen assembly
- Both the chromosome-size file and the refGene annotation must correspond to the same genome assembly as the peak coordinates (e.g. mm9 peaks with mm9 refGene); mismatched assemblies will produce wrong assignments
- The promoter window is a user-defined threshold (2500 bp around the TSS in the source workflow); different windows change which genes are matched

## Evidence

- [readme] Match genomic coordinates to genes based in any catalog of RefSeq transcript annotations: "Match genomic coordinates to genes based in any catalog of RefSeq transcript annotations"
- [readme] Most programs require chromInfo and refGene files: "Most programs require the user to provide two files for the particular genome that is being analyzed: the list of chromosomes (and their sizes) and the RefSeq gene transcript annotations."
- [discussion] RefSeq annotations from genome browsers: "The unit of information in SeqCode is the RefSeq annotation file, which is provided by most genome browsers"
- [methods] SeqCode relies on RefSeq annotations: "SeqCode relies on the RefSeq reference annotations and is able to deal with the genome and assembly release of every organism that is available from this consortium."
