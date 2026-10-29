---
name: sequence-book-output-validation
description: Use when a Delila-PY run has finished and the book of sequences emitted
  by the delila program must be checked against the input sites file before downstream
  motif cycles (malign/malin, alist/encode, makelogo).
when_to_use_negative:
- Input is not a delila-generated book of sequences (e.g. it is a FASTA, alignment,
  or sequence-logo output from later Delila-PY cycles)
- The sites file has column names, fewer/more than four columns, or strand values
  other than the words 'forward'/'reverse' — the pipeline contract is violated, so
  validation comparisons are meaningless
edam_operation: http://edamontology.org/operation_2478
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: delila
  role: Delila-PY program that consumes the library and genomic position files to
    generate the book of sequences being validated
  repo: https://github.com/GLBRC/delila
- name: Delila-PY
  role: Python3/Docker pipeline wrapping dbbk, catal and delila; invoked as delila_pipeline.py
    with -g (GenBank), -s (sites file), -l/-r boundaries
  repo: https://github.com/GLBRC/delila
- name: dbbk
  role: Processes the GenBank file into Delila database libraries that delila needs
    before it can produce the book
  repo: https://github.com/GLBRC/delila
- name: catal
  role: Generates the required catalyzed libraries from the dbbk output, a prerequisite
    for the delila book run
  repo: https://github.com/GLBRC/delila
- name: Docker
  role: Recommended execution environment (glbrc/delila image) in which the delila
    book generation and validation are run
  repo: https://hub.docker.com/r/glbrc/delila
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1128/mra.00160-21
    title: Delila-PY, a Pipeline for Utilizing the Delila Suite of Software to Identify
      Potential DNA Binding Motifs
  source_content_hash: sha256:940f896c2ecf1e2dbc794af2401f9641c212e521954af385ce86f14f30192570
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GLBRC/delila
  tools:
  - Delila-PY
  - Docker
  - Python3
  - catal
  - dbbk
  - delila
  edam_operation: http://edamontology.org/operation_2478
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Sequence book output validation

## Summary

Validates the Delila 'book of sequences' artifact produced by the delila program in the Delila-PY pipeline, confirming that exactly one extracted sequence entry exists per row of the input tab-delimited genomic position (sites) file and that the expected chromosome (e.g. NC_007493.2 of R. sphaeroides) is present in the records.

## When to use

Use when a Delila-PY run has finished and the book of sequences emitted by the delila program must be checked against the input sites file before downstream motif cycles (malign/malin, alist/encode, makelogo).

## When NOT to use

- Input is not a delila-generated book of sequences (e.g. it is a FASTA, alignment, or sequence-logo output from later Delila-PY cycles)
- The sites file has column names, fewer/more than four columns, or strand values other than the words 'forward'/'reverse' — the pipeline contract is violated, so validation comparisons are meaningless

## Inputs

- Book of sequences emitted by the delila program (Delila-PY)
- Tab-delimited genomic position (sites) file with four columns and no column names: chromosome name, name, strand, base pair position
- GenBank file for the target organism (e.g. R. sphaeroides chromosome NC_007493.2)
- User-specified left and right boundary values relative to each position

## Outputs

- Validation verdict that the book contains exactly one sequence entry per sites-file row
- Confirmation that the chromosome name (e.g. NC_007493.2) is present in the extracted sequence records
- A validated book of sequences suitable for the malign/malin information-content cycle

## How to apply

After running dbbk and catal to build the Delila libraries from the GenBank file, delila consumes the library files together with the genomic position file to emit the book of sequences. Validate by parsing the book and counting sequence entries: the count must equal the number of data rows in the four-column tab-delimited sites file (chromosome name, name, strand, base pair position, no column names). Verify that each book record's chromosome identifier matches the corresponding sites-file chromosome name, e.g. NC_007493.2 for R. sphaeroides, since the README requires site chromosome names to match the GenBank chromosome names. Confirm the extracted windows reflect the user-specified left and right boundaries (e.g. -l/-r values in delila_pipeline.py), so that a site at position 409225 with -10/-10 bounds yields position 409215 to 409235 on the declared strand. Only a book passing both the one-entry-per-row and chromosome-name checks should be fed into the first information-content cycle (malign and malin).

## Related tools

- **delila** (Delila-PY program that consumes the library and genomic position files to generate the book of sequences being validated) — https://github.com/GLBRC/delila
- **Delila-PY** (Python3/Docker pipeline wrapping dbbk, catal and delila; invoked as delila_pipeline.py with -g (GenBank), -s (sites file), -l/-r boundaries) — https://github.com/GLBRC/delila
- **dbbk** (Processes the GenBank file into Delila database libraries that delila needs before it can produce the book) — https://github.com/GLBRC/delila
- **catal** (Generates the required catalyzed libraries from the dbbk output, a prerequisite for the delila book run) — https://github.com/GLBRC/delila
- **Docker** (Recommended execution environment (glbrc/delila image) in which the delila book generation and validation are run) — https://hub.docker.com/r/glbrc/delila

## Examples

```
delila_pipeline.py -g Rsphaeroides_GenBank.gbff -l -10 -r -10 -s Location_File_Example.txt -t "Logo Title"
```

## Evaluation signals

- Number of sequence entries in the book equals the number of data rows in the tab-delimited sites file
- The chromosome name from the sites file (e.g. NC_007493.2 for R. sphaeroides) appears in the extracted sequence records of the book
- Each book record's coordinates span exactly the requested window: base pair position shifted by the -l (upstream) and -r (downstream) boundaries on the declared strand
- Book record identifiers correspond one-to-one to the unique 'name' column values in the sites file
- The dbbk and catal library prerequisites completed without error, since delila cannot emit the book from missing or empty libraries

## Limitations

- Site chromosome names must exactly match the chromosome names in the GenBank file, or delila will not extract the expected records
- The pipeline has been tested on MacOS and Linux (Ubuntu and CentOS); other operating systems may require additional work
- Left/right boundaries are always interpreted on the strand given in the sites file (forward or reverse), which can make window positions counterintuitive for upstream elements (e.g. -10 promoter uses Left = -20, Right = 0)

## Evidence

- [abstract] The library and genomic position files are used by delila ... to generate a book of sequences for subsequent use: "The library and genomic position files are used by delila (note that delila is a specific program within the Delila-PY software suite) to generate a book of sequences"
- [readme] Sites file format: "A sites file which is tab delimited with four columns (no column names):         
chromosome name : must match the chromosome names in genbank file"
- [readme] Example command: [evidence span withheld — anchor status: external_doc]
- [readme] Boundary semantics: [evidence span withheld — anchor status: external_doc]
