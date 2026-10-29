---
name: genbank-sequence-library-generation
description: 'Use when running Delila-PY on a new organism: you have an NCBI GenBank
  file (e.g., NC_007493.2 for R. sphaeroides) and must create the Delila libraries
  that the delila program consumes to build a book of sequences.'
when_to_use_negative:
- You do not yet have a GenBank file matching the chromosome names in your sites file
  (e.g., NC_007493.2 for R. sphaeroides) — dbbk/catal cannot build valid libraries
  without it.
- Your goal is de novo motif discovery from raw reads or already-aligned sequences
  rather than logo construction from genomic coordinates — tools like MEME or BioProspector
  may fit better.
- You already have valid Delila libraries for the organism and are moving directly
  to running delila on a sites file.
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0797
tools:
- name: Delila-PY
  role: Python3 pipeline wrapper (distributed as a Docker image) that orchestrates
    dbbk and catal as the library-generation step before the delila cycles
  repo: https://github.com/GLBRC/delila
- name: dbbk
  role: Runs with catal at the start of the pipeline to generate the required libraries
    from the GenBank file
  repo: https://github.com/GLBRC/delila
- name: catal
  role: Runs with dbbk to build the Delila libraries needed for subsequent book-of-sequence
    generation
  repo: https://github.com/GLBRC/delila
- name: Docker
  role: 'Recommended deployment: pull the glbrc/delila image so dbbk and catal run
    without manual compilation of individual programs'
  repo: https://docs.docker.com/engine/install/
- name: delila
  role: Downstream consumer of the generated libraries; uses them with the genomic
    position file to produce a book of sequences
  repo: https://github.com/GLBRC/delila
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/mra.00160-21
    title: Delila-PY, a Pipeline for Utilizing the Delila Suite of Software to Identify
      Potential DNA Binding Motifs
  source_content_hash: sha256:2b013964a08e69fcceeff09a1bed3d080dcd254c9028479c66734290aa123429
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
  - ri
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# genbank-sequence-library-generation

## Summary

Generate the Delila sequence libraries required by the Delila-PY pipeline from an organism's GenBank file using the dbbk and catal programs, the first prerequisite step before any binding-motif book or sequence logo can be produced.

## When to use

Use when running Delila-PY on a new organism: you have an NCBI GenBank file (e.g., NC_007493.2 for R. sphaeroides) and must create the Delila libraries that the delila program consumes to build a book of sequences.

## When NOT to use

- You do not yet have a GenBank file matching the chromosome names in your sites file (e.g., NC_007493.2 for R. sphaeroides) — dbbk/catal cannot build valid libraries without it.
- Your goal is de novo motif discovery from raw reads or already-aligned sequences rather than logo construction from genomic coordinates — tools like MEME or BioProspector may fit better.
- You already have valid Delila libraries for the organism and are moving directly to running delila on a sites file.

## Inputs

- GenBank file for the genome of interest (e.g., Rsphaeroides_GenBank.gbff, NC_007493.2), obtained from NCBI GenBank
- delila_pipeline.py with the scriptDir variable edited to point at the script directory

## Outputs

- Delila libraries generated from the GenBank file, consumed downstream by the delila program

## How to apply

Start from a GenBank file for the target organism downloaded from NCBI GenBank, such as the R. sphaeroides chromosome NC_007493.2. Before running delila, define the script directory by editing the scriptDir variable in delila_pipeline.py. Then run the dbbk and catal programs on the GenBank file, which the article identifies as the pipeline's starting step that generates the required libraries. These libraries, together with the user's tab-delimited sites file (chromosome, name, strand, base pair position) and left/right boundary distances (e.g., -l -10 -r -10), are then used by delila to generate the book of sequences. Correct library generation is a prerequisite for all three Delila-PY cycles (malign/malin alignment, ri filtering, and makelogo motif output).

## Related tools

- **Delila-PY** (Python3 pipeline wrapper (distributed as a Docker image) that orchestrates dbbk and catal as the library-generation step before the delila cycles) — https://github.com/GLBRC/delila
- **dbbk** (Runs with catal at the start of the pipeline to generate the required libraries from the GenBank file) — https://github.com/GLBRC/delila
- **catal** (Runs with dbbk to build the Delila libraries needed for subsequent book-of-sequence generation) — https://github.com/GLBRC/delila
- **Docker** (Recommended deployment: pull the glbrc/delila image so dbbk and catal run without manual compilation of individual programs) — https://docs.docker.com/engine/install/
- **delila** (Downstream consumer of the generated libraries; uses them with the genomic position file to produce a book of sequences) — https://github.com/GLBRC/delila

## Examples

```
delila_pipeline.py -g Rsphaeroides_GenBank.gbff -l -10 -r -10 -s Location_File_Example.txt -t "Logo Title"
```

## Evaluation signals

- The dbbk and catal runs complete without errors on the chosen GenBank file, producing library files in the configured script directory.
- The chromosome names in the downstream sites file (e.g., NC_007493.2) match the chromosome names in the GenBank file used for library generation.
- Subsequent execution of delila with the generated libraries and the tab-delimited sites file succeeds in producing a book of sequences.
- The delila_pipeline.py scriptDir variable was set before running, as the methods section requires prior to running delila.

## Limitations

- Libraries are organism-specific: the pipeline requires a GenBank file for the target organism, and chromosome names in the sites file must match the GenBank file exactly.
- The pipeline is tested on MacOS and Linux (Ubuntu and CentOS); other operating systems may require additional work.
- p2c is a prerequisite dependency (used to compile the Pascal-based Delila programs), and its translated output may need p2crc settings (MainType int, UseGets 0, -a flag) to compile under modern strict C compilers.

## Evidence

- [abstract] The pipeline starts by running the dbbk and catal programs to generate the required libraries from the GenBank file: "The pipeline starts by running the dbbk and catal programs to generate the required libraries from the GenBank file"
- [methods] Define script directory before running delila: "Prior to running delila you will need to define the script directory by editing the scriptDir varible in the delila_pipeline.py file."
- [readme] Sites file chromosome names must match GenBank: "chromosome name : must match the chromosome names in genbank file"
- [intro] GenBank file as required input: "GenBank file for your genome of interest (obtained from [NCBI GenBank](https://www.ncbi.nlm.nih.gov/genbank/))"
- [abstract] Delila libraries feed book generation: "The library and genomic position files are used by delila (note that delila is a specific program within the Delila-PY software suite) to generate a book of sequences"
