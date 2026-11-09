---
name: bivalent-chromatin-mark-interpretation
description: Use when ChIP-seq peak lists for two chromatin marks have been intersected
  (e.g. H3K4me3 and H3K27me3 MACS peaks in mESCs) and you must assign peaks to genes
  and biologically characterize bivalent targets.
when_to_use_negative:
- When peaks come from only a single histone mark rather than an overlap of activating
  and repressive marks — this skill's premise is co-marking (bivalency), not general
  ChIP-seq annotation
- When the goal is genome-wide signal visualization (meta-plots, heatmaps, boxplots)
  rather than peak-to-gene assignment and functional enrichment — SeqCode's produceTSSmaps/producePEAKplots
  tools are the appropriate choice instead
- When the genome assembly has no RefSeq annotation available, since matchpeaksgenes
  relies on RefSeq transcript annotations
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: SeqCode
  role: matchpeaksgenes command assigns each bivalent peak to RefSeq genes whose TSS
    falls within a 2500 bp promoter window, using chromInfo.txt and refGene.txt as
    required genome inputs
  repo: https://github.com/eblancoga/seqcode
- name: Enrichr
  role: GO Biological Process gene-set library used to test the collapsed bivalent
    gene list for developmental enrichment; terms ranked by combined score
- name: MACS
  role: Peak calling of the H3K4me3 and H3K27me3 ChIP-seq experiments producing the
    peak sets whose overlap defines the bivalent input
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-021-98889-7
    title: Productive visualization of high-throughput sequencing data using the SeqCode
      open portable platform
  source_content_hash: sha256:31736d92e97e8db00dda06684017bfae5eae654bb39df29e4bf4ea5b4444d383
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eblancoga/seqcode
  tools:
  - Enrichr
  - MACS
  - SeqCode
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Bivalent chromatin mark interpretation

## Summary

Interprets genomic regions co-marked by activating and repressive histone marks (e.g. H3K4me3+H3K27me3 bivalent peaks in mESCs) by mapping overlapping MACS peaks to nearby genes with SeqCode and testing the resulting gene set for developmental GO enrichment via Enrichr.

## When to use

Use when ChIP-seq peak lists for two chromatin marks have been intersected (e.g. H3K4me3 and H3K27me3 MACS peaks in mESCs) and you must assign peaks to genes and biologically characterize bivalent targets.

## When NOT to use

- When peaks come from only a single histone mark rather than an overlap of activating and repressive marks — this skill's premise is co-marking (bivalency), not general ChIP-seq annotation
- When the goal is genome-wide signal visualization (meta-plots, heatmaps, boxplots) rather than peak-to-gene assignment and functional enrichment — SeqCode's produceTSSmaps/producePEAKplots tools are the appropriate choice instead
- When the genome assembly has no RefSeq annotation available, since matchpeaksgenes relies on RefSeq transcript annotations

## Inputs

- BED file of bivalent peaks (overlap of H3K4me3 and H3K27me3 MACS peaks)
- chromInfo.txt chromosome size file for the mouse genome assembly
- RefSeq refGene.txt transcript annotation from the UCSC genome browser
- Bivalent gene list file (from matchpeaksgenes) for Enrichr submission

## Outputs

- Non-redundant peak-to-gene association table (genes within 2500 bp of peak-adjacent TSS)
- Collapsed bivalent gene-list file
- Enrichr GO Biological Process enrichment results ranked by combined score
- Development-related GO terms (e.g. embryonic/organ development) reproducing the Fig. 5c result

## How to apply

Take the bivalent peak BED file produced by intersecting H3K4me3 and H3K27me3 MACS peak calls, plus the mouse chromInfo.txt chromosome size file and the RefSeq refGene.txt annotation retrieved from the UCSC genome browser. Run SeqCode matchpeaksgenes, assigning each peak to RefSeq genes whose transcription start site falls within a 2500 bp promoter window of the peak. Collapse the peak-to-gene associations into a non-redundant bivalent gene-list file, then submit that list to Enrichr against the GO Biological Process library and rank enriched terms by the Enrichr combined score. The rationale is that bivalency at promoters (H3K4me3 with H3K27me3) is expected at lineage-regulator genes, so the 2500 bp TSS promoter window prioritizes regulatory associations over distal assignments.

## Related tools

- **SeqCode** (matchpeaksgenes command assigns each bivalent peak to RefSeq genes whose TSS falls within a 2500 bp promoter window, using chromInfo.txt and refGene.txt as required genome inputs) — https://github.com/eblancoga/seqcode
- **Enrichr** (GO Biological Process gene-set library used to test the collapsed bivalent gene list for developmental enrichment; terms ranked by combined score)
- **MACS** (Peak calling of the H3K4me3 and H3K27me3 ChIP-seq experiments producing the peak sets whose overlap defines the bivalent input)

## Examples

```
bin/matchpeaksgenes ChromInfo.txt refGene.txt bivalent_peaks.bed 2500 -v
```

## Evaluation signals

- The gene association table exists, is non-empty, and every associated gene falls within 2500 bp of an assigned peak TSS
- The collapsed bivalent gene list contains no duplicate gene entries after collapsing peak-to-gene associations
- Top-ranked Enrichr GO Biological Process terms (by combined score) are development-related, e.g. embryonic/organ development, reproducing the Fig. 5c result
- The input BED file truly contains peaks present in both the H3K4me3 and H3K27me3 MACS peak sets (bivalent overlap)

## Limitations

- matchpeaksgenes assigns peaks only via proximity to RefSeq gene TSSs within the chosen window; distal enhancer-like bivalent sites not near a TSS are not captured
- GO enrichment via Enrichr is a gene-set over-representation test — it reflects the biology of associated genes, not direct evidence that the bivalent mark regulates them
- The bivalent result depends on the quality and overlap thresholds of the upstream MACS peak calls for both marks

## Evidence

- [other] matchpeaksgenes assigns peaks to genes within a defined promoter window: "Match genomic coordinates to genes based in any catalog of RefSeq transcript annotations"
- [readme] matchpeaksgenes requires chromosome sizes and RefSeq annotations: "Most programs require the user to provide two files for the particular genome that is being analyzed: the list of chromosomes (and their sizes) and the RefSeq gene transcript annotations."
- [methods] SeqCode depends on RefSeq annotations for any organism release: "SeqCode relies on the RefSeq reference annotations and is able to deal with the genome and assembly release of every organism that is available from this consortium."
- [readme] a test invocation exists for matchpeaksgenes in the SeqCode test suite: "test_matchpeaksgenes.pl"
- [other] the task card describes the validation criterion: [evidence span withheld — anchor status: not_found]
