---
name: multiple-alignment-information-maximization
description: Use when you have a delila-generated book of unaligned input sequences
  and need to find the multiple alignment that maximizes information content before
  calculating per-sequence information or building a sequence logo.
when_to_use_negative:
- You do not yet have a delila-generated book of sequences — the pipeline must first
  run dbbk, catal, and delila on the GenBank file and sites file
- You only need a final motif/logo and already possess an independently verified alignment
  of binding sites — cycle 1 re-alignment is unnecessary and cycle 3 tools (dalvec,
  makelogo) suffice
- You want de novo motif discovery from raw reads with expectation maximization or
  Gibbs sampling — tools like MEME or BioProspector are the article's cited alternatives
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3168
tools:
- name: malign
  role: Searches among all input sequences for the alignment with the highest information
    content in cycle 1
  repo: https://github.com/GLBRC/delila
- name: malin
  role: Processes malign's alignment output; its results are fed back to delila to
    generate an updated book file
  repo: https://github.com/GLBRC/delila
- name: delila
  role: Consumes malin results to regenerate the updated book of sequences used in
    subsequent cycles
  repo: https://github.com/GLBRC/delila
- name: Delila-PY
  role: Python3/Docker pipeline that orchestrates the malign → malin → delila feedback
    loop as cycle 1 of three
  repo: https://github.com/GLBRC/delila
- name: p2c
  role: Pascal-to-C translator used to build the Delila suite programs when compiling
    outside Docker
  repo: https://github.com/FranklinChen/p2c
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1128/mra.00160-21
    title: Delila-PY, a Pipeline for Utilizing the Delila Suite of Software to Identify
      Potential DNA Binding Motifs
  source_content_hash: sha256:51ba5308645640b87aabd7b0db969cd1d98c785d9f3dc1d3d7e0fa68ad3015f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GLBRC/delila
  tools:
  - Delila-PY
  - Docker
  - Python3
  - alist
  - cmp
  - dalvec
  - delila
  - encode
  - makelogo
  - malign
  - malin
  - p2c
  - ri
  - rseq
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# multiple-alignment-information-maximization

## Summary

Cycle 1 of the Delila-PY pipeline: the malign and malin programs search among all input sequences for the multiple alignment with the highest information content, and malin's results are fed back to delila to generate an updated book of sequences for downstream motif construction.

## When to use

Use when you have a delila-generated book of unaligned input sequences and need to find the multiple alignment that maximizes information content before calculating per-sequence information or building a sequence logo.

## When NOT to use

- You do not yet have a delila-generated book of sequences — the pipeline must first run dbbk, catal, and delila on the GenBank file and sites file
- You only need a final motif/logo and already possess an independently verified alignment of binding sites — cycle 1 re-alignment is unnecessary and cycle 3 tools (dalvec, makelogo) suffice
- You want de novo motif discovery from raw reads with expectation maximization or Gibbs sampling — tools like MEME or BioProspector are the article's cited alternatives

## Inputs

- delila-generated book of sequences (produced from a GenBank file and a tab-delimited sites file of chromosome, name, strand, and base pair position)
- GenBank file for the target organism (obtained from NCBI GenBank)
- User-specified left and right boundaries relative to a genomic position (e.g., -l/-r offsets around a ChIP-seq peak center or transcription start site)

## Outputs

- malin results fed back to delila to generate an updated book file of re-aligned sequences
- Intermediate alignment suitable for cycle-2 per-sequence information content calculation (alist, encode, cmp, rseq)

## How to apply

Run the malign program on the delila-generated book of sequences to search for the alignment with the highest information content among all input sequences; malign explores candidate alignments rather than accepting the input ordering. Then run malin on malign's output and feed malin's results back to the delila program to generate an updated book file reflecting the improved alignment. This is the first of three cycles in Delila-PY; the updated book produced here is the input for cycle 2 (alist, encode, cmp, rseq per-sequence information calculation and ri-based removal of sequences with ≤0 information content). The rationale is that downstream per-sequence filtering and motif generation (dalvec, makelogo) are only meaningful once the alignment itself has been optimized for information content.

## Related tools

- **malign** (Searches among all input sequences for the alignment with the highest information content in cycle 1) — https://github.com/GLBRC/delila
- **malin** (Processes malign's alignment output; its results are fed back to delila to generate an updated book file) — https://github.com/GLBRC/delila
- **delila** (Consumes malin results to regenerate the updated book of sequences used in subsequent cycles) — https://github.com/GLBRC/delila
- **Delila-PY** (Python3/Docker pipeline that orchestrates the malign → malin → delila feedback loop as cycle 1 of three) — https://github.com/GLBRC/delila
- **p2c** (Pascal-to-C translator used to build the Delila suite programs when compiling outside Docker) — https://github.com/FranklinChen/p2c

## Examples

```
delila_pipeline.py -g Rsphaeroides_GenBank.gbff -l -10 -r -10 -s Location_File_Example.txt -t "Logo Title"
```

## Evaluation signals

- malin output can be fed back to delila without errors and produces a new book file (pipeline does not stall in cycle 1)
- Information content of the updated alignment should be at least as high as the initial ordering, since malign searches for the highest-information alignment
- The updated book must contain all input sequences still (per-sequence removal of ≤0-information sequences happens later, via ri in cycle 2)
- Downstream validation: the final-cycle sequence logo should resemble the logo from previously identified binding sites (e.g., FnrL), confirming the cycle-1 alignment was sound
- Final-cycle artifacts exist and are non-empty: PWM text file, sequence logo in postscript and PDF formats, dalvec output

## Limitations

- Delila-PY has been tested only on MacOS and Linux (Ubuntu and CentOS); additional work may be required on other operating systems
- The alignment search uses standard (often default) program values, providing less flexibility than hand-tuned Delila runs despite the suite's inherent flexibility
- Chromosome names in the sites file must exactly match the chromosome names in the GenBank file or delila cannot generate the book to be aligned

## Evidence

- [abstract] Cycle 1: use malign and malin to find the alignment with the highest information content among all input sequences, feed results back to delila to generate an updated book file: "The first cycle uses the malign and malin programs to search for the alignment with the highest information content among all input sequences."
- [abstract] malin results are fed back to delila to generate an updated book file: "The results of the malin program are fed back to the"
- [abstract] Delila-PY runs the Delila programs from Tom Schneider's lab, wrapped in a Python3/Docker pipeline for reproducibility: [evidence span withheld — anchor status: not_found]
- [abstract] delila generates the book of sequences that cycle 1 aligns, from library and genomic position files: "The library and genomic position files are used by delila (note that delila is a specific program within the Delila-PY software suite) to generate a book of sequences"
- [readme] Delila-PY is intended for quickly and reproducibly processing a file of genomic locations into a binding site logo: "Delila-PY uses standard (often default) values and allows the user to quickly and reproducibly process a file consisting of genomic locations to identify a potential logo."
