---
name: tab-delimited-transcriptome-table-generation
description: Use when a task requires anchoring GEO gene expression data to gene annotations
  from a user-provided GenBank .gbff file and producing a joined, Excel/R-ready tab-delimited
  table.
when_to_use_negative:
- The expression data are raw reads or not normalized/processed data imported into
  NCBI GEO — COnTORT can only download and combine normalized and processed GEO data.
- The organism's gene annotations are not available as a GenBank .gbff file, since
  the user must provide it and gffMatch retains only matches to annotations in that
  file.
- A single GEO series' raw metadata alone is needed — GEOannotate/GEOparse organization,
  not annotation-keyed table generation, is the appropriate step.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3174
tools:
- name: COnTORT
  role: Pipeline whose gffMatch step matches GenBank annotations to organized GEO
    expression data and writes the joined tab-delimited output
  repo: https://github.com/GLBRC/contort
- name: Bio (Biopython)
  role: Parses the GenBank (.gbff) file to build gene-annotation dictionaries (locus
    tag, old locus tag, gene name, synonyms, product)
- name: GEOparse
  role: Parses and organizes GEO series metadata and normalized data in the GEOannotate
    step feeding gffMatch
  repo: https://geoparse.readthedocs.io
- name: pandas
  role: Python module used for tabular manipulation of expression data and writing
    tab-delimited output
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1128/mra.00587-20
    title: 'COnTORT: COmprehensive Transcriptomic ORganizational Tool for Simultaneously
      Retrieving and Organizing Numerous Gene Expression Data Sets from the NCBI Gene
      Expression Omnibus Database'
  source_content_hash: sha256:e173eb4d361fad7d2c82188c8c0c3167400ce62c7cd156654a2c0bb626ac22ab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GLBRC/contort
  tools:
  - Bio
  - Bio (Biopython)
  - COnTORT
  - GEOparse
  - Python 3
  - pandas
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# tab-delimited-transcriptome-table-generation

## Summary

Generate a single tab-delimited transcriptomic output file in which the first five columns carry GenBank-derived gene annotations (locus tag, old locus tag, gene name, gene synonyms, product) followed by GEO expression values, produced by COnTORT's gffMatch/GEOannotate pipeline.

## When to use

Use when a task requires anchoring GEO gene expression data to gene annotations from a user-provided GenBank .gbff file and producing a joined, Excel/R-ready tab-delimited table.

## When NOT to use

- The expression data are raw reads or not normalized/processed data imported into NCBI GEO — COnTORT can only download and combine normalized and processed GEO data.
- The organism's gene annotations are not available as a GenBank .gbff file, since the user must provide it and gffMatch retains only matches to annotations in that file.
- A single GEO series' raw metadata alone is needed — GEOannotate/GEOparse organization, not annotation-keyed table generation, is the appropriate step.

## Inputs

- GenBank gene annotation file (.gbff) for the organism, downloaded from NCBI
- GEOparse-organized GEO series expression results from the GEOannotate step (GEOannotate_results directory)
- GEO soft.gz series files and normalized/processed expression data imported into NCBI GEO

## Outputs

- COnTORT_organized_transcriptomic_data.txt: a single tab-delimited text file with Locus_Tag, Old_Locus_Tag, Gene_Name, Gene_Synonyms, Product columns followed by mean-centered (and a non-mean-centered version) expression columns headed by GSM IDs
- Per-GEO-series files with the five gene-annotation columns followed by log2 normalized expression columns
- GeneOrf_match_output and mean_centered_results directories

## How to apply

Load the organism's GenBank (.gbff) file with the Biopython (Bio) module and build dictionaries mapping each gene record to its locus tag, old locus tag, gene name, gene synonyms, and product. Read the GEOparse-organized expression results from the GEOannotate step, search them for matches to the GenBank-derived annotations, and retain only rows with matches. Write new files per GEO series whose columns are the five annotation fields followed by the log2 normalized expression values keyed by the gene annotation, then mean-center the expression data per experiment and join all series on the gene annotation as the key, retaining all blanks for consistency. Save the resulting table (COnTORT_organized_transcriptomic_data.txt in GeneOrf_match_output / mean_centered_results directories); annotations absent from the GenBank file are listed as 'N/A'.

## Related tools

- **COnTORT** (Pipeline whose gffMatch step matches GenBank annotations to organized GEO expression data and writes the joined tab-delimited output) — https://github.com/GLBRC/contort
- **Bio (Biopython)** (Parses the GenBank (.gbff) file to build gene-annotation dictionaries (locus tag, old locus tag, gene name, synonyms, product))
- **GEOparse** (Parses and organizes GEO series metadata and normalized data in the GEOannotate step feeding gffMatch) — https://geoparse.readthedocs.io
- **pandas** (Python module used for tabular manipulation of expression data and writing tab-delimited output)

## Examples

```
pip install contort; contort  # provide the organism .gbff and search term; gffMatch(GBFF) produces GeneOrf_match_output/COnTORT_organized_transcriptomic_data.txt
```

## Evaluation signals

- Output is a tab-delimited file whose five leading columns are exactly Locus_Tag, Old_Locus_Tag, Gene_Name, Gene_Synonyms, and Product, with expression columns headed by GEO GSM IDs.
- Row count equals the gene/CDS count in the GBFF, restricted to genes with matches retained by gffMatch.
- Annotations absent from the GenBank file are filled with 'N/A'.
- One output file is mean-centered per experiment and a second is not mean-centered, as COnTORT produces both.
- All blanks are retained when joining series on the gene annotation key, for consistency.

## Limitations

- COnTORT can only download and combine normalized and processed data imported into NCBI GEO; raw or unprocessed data cannot be handled.
- Only rows with matches to the user-provided GenBank annotations are retained; unmatched data are dropped.
- The user must supply the GenBank (.gbff) file for annotation — it is not fetched automatically for the organism.

## Evidence

- [abstract] The first five columns contain the annotation information from the GenBank file (locus tag, old locus tag, gene name, gene synonyms, and product): "The first five columns contain the annotation information from the GenBank file (locus tag, old locus tag, gene name, gene synonyms, and product)"
- [methods] gffMatch description: "Using dictionaries created from the GenBank file for the orgainsm, this script will search for matches to the gene annotation in the GenBank file and retain only those data with matches."
- [readme] README output description: "Note that annotations no present in the GenBank file are listed with "N/A"."
- [intro] intro scope: "The data will be organized, mean centered, and then joined using the gene annotation into a single text file that can be easily manipulated or opened in Excel."
- [intro] intro limitation: "COnTORT can only download and combine normalized and processed data imported into NCBI GEO."
