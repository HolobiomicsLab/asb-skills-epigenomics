---
name: peak-to-gene-promoter-assignment
description: Use when you have a BED file of called peaks and need a non-redundant
  gene list for enrichment analysis, by linking each peak to RefSeq genes with a TSS
  within a promoter window (e.g. 2500 bp).
when_to_use_negative:
- The input is already a gene list or feature table — matchpeaksgenes requires peak
  genomic coordinates, not pre-computed genes
- The task is read mapping or peak calling on raw data (BAM/SAM/FASTQ); this skill
  operates downstream on already-called peak BED files
- Peaks are expected to regulate distal enhancers rather than promoters — a 2500 bp
  TSS-window assignment would miss long-range gene links
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0654
tools:
- name: SeqCode
  role: Provides the matchpeaksgenes application that matches genomic peak coordinates
    to genes based on RefSeq transcript annotations within a promoter window
  repo: https://github.com/eblancoga/seqcode
- name: Enrichr
  role: Consumes the collapsed non-redundant bivalent gene list to perform GO Biological
    Process term enrichment
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-021-98889-7
    title: Productive visualization of high-throughput sequencing data using the SeqCode
      open portable platform
  source_content_hash: sha256:5999e092f4dd14e5b7e2b60e28553b7aa13303037b8ea9ad9d376f882ccc5170
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eblancoga/seqcode
  tools:
  - Enrichr
  - SeqCode
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# peak-to-gene-promoter-assignment

## Summary

Assign ChIP-seq peak coordinates (e.g. bivalent H3K4me3/H3K27me3 overlapping peaks in mESCs) to genes using SeqCode's matchpeaksgenes, which associates each peak to RefSeq genes whose TSS falls within a promoter distance window (2500 bp in the reproduced GO-analysis workflow), enabling downstream gene-list enrichment such as Enrichr GO Biological Process.

## When to use

Use when you have a BED file of called peaks and need a non-redundant gene list for enrichment analysis, by linking each peak to RefSeq genes with a TSS within a promoter window (e.g. 2500 bp).

## When NOT to use

- The input is already a gene list or feature table — matchpeaksgenes requires peak genomic coordinates, not pre-computed genes
- The task is read mapping or peak calling on raw data (BAM/SAM/FASTQ); this skill operates downstream on already-called peak BED files
- Peaks are expected to regulate distal enhancers rather than promoters — a 2500 bp TSS-window assignment would miss long-range gene links

## Inputs

- BED file of called peaks (e.g. overlapping H3K4me3 and H3K27me3 MACS peaks, bivalent peak list from task_001)
- Chromosome size file (chromInfo.txt) for the genome assembly (e.g. mm9/mm10)
- RefSeq gene transcript annotation file (refGene.txt) from the UCSC genome browser
- Promoter distance window parameter (2500 bp from TSS)

## Outputs

- Peak-to-gene association table listing every RefSeq gene with a TSS within 2500 bp of an assigned peak
- Non-redundant list of bivalent genes (gene-list file) suitable for Enrichr GO Biological Process enrichment

## How to apply

Prepare the two genome files SeqCode requires: the chromosome size list (chromInfo.txt) and the RefSeq gene transcript annotation (refGene.txt), both retrievable from the UCSC genome browser for the genome assembly used. Run bin/matchpeaksgenes with the chromosome sizes, refGene.txt, the peak BED file (here the bivalent H3K4me3+/H3K27me3+ overlapping MACS peaks from task_001) and a promoter window of 2500 bp so each peak is assigned to genes whose transcription start site falls within that distance. Collapse the resulting peak-to-gene association table into a non-redundant gene list and write it to a gene-list file. The rationale is that promoter-proximal TSS assignment is the gene-mapping mode Enrichr-compatible gene lists need, and RefSeq annotation is the unit of information SeqCode uses for all gene-based operations. Validate that every gene in the output table lies within 2500 bp of an assigned peak's TSS before submitting the list to Enrichr.

## Related tools

- **SeqCode** (Provides the matchpeaksgenes application that matches genomic peak coordinates to genes based on RefSeq transcript annotations within a promoter window) — https://github.com/eblancoga/seqcode
- **Enrichr** (Consumes the collapsed non-redundant bivalent gene list to perform GO Biological Process term enrichment)

## Examples

```
bin/matchpeaksgenes chromInfo.txt refGene.txt bivalent_peaks.bed genes_out.txt 2500
```

## Evaluation signals

- The gene association table exists, is non-empty, and every associated gene falls within 2500 bp of an assigned peak's TSS
- Collapsed gene list is non-redundant (each RefSeq gene appears once) and is written to a gene-list file
- Enrichr top-ranked GO Biological Process terms are development-related (e.g. embryonic/organ development), reproducing the Fig. 5c result
- Running the shipped test (tests/test_matchpeaksgenes.pl) on the SeqCode build confirms the binary was successfully generated
- All input genes/peaks are consistent with the same genome assembly as the chromInfo.txt and refGene.txt files provided

## Limitations

- Assignment is limited to genes whose TSS falls inside the user-specified promoter window (2500 bp here); distal enhancer-to-gene relationships are not captured
- SeqCode relies on RefSeq reference annotations; gene mapping quality depends on the currency of the refGene.txt assembly release
- Graphical/data outputs of SeqCode-type tools can differ slightly across programs, so assignments may not be identical to those from other peak-to-gene mappers

## Evidence

- [readme] Match genomic coordinates to genes based in any catalog of RefSeq transcript annotations: "Match genomic coordinates to genes based in any catalog of RefSeq transcript annotations"
- [readme] Most programs require the user to provide two files for the particular genome that is being analyzed: the list of chromosomes (and their sizes) and the RefSeq gene transcript annotations.: "Most programs require the user to provide two files for the particular genome that is being analyzed: the list of chromosomes (and their sizes) and the RefSeq gene"
- [methods] SeqCode relies on the RefSeq reference annotations and is able to deal with the genome and assembly release of every organism: "SeqCode relies on the RefSeq reference annotations and is able to deal with the genome and assembly release of every organism that is available from this consortium."
- [discussion] The unit of information in SeqCode is the RefSeq annotation file, which is provided by most genome browsers: "The unit of information in SeqCode is the RefSeq annotation file, which is provided by most genome browsers"
