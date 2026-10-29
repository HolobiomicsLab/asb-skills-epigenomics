---
name: genbank-genome-library-construction
description: Use when you have a GenBank file for a target genome (e.g. R. sphaeroides
  NC_007493.2) and a tab-delimited sites file of genomic positions, and need to extract
  coordinate-windowed sequences into a Delila book artifact for motif construction.
when_to_use_negative:
- You lack a GenBank-format genome record — the pipeline starts from dbbk on a GenBank
  file, not from FASTA or other genome formats
- Sites file column names or chromosome labels do not match the GenBank chromosome
  names (e.g. NC_007493.2); delila extraction will not resolve the requested loci
- The goal is motif discovery without defined genomic coordinates — this skill only
  extracts position-relative windows; use de novo motif finders like MEME or BioProspector
  instead
edam_operation: http://edamontology.org/operation_2402
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0749
tools:
- name: dbbk
  role: Processes the GenBank genome record into Delila database libraries
  repo: https://github.com/GLBRC/delila
- name: catal
  role: Generates the required catalyzed libraries from the dbbk output
  repo: https://github.com/GLBRC/delila
- name: delila
  role: Uses the library and genomic position files to extract sequences and generate
    the book of sequences
  repo: https://github.com/GLBRC/delila
- name: Delila-PY
  role: Python3 pipeline wrapper that orchestrates dbbk, catal, and delila with the
    standard parameters
  repo: https://github.com/GLBRC/delila
- name: Docker
  role: Recommended execution environment via the glbrc/delila image, avoiding compilation
    of individual programs
  repo: https://hub.docker.com/r/glbrc/delila
- name: Python3
  role: Language of delila_pipeline.py; scriptDir must be set before running delila
- name: p2c
  role: Pascal-to-C translator used to build the underlying Delila programs
  repo: https://github.com/FranklinChen/p2c
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1128/mra.00160-21
    title: Delila-PY, a Pipeline for Utilizing the Delila Suite of Software to Identify
      Potential DNA Binding Motifs
  source_content_hash: sha256:cf86ae044ee90cf8945f375644a2698fd64ffd7fc03a967d7887dd05b4fc7db8
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
  - p2c
  edam_operation: http://edamontology.org/operation_2402
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# genbank-genome-library-construction

## Summary

Constructs Delila database libraries from a GenBank genome file and uses them with a genomic positions file to extract sequences into a 'book of sequences' for downstream binding-motif analysis, as implemented in the Delila-PY pipeline.

## When to use

Use when you have a GenBank file for a target genome (e.g. R. sphaeroides NC_007493.2) and a tab-delimited sites file of genomic positions, and need to extract coordinate-windowed sequences into a Delila book artifact for motif construction.

## When NOT to use

- You lack a GenBank-format genome record — the pipeline starts from dbbk on a GenBank file, not from FASTA or other genome formats
- Sites file column names or chromosome labels do not match the GenBank chromosome names (e.g. NC_007493.2); delila extraction will not resolve the requested loci
- The goal is motif discovery without defined genomic coordinates — this skill only extracts position-relative windows; use de novo motif finders like MEME or BioProspector instead

## Inputs

- GenBank file for the genome of interest (e.g. Rsphaeroides_GenBank.gbff from NCBI GenBank)
- Tab-delimited genomic positions (sites) file with four columns and no column names: chromosome name, site name, strand (forward/reverse), base pair position

## Outputs

- Delila database libraries produced by dbbk and catalyzed libraries from catal
- Book of sequences containing one sequence entry per row of the genomic position file, for downstream malign/malin and motif-generation cycles

## How to apply

Run the dbbk program on the GenBank file to process the genome record into Delila database libraries, then run catal to generate the required catalyzed libraries from the dbbk output. Supply the library files together with a tab-delimited genomic position file (four columns, no header: chromosome name matching the GenBank record, unique site name, strand as forward/reverse, base pair position) to the delila program, which extracts sequences using user-specified left and right boundaries relative to each position (set via -l and -r flags, e.g. -l -10 -r -10) and emits the book of sequences. Before running delila, define the script directory by editing the scriptDir variable in delila_pipeline.py; the recommended execution environment is the glbrc/delila Docker image rather than compiling individual programs (p2c is used to translate Pascal sources).

## Related tools

- **dbbk** (Processes the GenBank genome record into Delila database libraries) — https://github.com/GLBRC/delila
- **catal** (Generates the required catalyzed libraries from the dbbk output) — https://github.com/GLBRC/delila
- **delila** (Uses the library and genomic position files to extract sequences and generate the book of sequences) — https://github.com/GLBRC/delila
- **Delila-PY** (Python3 pipeline wrapper that orchestrates dbbk, catal, and delila with the standard parameters) — https://github.com/GLBRC/delila
- **Docker** (Recommended execution environment via the glbrc/delila image, avoiding compilation of individual programs) — https://hub.docker.com/r/glbrc/delila
- **Python3** (Language of delila_pipeline.py; scriptDir must be set before running delila)
- **p2c** (Pascal-to-C translator used to build the underlying Delila programs) — https://github.com/FranklinChen/p2c

## Examples

```
delila_pipeline.py -g Rsphaeroides_GenBank.gbff -l -10 -r -10 -s Location_File_Example.txt -t "Logo Title"
```

## Evaluation signals

- The book of sequences contains exactly one sequence entry per row of the genomic position file
- The chromosome name (e.g. NC_007493.2) is present in the extracted sequence records
- dbbk and catal complete without errors and produce the required database and catalyzed library files
- Extracted windows match the requested left/right boundaries (e.g. -l -10 -r -10 gives 21 bp around each position on the specified strand)
- Example sites such as 'NC_007493.2 site_1 forward 409225' resolve to the correct base position in the book

## Limitations

- Tested on MacOS and Linux (Ubuntu and CentOS); other operating systems may require additional work
- Chromosome names in the sites file must exactly match names in the GenBank file
- Strand must be written as the words 'forward' or 'reverse'; boundaries are always interpreted on the given strand, which can be counterintuitive when searching upstream elements on the reverse strand

## Evidence

- [abstract] dbbk and catal produce libraries; delila produces the book: "The library and genomic position files are used by delila (note that delila is a specific program within the Delila-PY software suite) to generate a book of sequences"
- [abstract] dbbk and catal generate libraries from the GenBank file: "The pipeline starts by running the dbbk and catal programs to generate the required libraries from the GenBank file"
- [readme] Sites file format from README: "A sites file which is tab delimited with four columns (no column names): chromosome name : must match the chromosome names in genbank file"
- [methods] Script directory setup from methods: "Prior to running delila you will need to define the script directory by editing the scriptDir varible in the delila_pipeline.py file."
