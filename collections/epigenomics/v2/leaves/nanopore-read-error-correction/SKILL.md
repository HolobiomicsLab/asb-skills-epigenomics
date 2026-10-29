---
name: nanopore-read-error-correction
description: Use when assembling genomes from noisy Oxford Nanopore long reads and
  you must choose between correct-then-assemble vs assemble-then-correct correction
  strategies, or when polishing a draft long-read assembly to final accuracy with
  Illumina short reads.
when_to_use_negative:
- Input reads are already high-accuracy (e.g. Illumina-only short reads) and no long-read
  error problem exists
- The goal is basecaller model training or raw signal basecalling rather than read
  correction/polishing (use taiyaki/bonito instead)
- You need modified-base detection or methylation calling rather than assembly-grade
  read correction
edam_operation: http://edamontology.org/operation_3194
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: jts/nanopore-paper-analysis
  role: End-to-end Makefile pipeline (full-pipeline.make) that downloads ENA data,
    fetches software, runs read correction, assembly, and polishing to reproduce the
    2015 Nature Methods E. coli assembly
  repo: https://github.com/jts/nanopore-paper-analysis
- name: Illumina
  role: Short-read sequencing technology whose data serve as the high-accuracy input
    for error-correcting the noisy nanopore long reads
- name: Oxford Nanopore
  role: Long-read sequencing technology producing the high-error-rate reads that require
    correction before or after assembly
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.xinn.2021.100153
    title: Applications and potentials of nanopore sequencing in the (epi)genome and
      (epi)transcriptome era
  source_content_hash: sha256:a23069087538f95502e2a3d035b7915b16746f510a0b817139f74e34fe12276c
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
  edam_operation: http://edamontology.org/operation_3194
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# nanopore-read-error-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Correct the high raw error rate of Oxford Nanopore long reads — either before assembly (correct-then-assemble) or after contig construction (assemble-then-correct) — typically using complementary short-read data, to yield an accurate genome assembly such as the polished E. coli genome from the 2015 Nature Methods nanopore paper.

## When to use

Use when assembling genomes from noisy Oxford Nanopore long reads and you must choose between correct-then-assemble vs assemble-then-correct correction strategies, or when polishing a draft long-read assembly to final accuracy with Illumina short reads.

## When NOT to use

- Input reads are already high-accuracy (e.g. Illumina-only short reads) and no long-read error problem exists
- The goal is basecaller model training or raw signal basecalling rather than read correction/polishing (use taiyaki/bonito instead)
- You need modified-base detection or methylation calling rather than assembly-grade read correction

## Inputs

- Oxford Nanopore long sequencing reads (downloaded from ENA by the pipeline)
- Illumina short-read dataset used to error-correct the nanopore reads
- Python virtualenv with GCC >= 4.8
- Repository full-pipeline.make Makefile with required software versions

## Outputs

- Error-corrected nanopore reads (intermediate correction stage)
- Draft long-read assembly contigs (intermediate assemble stage)
- polished_genome.fasta — final polished high-accuracy E. coli genome assembly

## How to apply

Decide on a correction strategy first: the article distinguishes 'correct-then-assemble' (error-correct long nanopore reads with short reads before the assembler) from 'assemble-then-correct' (assemble raw long reads into contigs, then polish). In the jts/nanopore-paper-analysis pipeline, correction and polishing are driven end-to-end by the full-pipeline.make Makefile: it downloads the E. coli nanopore reads from ENA, fetches pinned versions of all required software, error-corrects the nanopore reads using the Illumina short-read dataset, assembles the corrected long reads into contigs, and runs a final polishing stage to produce polished_genome.fasta. Run it inside a Python virtualenv with GCC >= 4.8, sizing the single CORES parameter to the available CPU count. Judge success by the presence of a valid polished_genome.fasta whose contiguity and size match the published 2015 assembly; the repository's full-walkthrough IPython notebook reproduces the exact commands and figures.

## Related tools

- **jts/nanopore-paper-analysis** (End-to-end Makefile pipeline (full-pipeline.make) that downloads ENA data, fetches software, runs read correction, assembly, and polishing to reproduce the 2015 Nature Methods E. coli assembly) — https://github.com/jts/nanopore-paper-analysis
- **Illumina** (Short-read sequencing technology whose data serve as the high-accuracy input for error-correcting the noisy nanopore long reads)
- **Oxford Nanopore** (Long-read sequencing technology producing the high-error-rate reads that require correction before or after assembly)

## Examples

```
make -f full-pipeline.make CORES=64 polished_genome.fasta
```

## Evaluation signals

- polished_genome.fasta exists and is a valid FASTA file
- Assembly contiguity and total size are consistent with the polished E. coli genome reported in the 2015 Nature Methods analysis
- The pipeline completes using the exact command 'make -f full-pipeline.make CORES=64 polished_genome.fasta' with the paper-tagged repository version
- Re-running the full-walkthrough IPython notebook reproduces the commands, results, and figures from the paper
- Correction stage consumed the same public ENA input reads used in the original 2015 analysis

## Limitations

- The full-pipeline.make Makefile requires GCC >= 4.8 and must be run within a Python virtualenv
- Only the 'paper'-tagged repository version is guaranteed to reproduce the article's results; post-publication merged patches clean up code but functionality is unchanged
- The article notes assembly algorithms should be optimized for nanopore reads, and new alignment algorithms are needed for ultra-long reads — generic short-read tools may underperform
- Correction is CPU-intensive; the single CORES parameter controls how many cores are used for the assembly

## Evidence
