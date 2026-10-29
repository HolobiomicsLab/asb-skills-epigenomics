---
name: delila-pipeline-cycle-orchestration
description: Use when you have a GenBank file and a tab-delimited file of genomic
  positions (e.g., ChIP-seq peaks or transcription start sites) and need to reconstruct
  a DNA binding motif as a sequence logo and position weight matrix.
when_to_use_negative:
- You only have raw reads or unaligned sequences without genomic coordinates — Delila-PY
  requires a GenBank file and a sites file of genomic positions
- You need de novo motif discovery from sequences alone with no reference genome context;
  tools like MEME or BioProspector operate on different inputs
- Your organism's genome is not available as a GenBank file compatible with dbbk/catal
  library generation
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0749
tools:
- name: Delila-PY
  role: Python3 pipeline orchestrating all Delila programs across the three cycles
  repo: https://github.com/GLBRC/delila
- name: delila
  role: Generates the book of sequences from libraries and genomic position files;
    regenerates updated books after cycles 1 and 2
  repo: https://github.com/GLBRC/delila
- name: malign
  role: 'Cycle 1: searches for the alignment with the highest information content
    among all input sequences'
  repo: https://github.com/GLBRC/delila
- name: ri
  role: 'Cycle 2: removes poorly matching sequences with ≤0 information content'
  repo: https://github.com/GLBRC/delila
- name: dalvec
  role: 'Cycle 3: contributes to final motif generation'
- name: makelogo
  role: 'Cycle 3: generates the sequence logo in PostScript and PDF'
- name: dbbk
  role: Generates required Delila libraries from the GenBank file
- name: catal
  role: Generates required Delila libraries from the GenBank file alongside dbbk
- name: Docker
  role: Recommended execution environment via the glbrc/delila image
  repo: https://hub.docker.com/r/glbrc/delila
provenance:
  source_task_ids:
  - task_004
  - task_003
  - task_001
  source_papers:
  - doi: 10.1128/mra.00160-21
    title: Delila-PY, a Pipeline for Utilizing the Delila Suite of Software to Identify
      Potential DNA Binding Motifs
  source_content_hash: sha256:8ab609ac4f9abf8b89dfed1b7f2461c08436a15cd126bfcdf9b5789ca5f3a40c
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
  - ri
  - rseq
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# delila-pipeline-cycle-orchestration

## Summary

Orchestrates the Delila-PY three-cycle information-maximization loop that turns a GenBank file plus a genomic sites file into a DNA sequence logo and PWM. Cycle 1 aligns sequences (malign/malin), cycle 2 removes poor matches (ri, ≤0 bits), and cycle 3 generates the motif (dalvec, makelogo).

## When to use

Use when you have a GenBank file and a tab-delimited file of genomic positions (e.g., ChIP-seq peaks or transcription start sites) and need to reconstruct a DNA binding motif as a sequence logo and position weight matrix.

## When NOT to use

- You only have raw reads or unaligned sequences without genomic coordinates — Delila-PY requires a GenBank file and a sites file of genomic positions
- You need de novo motif discovery from sequences alone with no reference genome context; tools like MEME or BioProspector operate on different inputs
- Your organism's genome is not available as a GenBank file compatible with dbbk/catal library generation

## Inputs

- GenBank file for the target organism (e.g., NC_007493.2 for R. sphaeroides)
- Tab-delimited sites file with four columns (no header): chromosome name, site name, strand (forward/reverse), base pair position
- Left and right boundary distances relative to each genomic position (e.g., -l -10 -r -10)

## Outputs

- DNA sequence logo in PostScript and PDF formats
- Position weight matrix (PWM) values as a text file
- Individual DNA sequences used to generate the motif
- dalvec output

## How to apply

First run dbbk and catal on the GenBank file to build the Delila libraries, then run delila with the libraries and the sites file (chromosome, name, strand, base pair position) plus left/right boundaries relative to each position to produce the initial book of sequences. Cycle 1 uses malign and malin to search for the alignment with the highest information content among all input sequences, feeding malin results back to delila to regenerate the book. Cycle 2 computes per-sequence information content with alist, encode, cmp, and rseq, then removes poorly matching sequences (information content ≤0 bits) with the ri program and regenerates the book with delila. Cycle 3 runs alist, encode, cmp, rseq, dalvec, and makelogo on the final book to produce the DNA motif. Configure the pipeline by setting the scriptDir variable in delila_pipeline.py, and prefer the Docker image (glbrc/delila) for reproducible execution.

## Related tools

- **Delila-PY** (Python3 pipeline orchestrating all Delila programs across the three cycles) — https://github.com/GLBRC/delila
- **delila** (Generates the book of sequences from libraries and genomic position files; regenerates updated books after cycles 1 and 2) — https://github.com/GLBRC/delila
- **malign** (Cycle 1: searches for the alignment with the highest information content among all input sequences) — https://github.com/GLBRC/delila
- **ri** (Cycle 2: removes poorly matching sequences with ≤0 information content) — https://github.com/GLBRC/delila
- **dalvec** (Cycle 3: contributes to final motif generation)
- **makelogo** (Cycle 3: generates the sequence logo in PostScript and PDF)
- **dbbk** (Generates required Delila libraries from the GenBank file)
- **catal** (Generates required Delila libraries from the GenBank file alongside dbbk)
- **Docker** (Recommended execution environment via the glbrc/delila image) — https://hub.docker.com/r/glbrc/delila

## Examples

```
delila_pipeline.py -g Rsphaeroides_GenBank.gbff -l -10 -r -10 -s Location_File_Example.txt -t "Logo Title"
```

## Evaluation signals

- The final sequence logo (PostScript/PDF) exists and resembles the logo generated from previously identified binding sites (e.g., Fig. 1C for FnrL), supporting predictive power
- PWM text file is parseable and per-position information content values fall within the 0–2 bits range
- Number of retained and removed sequences in cycle 2 exactly equals counts obtained by independently applying the ≤0-bit cutoff to the per-sequence information-content table; no retained sequence has information content ≤0 bits
- Cycle 3 artifacts (PWM text file, sequence logo ps/pdf, dalvec output) all exist after the run completes

## Limitations

- Delila-PY has been tested on MacOS and Linux (Ubuntu and CentOS); running on other operating systems may require additional work
- Sites file chromosome names must exactly match the chromosome names in the GenBank file
- The logo title (-t) must be enclosed in quotes or it defaults to the species name

## Evidence

- [abstract] Three cycles in Delila-PY: "The first cycle uses the malign and malin programs to search for the alignment with the highest information content among all input sequences."
- [abstract] Cycle 2 filter: "Delila-PY removes poorly matching sequences (≤0 information content, ri program). Then, a new file is generated and fed back to the delila program to generate an updated book file"
- [methods] Pipeline setup: "Prior to running delila you will need to define the script directory by editing the scriptDir varible in the delila_pipeline.py file."
- [readme] Input requirements: "A sites file which is tab delimited with four columns (no column names): chromosome name : must match the chromosome names in genbank file"
- [readme] Basic command: [evidence span withheld — anchor status: external_doc]
- [abstract] Output formats: "Delila-PY produces a DNA sequence logo in postscript and PDF formats, the position weight matrix (PWM) (11) values of the motif as a text file, and the individual DNA sequences used to generate the"
