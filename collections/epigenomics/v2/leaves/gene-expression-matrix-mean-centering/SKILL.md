---
name: gene-expression-matrix-mean-centering
description: Use when compiling multiple NCBI GEO series into one expression matrix
  and you need to remove per-experiment offsets so values from different GSE studies
  are comparable before joining or downstream correlation analysis.
when_to_use_negative:
- COnTORT can only download and combine normalized and processed data imported into
  NCBI GEO — do not use for raw (unprocessed) sequencing data
- Genes absent from the provided GenBank file are dropped/annotated N/A — do not use
  if your organism lacks a GenBank annotation file or you need to retain unannotated
  features
- Do not use if the analysis requires non-centered expression magnitudes without the
  companion non-mean-centered output
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3524
tools:
- name: COnTORT
  role: Pipeline that performs the mean-centering of each GEO experiment's expression
    values and joins them into a single output text file
  repo: https://github.com/GLBRC/contort
- name: Python 3
  role: Runtime for COnTORT and its mean-centering/joining logic
- name: pandas
  role: Python module used by COnTORT for tabular manipulation of expression matrices
- name: Bio
  role: Python module used to parse the GenBank file into annotation dictionaries
    used as the join key
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1128/mra.00587-20
    title: 'COnTORT: COmprehensive Transcriptomic ORganizational Tool for Simultaneously
      Retrieving and Organizing Numerous Gene Expression Data Sets from the NCBI Gene
      Expression Omnibus Database'
  source_content_hash: sha256:88521262654441ce6350f35f7ec3723bdef2e4d3a99cf9f25d4426b6b9734163
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GLBRC/contort
  tools:
  - Bio
  - COnTORT
  - Python 3
  - pandas
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# gene-expression-matrix-mean-centering

## Summary

Within the COnTORT pipeline, normalized log2 expression values from each GEO series (GSM) are mean-centered per experiment to control for between-experiment differences before being joined, with the GenBank gene annotation (locus tag, gene name, synonyms, product) as the key, into a single tab-delimited output file.

## When to use

Use when compiling multiple NCBI GEO series into one expression matrix and you need to remove per-experiment offsets so values from different GSE studies are comparable before joining or downstream correlation analysis.

## When NOT to use

- COnTORT can only download and combine normalized and processed data imported into NCBI GEO — do not use for raw (unprocessed) sequencing data
- Genes absent from the provided GenBank file are dropped/annotated N/A — do not use if your organism lacks a GenBank annotation file or you need to retain unannotated features
- Do not use if the analysis requires non-centered expression magnitudes without the companion non-mean-centered output

## Inputs

- Organized per-GSE expression tables from GEOannotate (metadata plus normalized log2 expression per gene, in GEOannotate_results)
- GeneOrf_match_output tables retaining only genes matched to the user-provided GenBank (.gbff) annotation file
- GenBank (.gbff) annotation file from NCBI (for the annotation key columns)

## Outputs

- COnTORT_organized_transcriptomic_data.txt: tab-delimited file with five annotation columns (Locus_Tag, Old_Locus_Tag, Gene_Name, Gene_Synonyms, Product) followed by mean-centered expression columns headed by GSM IDs
- A parallel non-mean-centered output file
- Per-experiment mean-centered files in the mean_centered_results directory

## How to apply

After gffMatch retains only expression rows with matches to the user-provided GenBank (.gbff) annotation dictionaries (locus tag, old locus tag, gene name, synonyms, product), COnTORT mean-centers each experiment's log2 normalized expression values — subtracting the experiment's mean so between-experiment differences are controlled — and writes per-experiment tables to the mean_centered_results directory. The mean-centered columns are then joined using the gene annotation as the key, retaining blank entries for consistency, with GSM IDs as column headers, producing COnTORT_organized_transcriptomic_data.txt whose first five columns carry Locus_Tag, Old_Locus_Tag, Gene_Name, Gene_Synonyms, and Product. A second non-mean-centered file is produced in parallel so users can choose the scale-sensitive representation.

## Related tools

- **COnTORT** (Pipeline that performs the mean-centering of each GEO experiment's expression values and joins them into a single output text file) — https://github.com/GLBRC/contort
- **Python 3** (Runtime for COnTORT and its mean-centering/joining logic)
- **pandas** (Python module used by COnTORT for tabular manipulation of expression matrices)
- **Bio** (Python module used to parse the GenBank file into annotation dictionaries used as the join key)

## Examples

```
pip install contort; contort
```

## Evaluation signals

- Output file exists and is tab-delimited, with the first row containing GSM IDs as column headers
- First five columns carry Locus_Tag, Old_Locus_Tag, Gene_Name, Gene_Synonyms, and Product headers
- Locus_Tag fields are populated for annotated genes, with N/A only where absent from the GenBank file
- Row count equals the number of annotated genes matched in the GenBank file
- Each mean-centered expression column has a mean of approximately zero; a parallel non-mean-centered file with identical row structure also exists

## Limitations

- COnTORT can only download and combine normalized and processed data imported into NCBI GEO, so raw data cannot be mean-centered via this pipeline
- Annotations not present in the provided GenBank file are listed with "N/A"; unmatched genes are not retained
- Acquisition and organization of many human or yeast series takes around an hour on a standard desktop computer (~20 min for bacterial genomes)

## Evidence

- [abstract] One output file will mean-center the data based on experiments to control for differences between experiments, while the second output file will not mean-center the data.: "One output file will mean-center the data based on experiments to control for differences between experiments, while the second output file will not mean-center the data."
- [readme] The data are then mean centered and joined together with the gene annotations annotation as the key.: "The data are then mean centered and joined together with the gene annotations annotation as the key. All blanks are retained for consistency."
- [readme] Primary output is a single text file containing the gene annotations on the first five columns and the mean centered expression data present in the remaining columns.: "Primary output is a single text file containing the gene annotations on the first five columns and the mean centered expression data present in the remaining columns."
- [intro] Mean-center data based on experiments to control for differences between experiments: "The data will be organized, mean centered, and then joined using the gene annotation into a single text file"
