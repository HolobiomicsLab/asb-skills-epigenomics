---
name: user-input-burden-comparison-across-pipelines
description: Use when benchmarking CIPHER's speed and ease-of-use against other NGS
  analysis pipelines (e.g. HiChIP, MAP-RSeq, bcbio-nextgen) on the same public ChIP-seq
  or RNA-seq FASTQ datasets.
when_to_use_negative:
- The goal is accuracy or result-quality comparison rather than runtime and ease-of-use
  benchmarking
- Datasets mix single-end and pair-end reads in one CONFIG file — CIPHER does not
  handle this use case yet
- The task requires running only a subset of tools rather than an entire workflow
  — CIPHER currently only runs entire workflows
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0102
tools:
- name: CIPHER
  role: Pipeline under test; its six required flags (--mode, --config, --fa, --gtf,
    --lib, --readLen) define the low user-input burden being measured
  repo: https://github.com/c-guzman/cipher-workflow-platform
- name: Nextflow
  role: Workflow engine used to launch CIPHER and execute datasets in parallel, the
    source of CIPHER's speed advantage over serial pipelines
  repo: https://www.nextflow.io/
- name: Singularity
  role: Container runtime (README) used with the -with-singularity flag to containerize
    CIPHER runs for reproducible benchmarking
  repo: http://singularity.lbl.gov/index.html
- name: HiChIP
  role: ChIP-seq comparison pipeline; took ~4X longer (~30 h vs ~8 h) on the same
    6 ChIP-seq samples
- name: MAP-RSeq
  role: RNA-seq comparison pipeline; took ~15X longer (~126 h estimated vs ~8 h) on
    18 RNA-seq samples
- name: bcbio-nextgen
  role: Additional comparison pipeline run on the identical FASTQ inputs to record
    installation/configuration burden
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-017-1770-1
    title: 'CIPHER: a flexible and extensive workflow platform for integrative next-generation
      sequencing data analysis and genomic regulatory element prediction'
  source_content_hash: sha256:c0b733e858a1c3fb2e6a8041ace93dd9fd07aa320321c1265fb34ef9f15b897b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/c-guzman/cipher-workflow-platform
  tools:
  - CIPHER
  - Docker
  - HiChIP
  - MAP-RSeq
  - Nextflow
  - Singularity
  - bcbio-nextgen
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# user-input-burden-comparison-across-pipelines

## Summary

A benchmarking method that quantifies how CIPHER's wall-clock runtime and user-input burden (number of required flags/configuration steps) compare with alternative NGS pipelines such as HiChIP (ChIP-seq) and MAP-RSeq (RNA-seq) run on identical FASTQ datasets.

## When to use

Use when benchmarking CIPHER's speed and ease-of-use against other NGS analysis pipelines (e.g. HiChIP, MAP-RSeq, bcbio-nextgen) on the same public ChIP-seq or RNA-seq FASTQ datasets.

## When NOT to use

- The goal is accuracy or result-quality comparison rather than runtime and ease-of-use benchmarking
- Datasets mix single-end and pair-end reads in one CONFIG file — CIPHER does not handle this use case yet
- The task requires running only a subset of tools rather than an entire workflow — CIPHER currently only runs entire workflows

## Inputs

- Single-ended or pair-ended FASTQ files (e.g. ENCODE ChIP-seq H3K4me1 HCT116, GEO RNA-seq)
- Tab-separated CONFIG file (5 columns single-end, 6 columns pair-end: MergeID, SampleID, Path 1, Path 2, InputID, Mark)
- Ensembl/Gencode reference genome FASTA (--fa/--fasta)
- Ensembl/Gencode GTF annotation (--gtf)
- Gene/LIB reference file (--lib) and read length (--readLen)
- Same FASTQ inputs for comparison pipelines (HiChIP, MAP-RSeq, bcbio-nextgen)

## Outputs

- Benchmark table with one row per pipeline (CIPHER, HiChIP, MAP-RSeq) and populated runtime and input-burden columns
- Total wall-clock runtime per pipeline (e.g. 7 h 23 min for six CIPHER ChIP-seq datasets)
- Count of user-supplied parameters/steps required per pipeline to reach QC reports, BAM files, bigWig tracks, and peak/DGE results

## How to apply

Obtain the same public input data for all pipelines (e.g. ENCODE ChIP-seq H3K4me1 HCT116 FASTQ files, ~30 million reads per dataset, or GEO RNA-seq FASTQs) plus Ensembl/Gencode reference FASTA and GTF files. Run CIPHER via Nextflow with containerization, supplying the six required flags --mode, --config, --fa/--fasta, --gtf, --lib, and --readLen (plus the optional --download_data shortcut for reference files), and run the comparison pipelines (HiChIP, MAP-RSeq, bcbio-nextgen) on the identical FASTQ inputs under equivalent settings, noting any extra installation, configuration, or scripting effort they require. For each pipeline, record total wall-clock runtime and the count of user-supplied parameters/steps needed to reach finished outputs (QC reports, BAM files, bigWig tracks, peak/DGE results). Rationale: CIPHER's parallel Nextflow execution is expected to outpace serial pipelines — the paper reports HiChIP took ~4X longer (~30 h vs ~8 h on 6 ChIP-seq samples) and MAP-RSeq ~15X longer (~126 h estimated vs ~8 h on 18 RNA-seq samples). Compile measurements into a table with one row per pipeline and columns for runtime and per-step execution burden.

## Related tools

- **CIPHER** (Pipeline under test; its six required flags (--mode, --config, --fa, --gtf, --lib, --readLen) define the low user-input burden being measured) — https://github.com/c-guzman/cipher-workflow-platform
- **Nextflow** (Workflow engine used to launch CIPHER and execute datasets in parallel, the source of CIPHER's speed advantage over serial pipelines) — https://www.nextflow.io/
- **Singularity** (Container runtime (README) used with the -with-singularity flag to containerize CIPHER runs for reproducible benchmarking) — http://singularity.lbl.gov/index.html
- **HiChIP** (ChIP-seq comparison pipeline; took ~4X longer (~30 h vs ~8 h) on the same 6 ChIP-seq samples)
- **MAP-RSeq** (RNA-seq comparison pipeline; took ~15X longer (~126 h estimated vs ~8 h) on 18 RNA-seq samples)
- **bcbio-nextgen** (Additional comparison pipeline run on the identical FASTQ inputs to record installation/configuration burden)

## Examples

```
nextflow run cipher.nf -with-singularity cipher.img --mode chip --config CONFIG.txt --fa genome.fa --gtf genes.gtf --lib lib.txt --readLen 50
```

## Evaluation signals

- Recorded CIPHER runtime is consistent with the paper's report that six ChIP-seq datasets (~30 million reads each) completed in 7 h and 23 min
- Benchmark table contains one row for each of the three pipelines (CIPHER, HiChIP, MAP-RSeq) with populated runtime and input-burden fields
- Measured runtime ratios reproduce the reported ~4X (HiChIP vs CIPHER, ChIP-seq) and ~15X (MAP-RSeq vs CIPHER, RNA-seq) differences
- User-input counts reflect CIPHER's six required flags (--mode, --config, --fasta, --gtf, --lib, --readLen) plus the optional --download_data shortcut
- All pipelines were run on identical FASTQ inputs and equivalent reference FASTA/GTF files, so runtime comparisons are apples-to-apples

## Limitations

- CIPHER currently only runs entire workflows; users cannot select a subset of tools, which limits per-step burden comparisons
- CIPHER's RNA-seq pipeline is largely optimized for two-type experimental designs (e.g. WT versus KO), lacking multi-design DEG and time-series analysis
- When running locally, the -qs <INT> flag must control parallelization or the workflow may exhaust memory and abruptly end
- The README uses Singularity containers (sudo singularity create/bootstrap), while containerized runs may alternatively rely on Docker per the paper; container setup requires sudo access

## Evidence

- [results] CIPHER processed all ChIP-seq datasets (~30 million reads per dataset) in 7 h and 23 min: "CIPHER processed all ChIP-seq datasets (~30 million reads per dataset) in 7 h and 23 min"
- [results] parallel processing rationale: [evidence span withheld — anchor status: not_found]
- [readme] README run command: [evidence span withheld — anchor status: external_doc]
- [intro] CIPHER challenges framing: "there are no currently available tools designed to tackle all five challenges simultaneously"
