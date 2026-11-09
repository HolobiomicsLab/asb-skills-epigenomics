---
name: gene-annotation-column-preservation
description: Use when mean-centering or otherwise transforming a COnTORT-style expression
  matrix where the first five columns are GenBank annotations and must survive transformation
  unchanged, including 'N/A' blanks.
when_to_use_negative:
- Input data are not normalized and processed data imported into NCBI GEO — COnTORT
  can only download and combine normalized and processed data
- The matrix has no GenBank annotation columns (e.g., raw GEO series files prior to
  gffMatch) — annotation preservation applies to post-gffMatch organized data
- You only need the non-mean-centered output file; no transformation is applied in
  that case
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3174
tools:
- name: COnTORT
  role: Pipeline producing the organized, annotated expression matrix and the mean-centered
    joined output file
  repo: https://github.com/GLBRC/contort
- name: pandas
  role: Python module used to load the expression matrix, compute per-column (per-experiment)
    means, and write the tab-delimited mean-centered output while retaining annotation
    columns
- name: Python 3
  role: Runtime environment for the COnTORT mean-centering and annotation-join step
  repo: https://github.com/GLBRC/contort
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1128/mra.00587-20
    title: 'COnTORT: COmprehensive Transcriptomic ORganizational Tool for Simultaneously
      Retrieving and Organizing Numerous Gene Expression Data Sets from the NCBI Gene
      Expression Omnibus Database'
  source_content_hash: sha256:250b5807cf9240962fe98d1d414b29491826504d9c352506f63f71de0aa8cc36
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GLBRC/contort
  tools:
  - COnTORT
  - Python 3
  - pandas
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# gene-annotation-column-preservation

## Summary

Preserves the five GenBank-derived gene annotation columns (Locus_Tag, Old_Locus_Tag, Gene_Name, Gene_Synonyms, Product) and all 'N/A' blanks while mean-centering COnTORT expression data per experiment (GSM ID), so the mean-centered output remains joinable by gene annotation.

## When to use

Use when mean-centering or otherwise transforming a COnTORT-style expression matrix where the first five columns are GenBank annotations and must survive transformation unchanged, including 'N/A' blanks.

## When NOT to use

- Input data are not normalized and processed data imported into NCBI GEO — COnTORT can only download and combine normalized and processed data
- The matrix has no GenBank annotation columns (e.g., raw GEO series files prior to gffMatch) — annotation preservation applies to post-gffMatch organized data
- You only need the non-mean-centered output file; no transformation is applied in that case

## Inputs

- Organized non-mean-centered expression matrix (tab-delimited): five GenBank annotation columns (Locus_Tag, Old_Locus_Tag, Gene_Name, Gene_Synonyms, Product) plus expression columns keyed by GSM IDs
- GenBank (.gbff) gene annotation file for the organism of choice, downloaded from NCBI

## Outputs

- Mean-centered tab-delimited text file with identical dimensions to the input: five annotation columns (with 'N/A' for unannotated genes) followed by mean-centered expression columns per experiment
- Verification check: each expression column mean within a small tolerance of zero; row/column count matches input matrix

## How to apply

Load the organized non-mean-centered expression matrix — first five columns holding Locus_Tag, Old_Locus_Tag, Gene_Name, Gene_Synonyms, and Product from the user-provided GenBank (.gbff) file, remaining columns holding log2-normalized expression values keyed by GSM IDs — using pandas in Python 3. Compute the mean of each expression column (experiment) and subtract it from every value in that column, leaving the five annotation columns and the row/column count untouched. Write the mean-centered table as a tab-delimited text file (COnTORT_organized_transcriptomic_data.txt in mean_centered_results/), retaining all blanks ('N/A' annotations) for consistency so gene annotation remains the join key. The rationale, per the README's gffMatch description, is that 'The data are then mean centered and joined together with the gene annotations annotation as the key' — corrupting or dropping annotation columns or blanks would break the join and downstream use in Excel or R.

## Related tools

- **COnTORT** (Pipeline producing the organized, annotated expression matrix and the mean-centered joined output file) — https://github.com/GLBRC/contort
- **pandas** (Python module used to load the expression matrix, compute per-column (per-experiment) means, and write the tab-delimited mean-centered output while retaining annotation columns)
- **Python 3** (Runtime environment for the COnTORT mean-centering and annotation-join step) — https://github.com/GLBRC/contort

## Examples

```
python contort.py  # after providing the GenBank .gbff; COnTORT mean-centers per experiment and writes COnTORT_organized_transcriptomic_data.txt with annotation columns intact
```

## Evaluation signals

- Each expression column's mean in the output is within a small tolerance of zero
- Output row and column counts match the input organized matrix (no rows or annotation columns lost)
- The five annotation columns are byte-identical to the input matrix, including all 'N/A' entries
- Output header row references the specific GEO IDs (GSM_ID_1, GSM_ID_2, ...) for each experiment
- Output is tab-delimited and openable/manipulable in Excel or R as COnTORT specifies

## Limitations

- COnTORT can only download and combine normalized and processed data imported into NCBI GEO
- Genes whose annotations are not present in the provided GenBank file are listed with 'N/A', so blanks must be retained rather than dropped or imputed
- A separate non-mean-centered output file is also produced; mean-centering controls for differences between experiments but the uncentered file is the reference for raw normalized values

## Evidence

- [abstract] One output file will mean-center the data based on experiments to control for differences between experiments, while the second output file will not mean-center the data.: "One output file will mean-center the data based on experiments to control for differences between experiments, while the second output file will not mean-center the data."
- [abstract] The first five columns contain the annotation information from the GenBank file (locus tag, old locus tag, gene name, gene synonyms, and product): "The first five columns contain the annotation information from the GenBank file (locus tag, old locus tag, gene name, gene synonyms, and product)"
- [readme] The data are then mean centered and joined together with the gene annotations annotation as the key. All blanks are retained for consistency.: "The data are then mean centered and joined together with the gene annotations annotation as the key. All blanks are retained for consistency."
- [readme] Note that annotations no present in the GenBank file are listed with "N/A".: "Note that annotations no present in the GenBank file are listed with "N/A"."
- [readme] mean_centered_results   - the mean centered expression data for each experiment: "mean_centered_results   - the mean centered expression data for each experiment"
