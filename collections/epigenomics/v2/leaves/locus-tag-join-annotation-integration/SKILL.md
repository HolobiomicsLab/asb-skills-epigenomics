---
name: locus-tag-join-annotation-integration
description: Use when bulk-downloaded GEO series expression data must be keyed to
  an organism's gene annotations via locus tags before downstream joins, correlation,
  or cross-experiment comparison.
when_to_use_negative:
- The organism has no GenBank .gbff annotation file available — the match step retains
  only rows with GenBank annotation matches, so unannotated genes would be dropped.
- The GEO data of interest are raw or unnormalized — COnTORT can only download and
  combine normalized and processed data imported into NCBI GEO.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: COnTORT
  role: Pipeline implementing the GenBank-to-expression matching and annotation-keyed
    join
  repo: https://github.com/GLBRC/contort
- name: GEOparse
  role: Extracts gene metadata and normalized log2 expression data from each GSE soft.gz
    series file
  repo: https://geoparse.readthedocs.io
- name: Bio
  role: Parses the GenBank .gbff file to build locus-tag annotation dictionaries
- name: pandas
  role: Tabular joining of mean-centered expression columns using gene annotation
    as key
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1128/mra.00587-20
    title: 'COnTORT: COmprehensive Transcriptomic ORganizational Tool for Simultaneously
      Retrieving and Organizing Numerous Gene Expression Data Sets from the NCBI Gene
      Expression Omnibus Database'
  source_content_hash: sha256:1c9f4c46b9cf58f2b05da99eea3bdea290005c8291a771ccafc26679fae1beb4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GLBRC/contort
  tools:
  - Bio
  - COnTORT
  - GEOparse
  - Python 3
  - pandas
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# locus-tag-join-annotation-integration

## Summary

Integrates GEO-organized gene expression tables with a user-provided GenBank (.gbff) annotation file by matching locus tags, producing a single tab-delimited matrix whose first five columns carry GenBank annotation (locus tag, old locus tag, gene name, gene synonyms, product) and whose remaining columns hold per-GSM expression values.

## When to use

Use when bulk-downloaded GEO series expression data must be keyed to an organism's gene annotations via locus tags before downstream joins, correlation, or cross-experiment comparison.

## When NOT to use

- The organism has no GenBank .gbff annotation file available — the match step retains only rows with GenBank annotation matches, so unannotated genes would be dropped.
- The GEO data of interest are raw or unnormalized — COnTORT can only download and combine normalized and processed data imported into NCBI GEO.

## Inputs

- GenBank gene annotation file (.gbff) for the organism, downloaded from NCBI
- Organized GEO series expression tables from GEOannotate (GEOannotate_results directory), extracted from GSE soft.gz files via GEOparse

## Outputs

- Tab-delimited matrix (COnTORT_organized_transcriptomic_data.txt) with five annotation columns (Locus_Tag, Old_Locus_Tag, Gene_Name, Gene_Synonyms, Product) followed by mean-centered expression columns headed by GSM IDs
- Companion non-mean-centered output file
- Per-experiment mean-centered files in the mean_centered_results directory

## How to apply

Build dictionaries from the user-supplied GenBank .gbff file (parsed with the Bio module) mapping each locus tag to its old locus tag, gene name, gene synonyms, and product. Match the GEOannotate-organized expression tables (log2-normalized data extracted per GSE series with GEOparse) against these dictionaries, retaining only rows with annotation matches and writing per-series results to the GeneOrf_match_output directory. Mean-center each experiment's expression values to control for between-experiment differences, then join all mean-centered expression columns using the gene annotation as the key, retaining blank entries for consistency. Use GSM IDs as column headers; mark annotation fields absent from the GenBank file as 'N/A'.

## Related tools

- **COnTORT** (Pipeline implementing the GenBank-to-expression matching and annotation-keyed join) — https://github.com/GLBRC/contort
- **GEOparse** (Extracts gene metadata and normalized log2 expression data from each GSE soft.gz series file) — https://geoparse.readthedocs.io
- **Bio** (Parses the GenBank .gbff file to build locus-tag annotation dictionaries)
- **pandas** (Tabular joining of mean-centered expression columns using gene annotation as key)

## Examples

```
git clone https://github.com/GLBRC/contort.git; python contort.py
```

## Evaluation signals

- Output file exists and is tab-delimited
- First five columns carry Locus_Tag, Old_Locus_Tag, Gene_Name, Gene_Synonyms, and Product headers
- Locus-tag fields are populated for annotated genes, with 'N/A' only where absent from the GenBank file
- Row count equals the number of annotated genes matched in the GenBank file
- First row of the expression columns references specific GEO IDs (GSM identifiers) for each experiment

## Limitations

- COnTORT can only download and combine normalized and processed data imported into NCBI GEO; raw data are out of scope.
- Annotations not present in the GenBank file are listed as 'N/A', and only rows with annotation matches are retained.
- Mean-centering is per experiment to control for between-experiment differences; a non-mean-centered file is also produced for users needing uncentered values.

## Evidence

- [abstract] The first five columns contain the annotation information from the GenBank file (locus tag, old locus tag, gene name, gene synonyms, and product): "The first five columns contain the annotation information from the GenBank file (locus tag, old locus tag, gene name, gene synonyms, and product)"
- [readme] Using dictionaries created from the GenBank file for the orgainsm, this script will search for matches to the gene annotation in the GenBank file and retain only those data with matches.: "Using dictionaries created from the GenBank file for the orgainsm, this script will search for matches to the gene annotation in the GenBank file and retain only those data with matches."
- [readme] The data are then mean centered and joined together with the gene annotations annotation as the key. All blanks are retained for consistency.: "The data are then mean centered and joined together with the gene annotations annotation as the key. All blanks are retained for consistency."
- [readme] Note that annotations no present in the GenBank file are listed with "N/A".: "Note that annotations no present in the GenBank file are listed with "N/A"."
