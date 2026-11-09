---
name: sequence-logo-generation
description: Use when you have a tab-delimited file of genomic binding-site coordinates
  (chromosome, name, strand, base pair position) plus a GenBank genome and need a
  reproducible DNA motif logo/PWM.
when_to_use_negative:
- You need de novo motif discovery from raw sequencing reads without genomic coordinates
  — Delila-PY requires a sites file of genomic positions and a GenBank file
- Your genome of interest has no GenBank record available, since dbbk and catal require
  it to build the Delila libraries
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
tools:
- name: Delila-PY
  role: Python3 pipeline orchestrating the Delila programs to build a binding-site
    logo from genomic locations
  repo: https://github.com/GLBRC/delila
- name: dbbk
  role: Generates the required Delila libraries from the GenBank file
- name: catal
  role: Generates the required Delila libraries from the GenBank file
- name: delila
  role: Uses library and genomic position files to generate a book of sequences; also
    regenerates updated books between cycles
- name: malign
  role: 'Cycle 1: searches for the alignment with the highest information content
    among all input sequences'
- name: ri
  role: 'Cycle 2: removes poorly matching sequences with ≤0 information content'
- name: makelogo
  role: 'Cycle 3: generates the final sequence logo from dalvec output'
- name: Docker
  role: 'Recommended runtime: glbrc/delila image lets users run the pipeline without
    compiling individual programs'
- name: p2c
  role: Pascal-to-C translator used to build the Delila programs from their Pascal
    sources
  repo: https://github.com/FranklinChen/p2c
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1128/mra.00160-21
    title: Delila-PY, a Pipeline for Utilizing the Delila Suite of Software to Identify
      Potential DNA Binding Motifs
  source_content_hash: sha256:a4dfdcae5a88c9246703052fa16f4f55b53e1aeca529fe62a228d86174364f39
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GLBRC/delila
  tools:
  - Delila-PY
  - Docker
  - Python3
  - alist
  - catal
  - cmp
  - dalvec
  - dbbk
  - delila
  - encode
  - makelogo
  - malign
  - malin
  - p2c
  - ri
  - rseq
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 20
---

# sequence-logo-generation

## Summary

Generate a DNA binding-site sequence logo and PWM from a set of genomic coordinates using the Delila-PY pipeline, which wraps the classic Delila suite in a Python3/Docker workflow with a three-cycle information-maximization loop.

## When to use

Use when you have a tab-delimited file of genomic binding-site coordinates (chromosome, name, strand, base pair position) plus a GenBank genome and need a reproducible DNA motif logo/PWM.

## When NOT to use

- You need de novo motif discovery from raw sequencing reads without genomic coordinates — Delila-PY requires a sites file of genomic positions and a GenBank file
- Your genome of interest has no GenBank record available, since dbbk and catal require it to build the Delila libraries

## Inputs

- GenBank file for the genome of interest (e.g. NC_007493.2 from NCBI GenBank)
- Tab-delimited sites file with four columns and no column names: chromosome name, site name, strand (forward/reverse), base pair position

## Outputs

- DNA sequence logo in PostScript and PDF formats
- Position weight matrix (PWM) values as a text file
- Individual DNA sequences used to generate the motif
- dalvec output from the final cycle

## How to apply

First set the scriptDir variable in delila_pipeline.py, then run the dbbk and catal programs on the GenBank file (e.g. NC_007493.2 for R. sphaeroides) to build the Delila libraries. Run delila with the libraries and the four-column sites file, specifying left/right boundaries relative to each base pair position (e.g. -l -10 -r -10 for a symmetric search window). Cycle 1 uses malign and malin to find the alignment with the highest information content among all input sequences, feeding results back to delila; cycle 2 computes per-sequence information content (alist, encode, cmp, rseq) and removes sequences with ≤0 bits via the ri program before regenerating the book; cycle 3 runs alist, encode, cmp, rseq, dalvec, and makelogo to produce the final motif. The recommended deployment is the glbrc/delila Docker image, which avoids compiling the individual Delila programs.

## Related tools

- **Delila-PY** (Python3 pipeline orchestrating the Delila programs to build a binding-site logo from genomic locations) — https://github.com/GLBRC/delila
- **dbbk** (Generates the required Delila libraries from the GenBank file)
- **catal** (Generates the required Delila libraries from the GenBank file)
- **delila** (Uses library and genomic position files to generate a book of sequences; also regenerates updated books between cycles)
- **malign** (Cycle 1: searches for the alignment with the highest information content among all input sequences)
- **ri** (Cycle 2: removes poorly matching sequences with ≤0 information content)
- **makelogo** (Cycle 3: generates the final sequence logo from dalvec output)
- **Docker** (Recommended runtime: glbrc/delila image lets users run the pipeline without compiling individual programs)
- **p2c** (Pascal-to-C translator used to build the Delila programs from their Pascal sources) — https://github.com/FranklinChen/p2c

## Examples

```
delila_pipeline.py -g Rsphaeroides_GenBank.gbff -l -10 -r -10 -s Location_File_Example.txt -t "Logo Title"
```

## Evaluation signals

- The sequence logo file exists in both PostScript and PDF formats
- The PWM text file is parseable and per-position information content falls within 0-2 bits
- The logo resembles the logo from independently identified binding sites (e.g. Delila-PY FnrL logo resembles the previously identified FnrL binding-site logo)
- No sequences with ≤0 bits information content remain after the ri filtering step in cycle 2

## Limitations

- Some additional work may be required to run Delila-PY on operating systems other than MacOS and Linux (Ubuntu and CentOS)
- Running the pipeline outside Docker requires downloading and compiling the individual Delila programs
- Sites-file chromosome names must exactly match the chromosome names in the GenBank file

## Evidence

- [abstract] Pipeline starts with dbbk/catal on GenBank: "The pipeline starts by running the dbbk and catal programs to generate the required libraries from the GenBank file"
- [abstract] Cycle 1 malign/malin highest information content: "The first cycle uses the malign and malin programs to search for the alignment with the highest information content among all input sequences."
- [abstract] Filter step removes ≤0 bit sequences with ri: "Delila-PY removes poorly matching sequences (≤0 information content, ri program). Then, a new file is generated and fed back to the delila program"
- [abstract] Outputs of the final cycle: "Delila-PY produces a DNA sequence logo in postscript and PDF formats, the position weight matrix (PWM) (11) values of the motif as a text file"
- [readme] Sites file format: "A sites file which is tab delimited with four columns (no column names)"
- [readme] Basic CLI invocation with boundaries: [evidence span withheld — anchor status: external_doc]
- [abstract] FnrL validation finding: "The resulting sequence logo from Delila-PY (Fig. 1B) resembles the logo generated from previously identified FnrL binding sites (Fig. 1C)"
