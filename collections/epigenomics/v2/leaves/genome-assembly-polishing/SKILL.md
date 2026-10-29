---
name: genome-assembly-polishing
description: Use when the goal is to regenerate or replicate a published long-read
  (Oxford Nanopore) genome assembly pipeline from a released repository, specifically
  the polished E. coli assembly from the 2015 Nature Methods nanopore article.
when_to_use_negative:
- You only need basecalling model training or modified-base detection — use nanoporetech/taiyaki
  or nanoporetech/bonito instead of this assembly pipeline.
- Your data is not the public E. coli nanopore/short-read pair from the 2015 paper
  and you have not adapted the Makefile's ENA download and pinned software versions.
- You need a reimplementation of individual correction/assembly/polishing tools rather
  than a reproducible Makefile-driven run of the exact published pipeline.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: full-pipeline.make (jts/nanopore-paper-analysis)
  role: Master Makefile that downloads ENA input data, fetches required software versions,
    and runs the correct-then-assemble/assemble-then-correct pipeline to generate
    polished_genome.fasta
  repo: https://github.com/jts/nanopore-paper-analysis
- name: Oxford Nanopore
  role: Long-read sequencing data source for the E. coli assembly being reproduced
- name: Illumina
  role: Short-read data used in the pipeline's correction stage to error-correct nanopore
    reads before assembly
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.xinn.2021.100153
    title: Applications and potentials of nanopore sequencing in the (epi)genome and
      (epi)transcriptome era
  source_content_hash: sha256:8262170063209893a4088a22ee019ff1bcee7a48b46ccf023bf8a5b102645e71
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
  - full-pipeline.make (jts/nanopore-paper-analysis)
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# genome-assembly-polishing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the polished nanopore E. coli genome assembly from the 2015 Nature Methods paper by running the jts/nanopore-paper-analysis repository's end-to-end Makefile, which error-corrects long reads, assembles contigs, and polishes them into polished_genome.fasta.

## When to use

Use when the goal is to regenerate or replicate a published long-read (Oxford Nanopore) genome assembly pipeline from a released repository, specifically the polished E. coli assembly from the 2015 Nature Methods nanopore article.

## When NOT to use

- You only need basecalling model training or modified-base detection — use nanoporetech/taiyaki or nanoporetech/bonito instead of this assembly pipeline.
- Your data is not the public E. coli nanopore/short-read pair from the 2015 paper and you have not adapted the Makefile's ENA download and pinned software versions.
- You need a reimplementation of individual correction/assembly/polishing tools rather than a reproducible Makefile-driven run of the exact published pipeline.

## Inputs

- E. coli Oxford Nanopore long-read dataset (downloaded automatically from the ENA by the Makefile)
- Illumina short-read dataset used for error correction
- jts/nanopore-paper-analysis repository checkout (preferably the 'paper' tag)

## Outputs

- polished_genome.fasta — the final polished high-accuracy E. coli assembly
- Intermediate error-corrected reads and draft contigs produced by the pipeline stages

## How to apply

Clone the jts/nanopore-paper-analysis repository (the 'paper' tag holds the exact code used in the publication). Run `make -f full-pipeline.make CORES=64 polished_genome.fasta` from a Python virtualenv on a system with GCC >= 4.8; the Makefile downloads the E. coli nanopore reads from the ENA, fetches the pinned software versions, and drives three stages: short-read-based error correction of long reads (correct-then-assemble), long-read assembly into contigs, and polishing of the draft to produce polished_genome.fasta. The single CORES parameter controls CPU parallelism; the accompanying full-walkthrough IPython notebook documents the exact commands and figures from the paper for cross-checking.

## Related tools

- **full-pipeline.make (jts/nanopore-paper-analysis)** (Master Makefile that downloads ENA input data, fetches required software versions, and runs the correct-then-assemble/assemble-then-correct pipeline to generate polished_genome.fasta) — https://github.com/jts/nanopore-paper-analysis
- **Oxford Nanopore** (Long-read sequencing data source for the E. coli assembly being reproduced)
- **Illumina** (Short-read data used in the pipeline's correction stage to error-correct nanopore reads before assembly)

## Examples

```
make -f full-pipeline.make CORES=64 polished_genome.fasta
```

## Evaluation signals

- polished_genome.fasta is produced by `make -f full-pipeline.make CORES=64 polished_genome.fasta` without pipeline errors.
- The output is a valid FASTA file whose contiguity and total size are consistent with the polished E. coli assembly reported in the 2015 Nature Methods analysis.
- The run uses the 'paper'-tagged repository version or an equivalent, since post-publication patches cleaned code but 'no changes to functionality have been made'.
- Environment preconditions hold: GCC >= 4.8 and execution within a Python virtualenv, as the pipeline requires.
- The intermediate stages (EN data download from ENA, read correction, assembly, polishing) each complete, mirroring the steps demonstrated in the full-walkthrough IPython notebook.

## Limitations

- The pipeline takes a single CORES parameter, so scaling behavior is limited to CPU-core parallelism of the assembly steps.
- Exact reproduction depends on the pinned software versions fetched by the Makefile and on continued ENA availability of the original input data.
- The strategy is long-read-specific: the article notes there are two nanopore assembly strategies, 'correct-then-assemble' and 'assemble-then-correct', and this pipeline implements a hybrid of correction with short reads followed by long-read assembly.

## Evidence

- [readme] The full-pipeline.make Makefile will download the input data from the ENA, download the versions of all required software and run the complete assembly pipeline.: "The full-pipeline.make Makefile will download the input data from the ENA, download the versions of all required software and run the complete assembly pipeline."
- [readme] This pipeline requires GCC >= 4.8 and should be run within a Python virtualenv.: "This pipeline requires GCC >= 4.8 and should be run within a Python virtualenv."
- [readme] The IPython notebook full-walkthrough demonstrates the use of the pipeline, starting from a stock Ubuntu 14.04 system.: "The IPython notebook full-walkthrough in this repository demonstrates the use of the pipeline, starting from a stock Ubuntu 14.04 system."
- [intro] There are two strategies for long-read genome assembly of nanopore sequencing, namely correct-then-assemble and assemble-then-correct.: "there are two strategies for long-read genome assembly of nanopore sequencing, namely “correct-then-assemble” and “assemble-then-correct”"
