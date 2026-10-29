---
name: tab-delimited-output-assembly
description: Use when a completed COnTORT run (GenBank .gbff plus a GEO search term)
  has produced per-series matched expression files and they must be joined into the
  final annotated, tab-delimited transcriptomic matrix.
when_to_use_negative:
- The data of interest are raw reads or unprocessed GEO submissions — COnTORT can
  only download and combine normalized and processed data imported into NCBI GEO
- The input expression files have not yet been filtered against the GenBank annotation
  via gffMatch
- Only a single series needs reformatting with no cross-experiment joining or mean-centering
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: COnTORT
  role: Primary pipeline that performs the joining, mean-centering, and tab-delimited
    output assembly
  repo: https://github.com/GLBRC/contort
- name: GEOparse
  role: Collects per-gene metadata from .soft.gz GSE files and concatenates it with
    normalized expression data prior to assembly
  repo: https://geoparse.readthedocs.io
- name: Bio
  role: Builds gene dictionaries from the GenBank .gbff file used by gffMatch to annotate
    output rows
- name: pandas
  role: Python module used for the table manipulation underlying the joined output
    files
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/mra.00587-20
    title: 'COnTORT: COmprehensive Transcriptomic ORganizational Tool for Simultaneously
      Retrieving and Organizing Numerous Gene Expression Data Sets from the NCBI Gene
      Expression Omnibus Database'
  source_content_hash: sha256:08301b3c14cb18e30e7aca35ca0da0cde6b4460fd8b7c3cdab88e1573069b108
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# tab-delimited-output-assembly

## Summary

Assembles COnTORT's final tab-delimited text files by joining mean-centered (and non-mean-centered) log2 normalized GEO expression data to GenBank gene annotations, using the gene annotation as the join key. Produces COnTORT_organized_transcriptomic_data.txt and a parallel non-centered file for downstream analysis in Excel or R.

## When to use

Use when a completed COnTORT run (GenBank .gbff plus a GEO search term) has produced per-series matched expression files and they must be joined into the final annotated, tab-delimited transcriptomic matrix.

## When NOT to use

- The data of interest are raw reads or unprocessed GEO submissions — COnTORT can only download and combine normalized and processed data imported into NCBI GEO
- The input expression files have not yet been filtered against the GenBank annotation via gffMatch
- Only a single series needs reformatting with no cross-experiment joining or mean-centering

## Inputs

- GenBank .gbff annotation file for the organism of choice, downloaded from NCBI
- GDS search-result txt file from the user's NCBI GEO search term
- GEO series .soft.gz files with normalized expression data processed by GEOannotate and gffMatch

## Outputs

- COnTORT_organized_transcriptomic_data.txt — mean-centered, tab-delimited file with five GenBank annotation columns plus expression columns headed by GEO IDs
- Parallel non-mean-centered tab-delimited transcriptomic matrix
- Organized subdirectories: geo, GEOannotate_results, GeneOrf_match_output, mean_centered_results, FTP_files, log_files

## How to apply

After gffMatch has retained only expression rows whose genes match dictionaries built with Bio from the user's GenBank .gbff file, mean-center the log2 normalized expression values per experiment to control for inter-experiment differences. Then join all series using the gene annotation as the key, writing two tab-delimited outputs: one mean-centered and one not mean-centered. Each output places five annotation columns first (locus tag, old locus tag, gene name, synonyms, product; 'N/A' where absent) followed by expression columns headed by GEO accession IDs (e.g., GSM IDs). Retain all blanks for consistency, and run cleanUp to move intermediates into the geo, GEOannotate_results, GeneOrf_match_output, mean_centered_results, FTP_files, and log_files subdirectories. Verify both files exist, are tab-delimited, and carry the five annotation headers with GEO IDs in the first row.

## Related tools

- **COnTORT** (Primary pipeline that performs the joining, mean-centering, and tab-delimited output assembly) — https://github.com/GLBRC/contort
- **GEOparse** (Collects per-gene metadata from .soft.gz GSE files and concatenates it with normalized expression data prior to assembly) — https://geoparse.readthedocs.io
- **Bio** (Builds gene dictionaries from the GenBank .gbff file used by gffMatch to annotate output rows)
- **pandas** (Python module used for the table manipulation underlying the joined output files)

## Examples

```
pip install contort; contort
```

## Evaluation signals

- Both output files exist: the mean-centered file and the non-mean-centered file
- Files are tab-delimited with the five annotation header fields (Locus_Tag, Old_Locus_Tag, Gene_Name, Gene_Synonyms, Product) followed by GEO accession headers in the first row
- Absent annotations are filled with 'N/A' rather than left empty or dropped
- Mean-centered file values are centered per experiment, controlling for inter-experiment differences
- Intermediate files were relocated by cleanUp into geo, GEOannotate_results, GeneOrf_match_output, mean_centered_results, FTP_files, and log_files subdirectories

## Limitations

- COnTORT can only download and combine normalized and processed data imported into NCBI GEO
- Annotations not present in the provided GenBank file are listed as 'N/A' and cannot be enriched after the fact
- Runtime scales with genome size: about 20 minutes for numerous bacterial samples, roughly an hour for yeast or human genomes on a standard desktop computer

## Evidence

- [abstract] One output file will mean-center the data based on experiments to control for differences between experiments, while the second output file will not mean-center the data.: "One output file will mean-center the data based on experiments to control for differences between experiments, while the second output file will not mean-center the data."
- [abstract] The first five columns contain the annotation information from the GenBank file (locus tag, old locus tag, gene name, gene synonyms, and product): "The first five columns contain the annotation information from the GenBank file (locus tag, old locus tag, gene name, gene synonyms, and product)"
- [readme] The data are then mean centered and joined together with the gene annotations annotation as the key. All blanks are retained for consistency.: "The data are then mean centered and joined together with the gene annotations annotation as the key. All blanks are retained for consistency."
- [readme] COnTORT_organized_transcriptomic_data.txt is the primary output.: "COnTORT_organized_transcriptomic_data.txt is the primary output."
- [intro] joined using the gene annotation into a single text file that can be easily manipulated or opened in Excel.: "joined using the gene annotation into a single text file that can be easily manipulated or opened in Excel."
