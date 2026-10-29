---
name: position-weight-matrix-construction
description: Use when you have a GenBank file plus a tab-delimited sites file of genomic
  positions (e.g., ChIP-seq peak centers or transcription start sites) and need a
  binding-site logo and PWM.
when_to_use_negative:
- You need de novo motif discovery without genomic coordinates — the article contrasts
  Delila-PY with MEME (expectation maximization) and BioProspector (Gibbs sampling)
  tools
- Your input lacks a GenBank file or sites file with strand and base pair position;
  Delila-PY requires both before any processing
- The workflow requires Windows or other non-MacOS/non-Linux platforms — the README
  notes Delila-PY has been tested on MacOS and Linux (Ubuntu and CentOS) only
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3168
tools:
- name: Delila-PY
  role: Custom Python3 pipeline orchestrating the Delila programs through three information-maximization
    cycles to build the PWM and logo
  repo: https://github.com/GLBRC/delila
- name: delila
  role: Generates the book of sequences from library and genomic position files; regenerates
    updated books after each cycle
  repo: https://github.com/GLBRC/delila
- name: malign
  role: 'Cycle 1: searches for the alignment with the highest information content
    among all input sequences'
  repo: https://github.com/GLBRC/delila
- name: malin
  role: 'Cycle 1: its results are fed back to delila to generate an updated book file'
  repo: https://github.com/GLBRC/delila
- name: alist
  role: 'Cycles 2 and 3: aligns sequences for information content calculation'
  repo: https://github.com/GLBRC/delila
- name: encode
  role: 'Cycles 2 and 3: encodes aligned sequences for information analysis'
  repo: https://github.com/GLBRC/delila
- name: cmp
  role: 'Cycles 2 and 3: computes composition information for the aligned sequences'
  repo: https://github.com/GLBRC/delila
- name: rseq
  role: 'Cycles 2 and 3: computes individual sequence information content; values
    ≤0 trigger removal in cycle 2'
  repo: https://github.com/GLBRC/delila
- name: ri
  role: 'Cycle 2: removes poorly matching sequences with ≤0 information content'
  repo: https://github.com/GLBRC/delila
- name: dalvec
  role: 'Cycle 3: generates the DNA motif from individual sequence vectors'
  repo: https://github.com/GLBRC/delila
- name: makelogo
  role: 'Cycle 3: renders the final DNA sequence logo'
  repo: https://github.com/GLBRC/delila
- name: dbbk
  role: Generates required libraries from the GenBank file at pipeline start
  repo: https://github.com/GLBRC/delila
- name: catal
  role: Generates required catalogs from the GenBank file at pipeline start
  repo: https://github.com/GLBRC/delila
- name: Docker
  role: Recommended execution environment via the glbrc/delila image
  repo: https://hub.docker.com/r/glbrc/delila
- name: p2c
  role: Pascal-to-C translator used to compile the original Delila programs when building
    outside Docker
  repo: https://github.com/FranklinChen/p2c
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1128/mra.00160-21
    title: Delila-PY, a Pipeline for Utilizing the Delila Suite of Software to Identify
      Potential DNA Binding Motifs
  source_content_hash: sha256:0b258953779652b11625a0574ed30ba841033e1913168e8bf60483799539cd39
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
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8636363636363636
n_claims: 22
---

# position-weight-matrix-construction

## Summary

Construct a DNA binding-site position weight matrix (PWM) and sequence logo from genomic coordinates using the Delila-PY pipeline, which runs the Delila suite (malign, malin, alist, encode, cmp, rseq, ri, dalvec, makelogo) through a three-cycle information-maximization loop.

## When to use

Use when you have a GenBank file plus a tab-delimited sites file of genomic positions (e.g., ChIP-seq peak centers or transcription start sites) and need a binding-site logo and PWM.

## When NOT to use

- You need de novo motif discovery without genomic coordinates — the article contrasts Delila-PY with MEME (expectation maximization) and BioProspector (Gibbs sampling) tools
- Your input lacks a GenBank file or sites file with strand and base pair position; Delila-PY requires both before any processing
- The workflow requires Windows or other non-MacOS/non-Linux platforms — the README notes Delila-PY has been tested on MacOS and Linux (Ubuntu and CentOS) only

## Inputs

- GenBank file for the target organism (e.g., Rsphaeroides_GenBank.gbff from NCBI GenBank)
- Tab-delimited sites file with four columns (chromosome name, site name, strand, base pair position)
- Left boundary (upstream distance) and right boundary (downstream distance) relative to each genomic position
- Logo title string

## Outputs

- Position weight matrix (PWM) values as a text file
- DNA sequence logo in postscript and PDF formats
- Individual DNA sequences used to generate the motif
- dalvec output

## How to apply

Provide a GenBank file and a four-column tab-delimited sites file (chromosome, name, strand, base pair position) and run delila_pipeline.py with left/right boundaries relative to each site. The pipeline runs dbbk and catal to build libraries from the GenBank file, then delila generates a book of sequences. In cycle 1, malign and malin search for the alignment with the highest information content among all input sequences and feed results back to delila; in cycle 2, alist, encode, cmp, and rseq compute per-sequence information content, the ri program removes sequences with ≤0 information content, and delila regenerates an updated book; in cycle 3, alist, encode, cmp, rseq, dalvec, and makelogo produce the final DNA motif. The recommended execution mode is the glbrc/delila Docker image rather than compiling individual programs.

## Related tools

- **Delila-PY** (Custom Python3 pipeline orchestrating the Delila programs through three information-maximization cycles to build the PWM and logo) — https://github.com/GLBRC/delila
- **delila** (Generates the book of sequences from library and genomic position files; regenerates updated books after each cycle) — https://github.com/GLBRC/delila
- **malign** (Cycle 1: searches for the alignment with the highest information content among all input sequences) — https://github.com/GLBRC/delila
- **malin** (Cycle 1: its results are fed back to delila to generate an updated book file) — https://github.com/GLBRC/delila
- **alist** (Cycles 2 and 3: aligns sequences for information content calculation) — https://github.com/GLBRC/delila
- **encode** (Cycles 2 and 3: encodes aligned sequences for information analysis) — https://github.com/GLBRC/delila
- **cmp** (Cycles 2 and 3: computes composition information for the aligned sequences) — https://github.com/GLBRC/delila
- **rseq** (Cycles 2 and 3: computes individual sequence information content; values ≤0 trigger removal in cycle 2) — https://github.com/GLBRC/delila
- **ri** (Cycle 2: removes poorly matching sequences with ≤0 information content) — https://github.com/GLBRC/delila
- **dalvec** (Cycle 3: generates the DNA motif from individual sequence vectors) — https://github.com/GLBRC/delila
- **makelogo** (Cycle 3: renders the final DNA sequence logo) — https://github.com/GLBRC/delila
- **dbbk** (Generates required libraries from the GenBank file at pipeline start) — https://github.com/GLBRC/delila
- **catal** (Generates required catalogs from the GenBank file at pipeline start) — https://github.com/GLBRC/delila
- **Docker** (Recommended execution environment via the glbrc/delila image) — https://hub.docker.com/r/glbrc/delila
- **p2c** (Pascal-to-C translator used to compile the original Delila programs when building outside Docker) — https://github.com/FranklinChen/p2c

## Examples

```
delila_pipeline.py -g Rsphaeroides_GenBank.gbff -l -10 -r -10 -s Location_File_Example.txt -t "Logo Title"
```

## Evaluation signals

- Verify final-cycle artifacts exist: the PWM text file, the sequence logo in both postscript and PDF formats, and the dalvec output
- Compare the resulting sequence logo against a logo generated from previously identified binding sites (e.g., FnrL sites in R. sphaeroides); resemblance supports predictive power
- Confirm cycle-2 filtering removed all sequences with ≤0 information content via the ri program before the final cycle
- Check that sites-file chromosome names match the chromosome names in the supplied GenBank file
- If using Docker, verify the glbrc/delila image is pulled and the -v volume maps local input files into /opt/delila/example/container-data

## Limitations

- Tested only on MacOS and Linux (Ubuntu and CentOS); other operating systems may need additional work
- Requires matching chromosome names between the GenBank file and the sites file; mismatches will break processing
- Poorly matching sequences with ≤0 information content are discarded in cycle 2, so low-information sites are excluded from the final motif
- The discussion section of the source is only a changelog placeholder, so no additional limitations or discussion content were available

## Evidence

- [abstract] The first cycle uses the malign and malin programs to search for the alignment with the highest information content among all input sequences.: "The first cycle uses the malign and malin programs to search for the alignment with the highest information content among all input sequences."
- [abstract] Cycle 2 removes poorly matching sequences with ≤0 information content and regenerates the book.: "Delila-PY removes poorly matching sequences (≤0 information content, ri program). Then, a new file is generated and fed back to the delila program to generate an updated book file"
- [abstract] Final cycle programs that generate the DNA motif.: "The final cycle generates a DNA motif (alist, encode, cmp, rseq, dalvec, and makelogo)"
- [abstract] Delila-PY output artifacts.: "Delila-PY produces a DNA sequence logo in postscript and PDF formats, the position weight matrix (PWM) (11) values of the motif as a text file, and the individual DNA sequences used to generate the"
- [abstract] Validation of the pipeline against known binding sites.: "The resulting sequence logo from Delila-PY (Fig. 1B) resembles the logo generated from previously identified FnrL binding sites (Fig. 1C), supporting the utility and predictive power of Delila-PY"
- [readme] README description of required input files and tested platforms.: [evidence span withheld — anchor status: external_doc]
- [readme] Basic pipeline command from README.: [evidence span withheld — anchor status: external_doc]
