---
name: genomic-interval-extraction-from-gene-annotation
description: Use when you need to obtain concrete genomic coordinates from a gene
  annotation table (UCSC knownGenes track) to define fixed-length regions — e.g. 2
  kb windows spanning TSS and first splicing site — on which ChIP-Seq enrichment profiles
  will be extracted.
when_to_use_negative:
- When intervals are already provided as a BED/peak file from a peak caller such as
  MACS2 — no gene-annotation extraction is needed
- When the analysis requires whole-genome tiling rather than gene-anchored 2 kb windows
- When the regions of interest lack a splicing site within the window and must instead
  be defined purely by enrichment peaks
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0797
tools:
- name: UCSC genome browser
  role: Table browser source of the knownGenes annotation from which genomic intervals
    (first 2 kb of gene bodies) are extracted
- name: DGW
  role: 'Downstream consumer of the extracted intervals: H3K4me3 profiles over these
    seed regions are aligned and clustered'
  repo: https://github.com/lukauskas/dgw
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-016-1306-0
    title: 'DGW: an exploratory data analysis tool for clustering and visualisation
      of epigenomic marks'
  source_content_hash: sha256:3ba74e9dd3190999d7a64f2d1c4007b6ec3f54839d7f97abab17df1b016208ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lukauskas/dgw
  tools:
  - DGW
  - Python
  - UCSC genome browser
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# genomic-interval-extraction-from-gene-annotation

## Summary

Extracts defined genomic intervals (e.g. the first 2 kb of gene bodies) from the UCSC knownGenes annotation track so that epigenomic signal (e.g. ENCODE K562 H3K4me3 read-count profiles) can be pulled over each region for seeding or clustering in DGW.

## When to use

Use when you need to obtain concrete genomic coordinates from a gene annotation table (UCSC knownGenes track) to define fixed-length regions — e.g. 2 kb windows spanning TSS and first splicing site — on which ChIP-Seq enrichment profiles will be extracted.

## When NOT to use

- When intervals are already provided as a BED/peak file from a peak caller such as MACS2 — no gene-annotation extraction is needed
- When the analysis requires whole-genome tiling rather than gene-anchored 2 kb windows
- When the regions of interest lack a splicing site within the window and must instead be defined purely by enrichment peaks

## Inputs

- UCSC knownGenes annotation table (gene coordinates, TSS and exon structure)
- Reference genome assembly selection in the UCSC genome browser
- Optionally, a gene list (e.g. five seed genes) to restrict the extracted intervals

## Outputs

- Genomic interval coordinates (first 2 kb of each selected gene, containing TSS and first splicing site)
- Saved annotation file from the UCSC table browser output
- Interval set ready for extracting H3K4me3 read-count profiles from ENCODE BAM data

## How to apply

Query the UCSC genome browser table browser with the table set to `knownGenes` and download the gene annotation to file. Select candidate genes whose first 2 kb of the gene body contain both the transcription start site and the first splicing site, so the extracted intervals span the bimodal and single-peak H3K4me3 pattern types. Use these intervals as seed regions from which ENCODE K562 H3K4me3 read-count profiles are pulled, then bin the coverage for downstream DGW alignment and clustering. Rationale: anchors intervals to known genomic landmarks (TSS, FSS) so extracted profiles carry recognisable shape features that DGW can align without additional annotation input.

## Related tools

- **UCSC genome browser** (Table browser source of the knownGenes annotation from which genomic intervals (first 2 kb of gene bodies) are extracted)
- **DGW** (Downstream consumer of the extracted intervals: H3K4me3 profiles over these seed regions are aligned and clustered) — https://github.com/lukauskas/dgw

## Evaluation signals

- Each extracted interval spans both the transcription start site and the first splicing site of its source gene (so it covers both bimodal and single-peak pattern types)
- Intervals have the intended fixed length (initial 2 kb of the gene body)
- The knownGenes table output was saved to file and coordinates resolve against the same assembly as the ENCODE K562 data
- Downstream, the extracted seed regions yield profiles whose shapes align at TSS/FSS landmarks under DGW warping

## Limitations

- Only genes whose first splicing site falls within the extracted 2 kb window are suitable; intervals lacking points of interest would be dropped by DGW's --ignore-no-poi-regions flag
- Regions with no bin exceeding 10 reads would be ignored under the --min-pileup constraint downstream
- Interval extraction depends on the UCSC knownGenes table and cannot capture intergenic enrichment peaks not tied to annotated genes

## Evidence

- [methods] Extract the initial 2 kb of five genes from the UCSC knownGenes track and pull the H3K4me3 read-count profiles for these regions from the ENCODE K562 dataset: [evidence span withheld — anchor status: not_found]
- [methods] knownGenes table output: "the table set to `knownGenes`. Click `Get output` to get the data and save it to file."
- [intro] landmarks demonstration: "Subsequently, we demonstrate that DGW can align genomic landmarks such as TSSs and first splicing sites (FSSs) on real epigenomic data from the ENCODE project"
- [readme] DGW repository documentation pointer: "This is the repository for the open source DGW library developped by Saulius Lukauskas."
