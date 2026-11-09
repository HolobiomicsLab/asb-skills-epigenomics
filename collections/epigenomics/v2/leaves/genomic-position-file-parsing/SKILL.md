---
name: genomic-position-file-parsing
description: 'Use when constructing Delila-PY inputs: you have ChIP-seq/ChIP-chip
  peak centers or transcription start sites and need to encode each location with
  chromosome, strand, and base position, then define -l/-r boundaries for motif extraction.'
when_to_use_negative:
- You already have aligned sequences or a multiple-sequence alignment — Delila-PY
  takes only genomic coordinates and a GenBank file, not pre-extracted sequences
- Your coordinates use numeric strand codes (e.g. +1/-1) instead of the literal words
  'forward' or 'reverse' — the sites file must be reformatted first
- You need motif discovery across a whole genome without positional priors — this
  skill only extracts windows around user-specified positions
tools:
- name: Delila-PY
  role: Pipeline whose delila_pipeline.py script parses the sites file and -l/-r boundary
    arguments before invoking the Delila programs
  repo: https://github.com/GLBRC/delila
- name: delila
  role: Specific Delila-PY program that consumes the library and genomic position
    files to generate the book of sequences
  repo: https://github.com/GLBRC/delila
- name: Docker
  role: Recommended execution environment for running Delila-PY with the sites file
    mounted as a shared volume
  repo: https://docs.docker.com/engine/install/
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1128/mra.00160-21
    title: Delila-PY, a Pipeline for Utilizing the Delila Suite of Software to Identify
      Potential DNA Binding Motifs
  source_content_hash: sha256:688d4072e4edf0ea331ac27aa0f50f2ba08565534772ae9b1bfc38c3e2a85195
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
  - makelogo
  - malign
  - ri
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# genomic-position-file-parsing

## Summary

Preparing and validating the Delila-PY tab-delimited genomic position (sites) file — four unnamed columns (chromosome, name, strand, base pair position) plus strand-relative left/right boundaries — so the delila program can extract the correct sequences and produce the book of sequences for binding-site motif discovery.

## When to use

Use when constructing Delila-PY inputs: you have ChIP-seq/ChIP-chip peak centers or transcription start sites and need to encode each location with chromosome, strand, and base position, then define -l/-r boundaries for motif extraction.

## When NOT to use

- You already have aligned sequences or a multiple-sequence alignment — Delila-PY takes only genomic coordinates and a GenBank file, not pre-extracted sequences
- Your coordinates use numeric strand codes (e.g. +1/-1) instead of the literal words 'forward' or 'reverse' — the sites file must be reformatted first
- You need motif discovery across a whole genome without positional priors — this skill only extracts windows around user-specified positions

## Inputs

- Tab-delimited genomic position (sites) file with four columns and no column names: chromosome name, unique site name, strand (forward/reverse), base pair position
- GenBank file for the target genome (e.g. NC_007493.2 for R. sphaeroides), whose chromosome names must match the sites file
- Delila libraries generated from the GenBank file by the dbbk and catal programs

## Outputs

- Book of sequences containing exactly one extracted sequence entry per row of the sites file, with strand-relative left/right boundaries applied

## How to apply

Build a tab-delimited sites file with exactly four columns and no column names: chromosome name, a unique site name, strand written as the words 'forward' or 'reverse', and the base pair position; the chromosome name must exactly match the name in the GenBank file (e.g. NC_007493.2 for R. sphaeroides). Pass the file to delila_pipeline.py via -s and define boundaries with -l (upstream distance) and -r (downstream distance); boundaries are always interpreted on the given strand, so searching for the -10 promoter element upstream of a transcription start site uses Left=-20/Right=0, while the -35 element uses Left=+5/Right=+25. The delila program consumes the sites file together with the dbbk/catal-generated libraries and emits a book of sequences. Validate the book by confirming one sequence entry per sites-file row and that the expected chromosome name (e.g. NC_007493.2) is present in the extracted records.

## Related tools

- **Delila-PY** (Pipeline whose delila_pipeline.py script parses the sites file and -l/-r boundary arguments before invoking the Delila programs) — https://github.com/GLBRC/delila
- **delila** (Specific Delila-PY program that consumes the library and genomic position files to generate the book of sequences) — https://github.com/GLBRC/delila
- **Docker** (Recommended execution environment for running Delila-PY with the sites file mounted as a shared volume) — https://docs.docker.com/engine/install/

## Examples

```
delila_pipeline.py -g Rsphaeroides_GenBank.gbff -l -10 -r -10 -s Location_File_Example.txt -t "Logo Title"
```

## Evaluation signals

- The book of sequences contains exactly one sequence entry per row of the position file
- The chromosome name (e.g. NC_007493.2) is present in the extracted sequence records
- Every strand column entry is the literal word 'forward' or 'reverse' — no '+', '-', or numeric codes
- Boundary windows match the strand-relative semantics: for position 100 with Left=-10/Right=+10 the extracted bounds are 90 and 110, and for the -35 promoter case Left=+5/Right=+25 yields 105 and 125

## Limitations

- Chromosome names in the sites file must exactly match the names in the GenBank file, otherwise extraction fails
- Boundaries are always relative to the given strand, so upstream promoter searches can require positive Left/Right values, which is counterintuitive
- Delila-PY has been tested only on MacOS and Linux (Ubuntu and CentOS); other operating systems may require additional work

## Evidence

- [intro] sites-file schema: "A sites file which is tab delimited with four columns (no column names)"
- [abstract] boundaries: "Users indicate the left and right boundaries relative to a genomic position within the genomic position file"
- [abstract] delila consumption: "The library and genomic position files are used by delila (note that delila is a specific program within the Delila-PY software suite) to generate a book of sequences"
- [readme] chromosome matching: "chromosome name : must match the chromosome names in genbank file"
- [readme] boundary semantics: "Left = -10 
 Right = +10 
 For site position 100,  the bounds to find a logo are position 90 (Left) and position 110 (Right)"
