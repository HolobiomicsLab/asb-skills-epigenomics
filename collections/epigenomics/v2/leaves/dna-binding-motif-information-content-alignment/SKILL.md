---
name: dna-binding-motif-information-content-alignment
description: Use when you have a GenBank genome and a set of genomic positions (e.g.,
  ChIP-chip/ChIP-seq peak centers or transcription start sites) and need to discover
  or reproduce a DNA binding motif as a sequence logo and PWM.
when_to_use_negative:
- You lack both a GenBank genome file and a coordinate-anchored sites file — Delila-PY
  requires exactly these two inputs and cannot work from raw reads or an unanchored
  sequence set.
- Your goal is de novo motif discovery from sequences with no genomic coordinates
  or strand information; the sites file format (chromosome, name, strand, base pair
  position) is mandatory.
- You need an interactive, hand-tuned motif analysis — Delila-PY uses standard (often
  default) Delila parameter values for quick, reproducible runs, sacrificing the full
  flexibility of running the Delila programs individually.
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_3125
- http://edamontology.org/topic_0622
tools:
- name: Delila-PY
  role: Python3 pipeline wrapper that orchestrates the Delila programs end to end
  repo: https://github.com/GLBRC/delila
- name: Docker
  role: Recommended execution environment via the glbrc/delila Docker image
  repo: https://hub.docker.com/r/glbrc/delila
- name: dbbk
  role: Generates the required Delila libraries from the GenBank file
- name: catal
  role: Generates the required Delila libraries alongside dbbk
- name: delila
  role: Uses the library and genomic position files to generate the book of sequences
- name: malign
  role: 'Cycle 1: searches for the alignment with the highest information content
    among all input sequences'
- name: ri
  role: 'Cycle 2 filter: removes poorly matching sequences with information content
    of 0 bits or less'
- name: makelogo
  role: 'Cycle 3: generates the final sequence logo'
- name: p2c
  role: Pascal-to-C translator used to build the Delila programs from their original
    Pascal source
  repo: https://github.com/FranklinChen/p2c
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/mra.00160-21
    title: Delila-PY, a Pipeline for Utilizing the Delila Suite of Software to Identify
      Potential DNA Binding Motifs
  source_content_hash: sha256:9235186c284f5d6f28119e90b7de1d3a7c7ee317e0e374ecfd1183a352d68efe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GLBRC/delila
  tools:
  - Delila-PY
  - Docker
  - catal
  - dbbk
  - delila
  - makelogo
  - malign
  - p2c
  - ri
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_3125
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# dna-binding-motif-information-content-alignment

## Summary

A Delila-PY–based three-cycle information-theory workflow that aligns genomic binding-site sequences to maximize information content and produces a DNA sequence logo and PWM. It was validated by reproducing the R. sphaeroides FnrL binding motif from ChIP-chip-derived genomic locations.

## When to use

Use when you have a GenBank genome and a set of genomic positions (e.g., ChIP-chip/ChIP-seq peak centers or transcription start sites) and need to discover or reproduce a DNA binding motif as a sequence logo and PWM.

## When NOT to use

- You lack both a GenBank genome file and a coordinate-anchored sites file — Delila-PY requires exactly these two inputs and cannot work from raw reads or an unanchored sequence set.
- Your goal is de novo motif discovery from sequences with no genomic coordinates or strand information; the sites file format (chromosome, name, strand, base pair position) is mandatory.
- You need an interactive, hand-tuned motif analysis — Delila-PY uses standard (often default) Delila parameter values for quick, reproducible runs, sacrificing the full flexibility of running the Delila programs individually.

## Inputs

- GenBank file for the target organism (e.g., NC_007493.2 from NCBI GenBank)
- Tab-delimited sites file with four columns and no column names: chromosome name, unique site name, strand (forward/reverse), base pair position
- Left and right boundary offsets relative to each site position (e.g., -l -10 -r -10)
- Logo title string (optional, passed via -t)

## Outputs

- DNA sequence logo in PostScript and PDF formats (makelogo)
- Position weight matrix (PWM) values as a text file
- Individual DNA sequences used to generate the motif

## How to apply

Configure scriptDir in delila_pipeline.py, then run dbbk and catal on the GenBank file (e.g., NC_007493.2 for R. sphaeroides) to build Delila libraries. Supply a tab-delimited, four-column sites file (chromosome, name, strand, base pair position) and set left/right boundaries relative to each position via -l/-r (e.g., -l -10 -r -10 for symmetric search around a peak center). Run delila to produce the book of sequences, then cycle 1 (malign/malin) finds the alignment with the highest information content and regenerates the book; cycle 2 computes per-sequence information content with alist, encode, cmp, and rseq, and drops sequences at 0 bits or below using the ri program before regenerating the book; cycle 3 runs alist, encode, cmp, rseq, dalvec, and makelogo to produce the final motif. The rationale is iterative refinement: poor matches are pruned on an information-content threshold so the final logo reflects only sequences contributing positive information.

## Related tools

- **Delila-PY** (Python3 pipeline wrapper that orchestrates the Delila programs end to end) — https://github.com/GLBRC/delila
- **Docker** (Recommended execution environment via the glbrc/delila Docker image) — https://hub.docker.com/r/glbrc/delila
- **dbbk** (Generates the required Delila libraries from the GenBank file)
- **catal** (Generates the required Delila libraries alongside dbbk)
- **delila** (Uses the library and genomic position files to generate the book of sequences)
- **malign** (Cycle 1: searches for the alignment with the highest information content among all input sequences)
- **ri** (Cycle 2 filter: removes poorly matching sequences with information content of 0 bits or less)
- **makelogo** (Cycle 3: generates the final sequence logo)
- **p2c** (Pascal-to-C translator used to build the Delila programs from their original Pascal source) — https://github.com/FranklinChen/p2c

## Examples

```
delila_pipeline.py -g Rsphaeroides_GenBank.gbff -l -10 -r -10 -s Location_File_Example.txt -t "Logo Title"
```

## Evaluation signals

- The logo output file exists in both PostScript and PDF formats.
- The PWM text file is parseable, with per-position information content within the 0–2 bits range.
- The final logo (Fig. 1B) visually resembles the logo from previously identified FnrL binding sites (Fig. 1C).
- Cycle 2 correctly removes only sequences with information content of 0 bits or less before regenerating the book with delila.
- Running the provided example files in the Docker container completes without errors before applying the pipeline to new data.

## Limitations

- Tested only on MacOS and Linux (Ubuntu and CentOS); other operating systems require additional work.
- Delila-PY uses standard (often default) values, providing less flexibility and control than manually running the Delila programs individually.
- The paper's discussion section provides no additional guidance (only a changelog placeholder), so troubleshooting beyond the README is limited.
- Boundary choice (-l/-r) is strand-relative and scenario-dependent (e.g., -20/0 for a -10 promoter element vs -10/+10 around a peak center); wrong offsets will mis-center the motif.

## Evidence

- [abstract] Cycle 2 prunes low-information sequences: "Delila-PY removes poorly matching sequences (≤0 information content, ri program). Then, a new file is generated and fed back to the delila program"
- [abstract] Pipeline start: libraries from GenBank: "The pipeline starts by running the dbbk and catal programs to generate the required libraries from the GenBank file"
- [abstract] Validated logo resembles known FnrL motif: "The resulting sequence logo from Delila-PY (Fig. 1B) resembles the logo generated from previously identified FnrL binding sites (Fig. 1C)"
- [readme] Sites file format from README: "A sites file which is tab delimited with four columns (no column names): chromosome name, name, strand, position"
- [readme] Boundary semantics from README: [evidence span withheld — anchor status: external_doc]
- [abstract] Cycle 1 alignment search: "The first cycle uses the malign and malin programs to search for the alignment with the highest information content among all input sequences"
