---
name: gene-set-enrichment-interpretation
description: Use when you have a BED list of ChIP-seq peaks (e.g. bivalent H3K4me3+/H3K27me3+
  peaks in mESCs) and need to identify enriched biological functions such as developmental
  GO categories.
when_to_use_negative:
- Input is already a gene list with no peak coordinates — the matchpeaksgenes peak-to-TSS
  mapping step is unnecessary
- The genome/organism has no RefSeq annotation available from UCSC or the consortium,
  since SeqCode relies on RefSeq reference annotations
- Peak-to-gene assignment should use gene-body or other distance rules rather than
  a 2500 bp promoter window around the TSS
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SeqCode
  role: matchpeaksgenes assigns each peak to RefSeq genes whose TSS falls within the
    2500 bp promoter window, producing the gene list
  repo: https://github.com/eblancoga/seqcode
- name: Enrichr
  role: performs GO Biological Process enrichment on the bivalent gene list and ranks
    terms by combined score
- name: MACS
  role: produces the ChIP-seq peaks (H3K4me3, H3K27me3) that serve as the input regions
    for gene matching
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-021-98889-7
    title: Productive visualization of high-throughput sequencing data using the SeqCode
      open portable platform
  source_content_hash: sha256:082132dc23baef16ec40e963d2c7aae8f0bb9880d20733a33f81fe45e4b31d57
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eblancoga/seqcode
  tools:
  - Enrichr
  - MACS
  - SeqCode
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# gene-set-enrichment-interpretation

## Summary

Derive functional interpretation from a set of genomic regions by first mapping them to genes with SeqCode matchpeaksgenes (RefSeq annotations, 2500 bp TSS window), then running GO Biological Process enrichment with Enrichr and ranking terms by combined score.

## When to use

Use when you have a BED list of ChIP-seq peaks (e.g. bivalent H3K4me3+/H3K27me3+ peaks in mESCs) and need to identify enriched biological functions such as developmental GO categories.

## When NOT to use

- Input is already a gene list with no peak coordinates — the matchpeaksgenes peak-to-TSS mapping step is unnecessary
- The genome/organism has no RefSeq annotation available from UCSC or the consortium, since SeqCode relies on RefSeq reference annotations
- Peak-to-gene assignment should use gene-body or other distance rules rather than a 2500 bp promoter window around the TSS

## Inputs

- BED file of genomic peaks (e.g. overlapping H3K4me3 and H3K27me3 MACS peaks)
- Chromosome sizes file (chromInfo.txt)
- RefSeq gene annotation file (refGene.txt) from the UCSC genome browser
- Promoter window parameter (2500 bp around the TSS)

## Outputs

- Non-redundant list of target genes (gene-list file) associated to peaks within 2500 bp of the TSS
- Ranked GO Biological Process enrichment table with Enrichr combined scores

## How to apply

Load the peak BED file together with the chromosome size file (ChromInfo.txt) and the RefSeq refGene.txt annotation retrieved from the UCSC genome browser. Run SeqCode matchpeaksgenes assigning each peak to RefSeq genes whose transcription start site falls within a 2500 bp promoter window of the peak. Collapse the peak-to-gene associations into a non-redundant gene list and write it to a gene-list file. Submit that gene list to Enrichr against the GO Biological Process gene-set library and rank enriched terms by Enrichr combined score, inspecting the top categories (e.g. embryonic/organ development for bivalent genes, reproducing the Fig. 5c result).

## Related tools

- **SeqCode** (matchpeaksgenes assigns each peak to RefSeq genes whose TSS falls within the 2500 bp promoter window, producing the gene list) — https://github.com/eblancoga/seqcode
- **Enrichr** (performs GO Biological Process enrichment on the bivalent gene list and ranks terms by combined score)
- **MACS** (produces the ChIP-seq peaks (H3K4me3, H3K27me3) that serve as the input regions for gene matching)

## Examples

```
./tests/test_matchpeaksgenes.pl -v
```

## Evaluation signals

- The peak-to-gene association table exists, is non-empty, and every associated gene falls within 2500 bp of an assigned peak's TSS
- The collapsed gene list is non-redundant (no duplicate genes)
- Top-ranked GO Biological Process terms by Enrichr combined score are development-related (e.g. embryonic/organ development), reproducing the Fig. 5c result
- The gene-list file is in a format accepted by Enrichr for the GO Biological Process library submission

## Limitations

- Results depend on the promoter window choice (2500 bp); a different distance rule yields a different gene set and hence different enrichment terms
- SeqCode requires the user to supply genome-specific chromosome size and refGene.txt files for the analyzed assembly
- The web server version of SeqCode is limited in input size; the standalone command line has no such limits on the size of input files
- Enrichment interpretation is restricted to the gene-set library chosen (here GO Biological Process); other libraries would give other term rankings

## Evidence

- [readme] Match genomic coordinates to genes based in any catalog of RefSeq transcript annotations: "Match genomic coordinates to genes based in any catalog of RefSeq transcript annotations"
- [discussion] RefSeq annotation is SeqCode's information unit: "The unit of information in SeqCode is the RefSeq annotation file, which is provided by most genome browsers"
- [methods] SeqCode relies on RefSeq annotations: "SeqCode relies on the RefSeq reference annotations and is able to deal with the genome and assembly release of every organism"
- [readme] test scripts confirm binaries including matchpeaksgenes: "The tests/ folder contain a series of Perl scripts to test the correct functioning of each SeqCode command"
- [readme] Most programs require chromosome list and RefSeq annotations from UCSC: "Most programs require the user to provide two files for the particular genome that is being analyzed: the list of chromosomes (and their sizes) and the RefSeq gene transcript annotations."
