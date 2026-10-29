---
name: long-read-genome-assembly
description: Use when the goal is to regenerate or produce a polished genome assembly
  from nanopore long reads (optionally with Illumina short reads for correction/polishing)
  using a reproducible, end-to-end pipeline.
when_to_use_negative:
- The goal is basecalling or model training from raw FAST5 signal data (use taiyaki/bonito
  instead)
- Reads are short-read-only (Illumina) with no long-read data available
- A quick draft assembly without reproducible software versions is acceptable and
  the full ENA-download pipeline is unnecessary overhead
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Oxford Nanopore
  role: Source of the long nanopore sequencing reads assembled into the genome
- name: Illumina
  role: Short-read dataset used to error-correct the nanopore long reads before assembly/polishing
- name: jts/nanopore-paper-analysis
  role: Repository containing full-pipeline.make, which downloads data/software and
    runs the complete correct-polish assembly pipeline
  repo: https://github.com/jts/nanopore-paper-analysis
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.xinn.2021.100153
    title: Applications and potentials of nanopore sequencing in the (epi)genome and
      (epi)transcriptome era
  source_content_hash: sha256:df9c09e5f8a4a4470c5c625609a6b85193fac11adf8a8306acede18272aa1764
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/jts/nanopore-paper-analysis
  repo_url: https://github.com/jts/nanopore-paper-analysis
  tools:
  - Illumina
  - Oxford Nanopore
  - jts/nanopore-paper-analysis
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# long-read-genome-assembly

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assembles a complete genome from Oxford Nanopore long reads using a Makefile-driven pipeline (correct-then-assemble or assemble-then-correct strategies) with short-read error correction and polishing, reproducing the 2015 Nature Methods E. coli nanopore assembly.

## When to use

Use when the goal is to regenerate or produce a polished genome assembly from nanopore long reads (optionally with Illumina short reads for correction/polishing) using a reproducible, end-to-end pipeline.

## When NOT to use

- The goal is basecalling or model training from raw FAST5 signal data (use taiyaki/bonito instead)
- Reads are short-read-only (Illumina) with no long-read data available
- A quick draft assembly without reproducible software versions is acceptable and the full ENA-download pipeline is unnecessary overhead

## Inputs

- Nanopore long-read sequencing reads (downloaded from ENA)
- Illumina short-read dataset for error correction
- full-pipeline.make Makefile from jts/nanopore-paper-analysis
- CORES parameter (CPU core count, e.g. 64)

## Outputs

- polished_genome.fasta — final polished genome assembly (valid FASTA contigs)

## How to apply

Run the repository's full-pipeline.make Makefile, which downloads the input nanopore reads from the ENA archive and fetches the exact software versions used in the 2015 Nature Methods analysis. Apply short-read error correction to the long nanopore reads (correct-then-assemble stage), assemble the corrected long reads into contigs with the Makefile-invoked long-read assembler, and run the polishing stage to yield the final product. The pipeline requires GCC >= 4.8, should run inside a Python virtualenv, and takes a single CORES parameter controlling how many CPU cores are used for assembly. Choose the assembly strategy according to read quality: 'correct-then-assemble' versus 'assemble-then-correct' are the two documented long-read assembly strategies for nanopore sequencing.

## Related tools

- **Oxford Nanopore** (Source of the long nanopore sequencing reads assembled into the genome)
- **Illumina** (Short-read dataset used to error-correct the nanopore long reads before assembly/polishing)
- **jts/nanopore-paper-analysis** (Repository containing full-pipeline.make, which downloads data/software and runs the complete correct-polish assembly pipeline) — https://github.com/jts/nanopore-paper-analysis

## Examples

```
make -f full-pipeline.make CORES=64 polished_genome.fasta
```

## Evaluation signals

- polished_genome.fasta exists in the working directory after 'make -f full-pipeline.make CORES=64 polished_genome.fasta' completes
- Output file is a valid FASTA with contiguity/size consistent with the polished E. coli assembly reported in the 2015 Nature Methods analysis
- Pipeline runs within a Python virtualenv on a system with GCC >= 4.8 without version errors
- Input reads downloaded from the ENA match the public data used in the 2015 analysis
- The 'paper' tag of the repository (jts/nanopore-paper-analysis) reproduces the published results when functionality-affecting changes are absent

## Limitations

- Pipeline requires GCC >= 4.8 and a Python virtualenv; untested on non-Ubuntu-like platforms
- Assembly performance depends on read quality — assembly algorithms should be optimized for nanopore reads, and ultra-long reads require appropriate alignment algorithms
- The repository retains the 'paper' tag for exact published results; post-publication patches only clean code, but environment drift can still affect reproducibility

## Evidence

- [readme] full-pipeline.make Makefile reproduces the paper's assembly: "The full-pipeline.make Makefile will download the input data from the ENA, download the versions of all required software and run the complete assembly pipeline."
- [readme] command to generate the assembly: "This command will generate the assembly:

    make -f full-pipeline.make CORES=64 polished_genome.fasta"
