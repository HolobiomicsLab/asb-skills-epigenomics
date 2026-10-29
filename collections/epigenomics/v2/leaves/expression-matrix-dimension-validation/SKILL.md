---
name: expression-matrix-dimension-validation
description: Use when a mean-centered GEO expression matrix has been produced (e.g.,
  by COnTORT's gffMatch step) and you must confirm the transformation did not alter
  rows/columns or annotation columns before downstream analysis.
when_to_use_negative:
- The input is raw, unnormalized GEO data — COnTORT can only download and combine
  normalized and processed data imported into NCBI GEO, so mean-centering raw values
  is out of scope
- You need per-gene (row-wise) centering or scaling rather than per-experiment (column-wise)
  mean-centering — this skill specifically centers each GSM experiment column
- The matrix has not yet been annotated via the GenBank (.gbff) file and joined with
  gffMatch, so the five-column annotation layout does not yet exist
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: COnTORT
  role: Produces the organized, mean-centered and non-mean-centered output files that
    this validation step checks
  repo: https://github.com/GLBRC/contort
- name: pandas
  role: Python module used to load the tab-delimited matrix, compute per-column means,
    and write the mean-centered output
- name: Python 3
  role: Runtime environment for the validation script (COnTORT requires Python 3)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1128/mra.00587-20
    title: 'COnTORT: COmprehensive Transcriptomic ORganizational Tool for Simultaneously
      Retrieving and Organizing Numerous Gene Expression Data Sets from the NCBI Gene
      Expression Omnibus Database'
  source_content_hash: sha256:c042748fb7d1fb162466459dac3b1292c58070cf337538a5988cb5e68cdd7362
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GLBRC/contort
  tools:
  - COnTORT
  - Python 3
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# Expression matrix dimension validation

## Summary

Verification step for COnTORT's mean-centered transcriptomic output: after subtracting per-experiment (per-GSM column) means from the organized expression matrix, confirm the resulting tab-delimited file preserves the input dimensions and that each expression column's mean is approximately zero.

## When to use

Use when a mean-centered GEO expression matrix has been produced (e.g., by COnTORT's gffMatch step) and you must confirm the transformation did not alter rows/columns or annotation columns before downstream analysis.

## When NOT to use

- The input is raw, unnormalized GEO data — COnTORT can only download and combine normalized and processed data imported into NCBI GEO, so mean-centering raw values is out of scope
- You need per-gene (row-wise) centering or scaling rather than per-experiment (column-wise) mean-centering — this skill specifically centers each GSM experiment column
- The matrix has not yet been annotated via the GenBank (.gbff) file and joined with gffMatch, so the five-column annotation layout does not yet exist

## Inputs

- Organized non-mean-centered expression matrix as a tab-delimited text file (COnTORT_organized_transcriptomic_data.txt) with 5 GenBank annotation columns plus GSM-keyed expression columns
- Expected dimensions (row count and column count) of the input matrix

## Outputs

- Mean-centered expression matrix as a tab-delimited text file with unchanged dimensions and preserved N/A annotation blanks
- Validation report showing each expression column's mean is within tolerance of zero and dimensions match the input

## How to apply

Load the organized non-mean-centered tab-delimited matrix (first five columns are Locus_Tag, Old_Locus_Tag, Gene_Name, Gene_Synonyms, and Product GenBank annotations; remaining columns are expression values keyed by GSM IDs) with pandas in Python 3. Compute the mean of each expression column and subtract it from every value in that column, using pandas column-wise operations so the five annotation columns and the row count are untouched. Write the result as a tab-delimited text file, retaining all blanks ('N/A' annotations) exactly as in the input for consistency. Validate by checking that (a) the output matrix has the same number of rows and columns as the input, (b) the five annotation columns are byte-identical to the input, and (c) each expression column's mean is within a small tolerance (e.g., 1e-10) of zero. Rationale: COnTORT mean-centers 'based on experiments to control for differences between experiments', so a correct output has zero-mean columns but identical structure to the uncentered companion file.

## Related tools

- **COnTORT** (Produces the organized, mean-centered and non-mean-centered output files that this validation step checks) — https://github.com/GLBRC/contort
- **pandas** (Python module used to load the tab-delimited matrix, compute per-column means, and write the mean-centered output)
- **Python 3** (Runtime environment for the validation script (COnTORT requires Python 3))

## Examples

```
import pandas as pd; df = pd.read_csv('COnTORT_organized_transcriptomic_data.txt', sep='\t'); expr = df.columns[5:]; df[expr] = df[expr] - df[expr].mean(); df.to_csv('mean_centered.txt', sep='\t', index=False)
```

## Evaluation signals

- Output file dimensions (rows × columns) exactly equal the input matrix's dimensions, with the same five annotation header columns
- Every expression column's mean is within a small tolerance (e.g., |mean| < 1e-10) of zero after centering
- Annotation columns (Locus_Tag through Product) are byte-identical between input and output, with all 'N/A' blanks retained
- The output opens cleanly as a tab-delimited file in Excel or R with the first row of GSM ID headers intact
- A non-mean-centered companion output produced from the same run differs from the centered file only by per-column constant offsets

## Limitations

- COnTORT can only download and combine normalized and processed data imported into NCBI GEO, so validation assumes the input values were already normalized
- Annotations absent from the user-provided GenBank file are listed as 'N/A'; these blanks must be preserved, and their presence is not an error
- Column means may not be exactly zero due to floating-point arithmetic; comparisons require a small tolerance rather than exact equality

## Evidence

- [abstract] One output file will mean-center the data based on experiments to control for differences between experiments, while the second output file will not mean-center the data.: "One output file will mean-center the data based on experiments to control for differences between experiments, while the second output file will not mean-center the data."
- [readme] README: two output files: "There will be two files:  one where the results are mean-centered for each experiment and one where no mean-centering is performed."
- [readme] README: N/A annotations: "Note that annotations no present in the GenBank file are listed with "N/A"."
- [readme] README: join and retain blanks: "The data are then mean centered and joined together with the gene annotations annotation as the key. All blanks are retained for consistency."
- [intro] intro: organized, mean centered, joined: "The data will be organized, mean centered, and then joined using the gene annotation into a single text file"
