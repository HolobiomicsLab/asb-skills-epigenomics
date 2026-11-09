---
name: go-term-enrichment-analysis
description: Use when you have a BED list of bivalent (H3K4me3+/H3K27me3+) peaks in
  mESCs and need to assign them to nearby genes and test whether the resulting gene
  list is enriched for development-related GO terms.
when_to_use_negative:
- Input is already a curated gene list from a different gene-assignment rule — the
  2500 bp TSS-window definition would no longer be the origin of the list
- The peaks of interest are unimodal (single H3K4me3 or H3K27me3 marks) rather than
  overlapping bivalent peaks
- The genome annotation is not RefSeq-based (e.g. Gencode-only pipelines), since SeqCode
  relies on RefSeq reference annotations
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SeqCode
  role: matchpeaksgenes command assigns each bivalent peak to RefSeq genes whose TSS
    lies within the 2500 bp promoter window
  repo: https://github.com/eblancoga/seqcode
- name: Enrichr
  role: web service used for GO Biological Process term enrichment of the bivalent
    gene list; enriched terms are ranked by combined score
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-021-98889-7
    title: Productive visualization of high-throughput sequencing data using the SeqCode
      open portable platform
  source_content_hash: sha256:ea0ccda019db8cf8e3cc88f2830e7394f0a7928ce98e59d6ccb9532fd6750c42
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eblancoga/seqcode
  tools:
  - Enrichr
  - SeqCode
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# go-term-enrichment-analysis

## Summary

Links ChIP-seq-derived bivalent peak sets (overlapping H3K4me3 and H3K27me3 MACS peaks) to genes via SeqCode matchpeaksgenes and tests them for functional enrichment of developmental GO Biological Process terms in Enrichr, reproducing the Fig. 5c result of the mESC bivalency study.

## When to use

Use when you have a BED list of bivalent (H3K4me3+/H3K27me3+) peaks in mESCs and need to assign them to nearby genes and test whether the resulting gene list is enriched for development-related GO terms.

## When NOT to use

- Input is already a curated gene list from a different gene-assignment rule — the 2500 bp TSS-window definition would no longer be the origin of the list
- The peaks of interest are unimodal (single H3K4me3 or H3K27me3 marks) rather than overlapping bivalent peaks
- The genome annotation is not RefSeq-based (e.g. Gencode-only pipelines), since SeqCode relies on RefSeq reference annotations

## Inputs

- BED file of bivalent peaks (overlapping H3K4me3 and H3K27me3 MACS peak calls)
- ChromInfo.txt chromosome size file for the mouse genome assembly
- RefSeq refGene.txt gene transcript annotation from the UCSC genome browser
- Promoter window parameter (2500 bp around the TSS)

## Outputs

- Peak-to-gene association table (each gene within 2500 bp of an assigned peak TSS)
- Non-redundant bivalent gene-list file
- Ranked Enrichr GO Biological Process enrichment table ordered by combined score
- Set of top development-related GO terms (Fig. 5c result)

## How to apply

Load the bivalent peak BED file (overlapping H3K4me3 and H3K27me3 MACS peaks from task_001) together with the mouse chromInfo.txt chromosome-size file and the RefSeq refGene.txt annotation retrieved from the UCSC genome browser. Run SeqCode matchpeaksgenes, assigning each peak to RefSeq genes whose TSS falls within a 2500 bp promoter window of the peak — the unit of information SeqCode operates on is the RefSeq annotation file. Collapse the peak-to-gene associations into a non-redundant gene list and write it to a gene-list file. Submit the list to Enrichr against the GO Biological Process gene-set library, rank the enriched terms by Enrichr combined score, and inspect the top categories for developmental functions (e.g. embryonic/organ development).

## Related tools

- **SeqCode** (matchpeaksgenes command assigns each bivalent peak to RefSeq genes whose TSS lies within the 2500 bp promoter window) — https://github.com/eblancoga/seqcode
- **Enrichr** (web service used for GO Biological Process term enrichment of the bivalent gene list; enriched terms are ranked by combined score)

## Examples

```
bin/matchpeaksgenes ChromInfo.txt refGene.txt bivalent_peaks.bed bivalent_genes 2500  # then submit bivalent_genes.txt to Enrichr (GO Biological Process) and rank by combined score
```

## Evaluation signals

- The gene association table produced by matchpeaksgenes exists and is non-empty
- Every gene in the association table falls within 2500 bp of the TSS of an assigned peak
- The collapsed gene list is non-redundant (one entry per gene)
- The top-ranked GO Biological Process terms by Enrichr combined score are development-related (e.g. embryonic/organ development), reproducing the Fig. 5c result (expert_review)

## Limitations

- The biological interpretation depends on the promoter window choice (2500 bp of TSS via matchpeaksgenes); a different distance rule would yield a different gene set and enrichment result
- SeqCode requires the user to supply the chromosome-size file and RefSeq refGene.txt for the specific genome assembly; mis-matched assembly versions will invalidate gene association
- Enrichment is tested only against the GO Biological Process library in this workflow; other ontologies or gene-set libraries would need to be run separately

## Evidence

- [other] Run SeqCode matchpeaksgenes on the bivalent peak list, assigning each peak to RefSeq genes whose transcription start site falls within a 2500 bp promoter window: [evidence span withheld — anchor status: not_found]
- [other] Submit the bivalent gene list to Enrichr against the GO Biological Process gene-set library.: [evidence span withheld — anchor status: not_found]
- [other] Gene ontology analyses confirmed that bivalent genes are related to development, using the target genes obtained with the matchpeaksgenes rule (peak within 2500 bp of the TSS) and Enrichr: [evidence span withheld — anchor status: not_found]
- [discussion] The unit of information in SeqCode is the RefSeq annotation file, which is provided by most genome browsers: "The unit of information in SeqCode is the RefSeq annotation file, which is provided by most genome browsers"
- [readme] Match genomic coordinates to genes based in any catalog of RefSeq transcript annotations: "Match genomic coordinates to genes based in any catalog of RefSeq transcript annotations"
