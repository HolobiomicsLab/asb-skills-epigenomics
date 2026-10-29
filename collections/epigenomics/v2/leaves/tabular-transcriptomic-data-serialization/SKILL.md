---
name: tabular-transcriptomic-data-serialization
description: Use when joining per-experiment GEO expression tables with GenBank annotation
  into one Excel/R-readable tab-delimited matrix with annotation columns as the key.
when_to_use_negative:
- The target data are raw (unprocessed) reads — COnTORT can only download and combine
  normalized and processed data imported into NCBI GEO
- You need a non-tabular interchange format (e.g. HDF5/AnnData); this skill's output
  is deliberately a plain tab-delimited text file openable in Excel or R
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: COnTORT
  role: Pipeline that serializes the joined annotation-plus-expression matrix as its
    primary tab-delimited output
  repo: https://github.com/GLBRC/contort
- name: GEOparse
  role: Parses GSE soft.gz files to produce the per-series annotation and normalized
    data tables upstream of serialization
- name: Bio
  role: Parses the GenBank file into locus-tag dictionaries used to populate annotation
    columns
- name: pandas
  role: Python module used for joining and tabular output construction
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1128/mra.00587-20
    title: 'COnTORT: COmprehensive Transcriptomic ORganizational Tool for Simultaneously
      Retrieving and Organizing Numerous Gene Expression Data Sets from the NCBI Gene
      Expression Omnibus Database'
  source_content_hash: sha256:cfd3444a331b86bc2112acbe3a4c965b6146bc79c5d35d64e8f6ff49e52e5035
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
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# tabular-transcriptomic-data-serialization

## Summary

Serializes matched, mean-centered GEO expression data plus GenBank gene annotation into a single tab-delimited text file, with the first five columns holding locus tag, old locus tag, gene name, gene synonyms, and product and the remaining columns holding expression values keyed by GSM ID.

## When to use

Use when joining per-experiment GEO expression tables with GenBank annotation into one Excel/R-readable tab-delimited matrix with annotation columns as the key.

## When NOT to use

- The target data are raw (unprocessed) reads — COnTORT can only download and combine normalized and processed data imported into NCBI GEO
- You need a non-tabular interchange format (e.g. HDF5/AnnData); this skill's output is deliberately a plain tab-delimited text file openable in Excel or R

## Inputs

- Organized expression tables per GSE series from GEOannotate (GEOparse-parsed normalized log2 data)
- Per-experiment mean-centered expression files in mean_centered_results
- User-provided GenBank (.gbff) gene annotation file for the organism from NCBI

## Outputs

- COnTORT_organized_transcriptomic_data.txt — single tab-delimited file: five annotation columns (Locus_Tag, Old_Locus_Tag, Gene_Name, Gene_Synonyms, Product) plus GSM-ID-headed expression columns
- A mean-centered version and a non-mean-centered version of the joined file

## How to apply

After gffMatch retains only rows whose locus tags match dictionaries built from the user-provided GenBank (.gbff) file, mean-center each experiment's expression values to control for between-experiment differences, writing per-experiment files into the mean_centered_results directory. Then join the mean-centered expression columns using the gene annotation as the join key, retaining blank entries for consistency, and emit GSM IDs as column headers. Write two final files: one mean-centered per experiment and one without mean centering, both as single tab-delimited text files (e.g. COnTORT_organized_transcriptomic_data.txt). Annotate genes absent from the GenBank file with 'N/A' in the annotation fields rather than dropping them.

## Related tools

- **COnTORT** (Pipeline that serializes the joined annotation-plus-expression matrix as its primary tab-delimited output) — https://github.com/GLBRC/contort
- **GEOparse** (Parses GSE soft.gz files to produce the per-series annotation and normalized data tables upstream of serialization)
- **Bio** (Parses the GenBank file into locus-tag dictionaries used to populate annotation columns)
- **pandas** (Python module used for joining and tabular output construction)

## Examples

```
pip install contort; contort
```

## Evaluation signals

- Output file exists and is tab-delimited
- First five columns carry the headers Locus_Tag, Old_Locus_Tag, Gene_Name, Gene_Synonyms, and Product
- Locus-tag fields are populated for annotated genes (N/A only where absent from the GenBank file)
- Row count equals the number of annotated genes matched in the GenBank file
- First row of expression columns contains the specific GEO IDs (GSM IDs) for each experiment

## Limitations

- COnTORT can only download and combine normalized and processed data imported into NCBI GEO
- Annotations not present in the provided GenBank file are listed with 'N/A' and those rows cannot be annotated further

## Evidence

- [abstract] The first five columns contain the annotation information from the GenBank file (locus tag, old locus tag, gene name, gene synonyms, and product): "The first five columns contain the annotation information from the GenBank file (locus tag, old locus tag, gene name, gene synonyms, and product)"
- [abstract] One output mean-centered, one not: "One output file will mean-center the data based on experiments to control for differences between experiments, while the second output file will not mean-center the data."
- [readme] Join with annotations as key, blanks retained: "The data are then mean centered and joined together with the gene annotations annotation as the key. All blanks are retained for consistency."
- [readme] N/A for missing annotations, GSM headers: "Note that annotations no present in the GenBank file are listed with "N/A"."
- [intro] Single text file openable in Excel: "joined using the gene annotation into a single text file that can be easily manipulated or opened in Excel."
