---
name: gene-annotation-expression-matching
description: 'Use when normalizing expression data from multiple GEO series for one
  organism: expression rows must be filtered to genes present in a GenBank annotation
  file and joined across experiments by locus tag/gene name.'
when_to_use_negative:
- The organism lacks a GenBank .gbff annotation file, or the expression data are raw
  (COnTORT can only download and combine normalized and processed data imported into
  NCBI GEO)
- The goal is to keep all GEO genes regardless of GenBank annotation — gffMatch deliberately
  discards rows without annotation matches
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0204
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_3174
tools:
- name: COnTORT
  role: Pipeline providing the gffMatch routine that performs the GenBank-to-expression
    matching and joined output
  repo: https://github.com/GLBRC/contort
- name: Bio
  role: Used to build dictionaries from the GenBank .gbff file for annotation matching
- name: GEOparse
  role: Produces the per-series normalized expression data and gene metadata that
    gffMatch filters and joins
- name: pandas
  role: Dataframe manipulation for matching, mean-centering, and joining expression
    tables
- name: argparse
  role: Python module used by COnTORT scripts for command-line parsing
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/mra.00587-20
    title: 'COnTORT: COmprehensive Transcriptomic ORganizational Tool for Simultaneously
      Retrieving and Organizing Numerous Gene Expression Data Sets from the NCBI Gene
      Expression Omnibus Database'
  source_content_hash: sha256:00ca356c9e6d73f26ca5fd8b9a5b51eb4568e10222c69a6ec3468a0fd6de6eb5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GLBRC/contort
  tools:
  - Bio
  - COnTORT
  - GEOparse
  - Python 3
  - argparse
  - ftplib
  - pandas
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0204
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# gene-annotation-expression-matching

## Summary

Matches gene expression rows from GEO series data to gene annotations in a user-provided GenBank (.gbff) file, retaining only rows whose genes match GenBank annotations, then mean-centers and joins all series by annotation key. This is the core organization step of COnTORT that turns heterogeneous GEO series into a single usable matrix.

## When to use

Use when normalizing expression data from multiple GEO series for one organism: expression rows must be filtered to genes present in a GenBank annotation file and joined across experiments by locus tag/gene name.

## When NOT to use

- The organism lacks a GenBank .gbff annotation file, or the expression data are raw (COnTORT can only download and combine normalized and processed data imported into NCBI GEO)
- The goal is to keep all GEO genes regardless of GenBank annotation — gffMatch deliberately discards rows without annotation matches

## Inputs

- GenBank gene annotation file for the organism (.gbff) downloaded from NCBI
- Organized GEO series files from the GEOannotate step (GEOparse results: per-gene metadata concatenated with normalized expression data per series)

## Outputs

- Per-series files with five annotation columns (Locus_Tag, Old_Locus_Tag, Gene_Name, Gene_Synonyms, Product) followed by log2 normalized expression data
- Mean-centered expression data per experiment (mean_centered_results directory)
- COnTORT_organized_transcriptomic_data.txt — single tab-delimited joined matrix, annotations in first five columns, GEO IDs as column headers in first row
- Parallel non-mean-centered transcriptomic matrix file

## How to apply

Run gffMatch, passing the user's GenBank (.gbff) file; dictionaries built with Bio from the .gbff are used to search for matches to the gene annotation and retain only data with matches. For each GEO series (organized previously by GEOannotate/GEOparse), write per-series files with five annotation columns — locus tag, old locus tag, gene name, gene synonyms, product, each written as 'N/A' when absent — followed by the log2 normalized expression data. Then mean-center the matched expression values per experiment to control for inter-experiment differences and join all series into a single tab-delimited file using the gene annotation as the key, retaining all blanks for consistency. Two outputs are produced: a mean-centered file and a parallel non-mean-centered file, both with GEO accession headers in the first row.

## Related tools

- **COnTORT** (Pipeline providing the gffMatch routine that performs the GenBank-to-expression matching and joined output) — https://github.com/GLBRC/contort
- **Bio** (Used to build dictionaries from the GenBank .gbff file for annotation matching)
- **GEOparse** (Produces the per-series normalized expression data and gene metadata that gffMatch filters and joins)
- **pandas** (Dataframe manipulation for matching, mean-centering, and joining expression tables)
- **argparse** (Python module used by COnTORT scripts for command-line parsing)

## Examples

```
python contort.py  # provide .gbff and GEO search term; gffMatch step matches expression rows to GenBank annotations and joins them
```

## Evaluation signals

- Both output files (mean-centered and non-mean-centered) exist, are tab-delimited, and carry the five annotation header fields with GEO accession headers in the first row
- Every expression row's locus tag/gene matches the provided GenBank .gbff; unmatched annotations appear as 'N/A' in the annotation columns
- Mean-centered columns have per-experiment means of approximately zero; the non-mean-centered file retains original log2 normalized values
- All blanks are retained for consistency between the two output files (row sets should be identical)
- Subdirectories geo, GEOannotate_results, GeneOrf_match_output, mean_centered_results, FTP_files, and log_files are populated by cleanUp

## Limitations

- COnTORT can only download and combine normalized and processed data imported into NCBI GEO — raw data cannot be matched or organized
- Annotations not present in the GenBank file are listed with 'N/A', and non-matching expression rows are dropped by gffMatch
- Matching quality depends entirely on the user-supplied GenBank annotation file; only one organism's annotations are applied per run

## Evidence

- [methods] gffMatch retains only matching rows: "Using dictionaries created from the GenBank file for the orgainsm, this script will search for matches to the gene annotation in the GenBank file"
- [readme] Annotation columns and N/A convention: "Note that annotations no present in the GenBank file are listed with "N/A"."
- [abstract] Two outputs, mean-centered and not: "One output file will mean-center the data based on experiments to control for differences between experiments, while the second output file will not mean-center the data."
- [readme] Log2 data and blanks retained: "The data are then mean centered and joined together with the gene annotations annotation as the key. All blanks are retained for consistency."
- [readme] Normalized data only: "Note that COnTORT can only download and combine normalized and processed data imported into NCBI GEO."
