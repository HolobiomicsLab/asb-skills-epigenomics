---
name: rna-seq-data-processing-execution
description: Use when raw FASTQ RNA-seq data (e.g. WT versus KO designs from GEO)
  must be processed end-to-end into QC reports, BAM files, bigWig tracks, and differential
  gene expression results with a reproducible, low-input pipeline.
when_to_use_negative:
- The design requires multi-group or time-series differential expression — CIPHER's
  RNA-seq pipeline is optimized for typical two-type designs (e.g. WT versus KO) only
- Only a subset of tools (e.g. trimming alone) is needed — CIPHER currently only runs
  entire workflows, not individual tools
- A single CONFIG file contains both single- and pair-ended libraries — the README
  explicitly warns CIPHER does not handle this use-case yet
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CIPHER
  role: The workflow platform whose RNA-seq mode is executed; orchestrates trimming,
    mapping, QC, track generation, and DGE analysis
  repo: https://github.com/c-guzman/cipher-workflow-platform
- name: Nextflow
  role: Workflow language used to launch cipher.nf and manage parallelization across
    samples and cluster executors
- name: Docker
  role: Container-based virtualization used by CIPHER to package software dependencies
    for reproducible execution (article); README alternatively uses Singularity images
- name: MAP-RSeq
  role: Serial RNA-seq comparison pipeline benchmarked against CIPHER (took ~15X longer,
    ~126 h vs ~8 h, on 18 RNA-seq samples)
- name: MultiQC
  role: Aggregates the QC outputs used to validate finished RNA-seq runs
- name: bcbio-nextgen
  role: Alternative pipeline used as a benchmark comparator in runtime/user-input
    comparisons
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-017-1770-1
    title: 'CIPHER: a flexible and extensive workflow platform for integrative next-generation
      sequencing data analysis and genomic regulatory element prediction'
  source_content_hash: sha256:94847181c28001d9098788c490619a23c389ab95d7ca10bd17ac2a70fcde0b85
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/c-guzman/cipher-workflow-platform
  tools:
  - CIPHER
  - Docker
  - MAP-RSeq
  - MultiQC
  - Nextflow
  - bcbio-nextgen
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# rna-seq-data-processing-execution

## Summary

Execute the CIPHER RNA-seq mode of the Nextflow-based NGS workflow platform, which runs raw sequence evaluation, read trimming/adapter removal, mapping and quality filtering, visualization track generation, QC assessment, and differential gene expression analysis on single- or pair-ended RNA-seq datasets with minimal user input, using containerized tools.

## When to use

Use when raw FASTQ RNA-seq data (e.g. WT versus KO designs from GEO) must be processed end-to-end into QC reports, BAM files, bigWig tracks, and differential gene expression results with a reproducible, low-input pipeline.

## When NOT to use

- The design requires multi-group or time-series differential expression — CIPHER's RNA-seq pipeline is optimized for typical two-type designs (e.g. WT versus KO) only
- Only a subset of tools (e.g. trimming alone) is needed — CIPHER currently only runs entire workflows, not individual tools
- A single CONFIG file contains both single- and pair-ended libraries — the README explicitly warns CIPHER does not handle this use-case yet

## Inputs

- Single- or pair-ended RNA-seq FASTQ/FASTQ.GZ files (e.g. from GEO)
- Tab-separated 6-column pair-ended CONFIG file (MergeID, SampleID, Path1, Path2, InputID, Mark)
- Reference genome FASTA (--fa/--fasta)
- Gene annotation GTF (--gtf)
- Library type (--lib) and read length (--readLen) parameters

## Outputs

- Quality control assessment reports (MultiQC-style)
- Mapped BAM files with quality filtering
- bigWig visualization tracks
- Differential gene expression results for two-type (e.g. WT vs KO) designs

## How to apply

Build a tab-separated CONFIG file with 6 columns for pair-ended data (MergeID, SampleID, R1 path, R2 path, InputID, Mark; use '-' for InputID in RNA-seq since no input/control file is needed) and never mix single- and pair-ended rows in one CONFIG. Install only Nextflow and a container runtime (Docker per the article; Singularity per the README), then run `nextflow run cipher.nf` with `--mode rna`, `--config`, plus reference flags `--fa`/`--fasta`, `--gtf`, `--lib`, and `--readLen`; optionally pass `--download_data` to auto-fetch the appropriate Ensembl/Gencode reference files. Run with `-qs <INT>` on a local machine to limit parallelized processes and avoid out-of-memory aborts, or edit `nextflow.config` with a cluster executor (SGE, LSF, SLURM, PBS/Torque) for cluster execution. Rationale: CIPHER parallelizes datasets, which the paper reports made it substantially faster than serial pipelines — MAP-RSeq needed ~15X longer (~126 h vs ~8 h) on 18 RNA-seq samples — while requiring only ~6 mandatory user flags. Outputs include QC reports, BAM alignments, bigWig visualization tracks, and differential gene expression results, verified via MultiQC-style extensive quality control assessment.

## Related tools

- **CIPHER** (The workflow platform whose RNA-seq mode is executed; orchestrates trimming, mapping, QC, track generation, and DGE analysis) — https://github.com/c-guzman/cipher-workflow-platform
- **Nextflow** (Workflow language used to launch cipher.nf and manage parallelization across samples and cluster executors)
- **Docker** (Container-based virtualization used by CIPHER to package software dependencies for reproducible execution (article); README alternatively uses Singularity images)
- **MAP-RSeq** (Serial RNA-seq comparison pipeline benchmarked against CIPHER (took ~15X longer, ~126 h vs ~8 h, on 18 RNA-seq samples))
- **MultiQC** (Aggregates the QC outputs used to validate finished RNA-seq runs)
- **bcbio-nextgen** (Alternative pipeline used as a benchmark comparator in runtime/user-input comparisons)

## Examples

```
nextflow run cipher.nf -with-singularity cipher.img --mode rna --config CONFIG --fa genome.fa --gtf genes.gtf --lib LIB --readLen 100
```

## Evaluation signals

- The run completes with only the required flags supplied (--mode rna, --config, --fasta/--fa, --gtf, --lib, --readLen), confirming low user-input burden
- Finished outputs exist for every sample: QC reports, BAM files, bigWig tracks, and DGE results
- On a comparable hardware/time budget, runtime should be far below serial pipelines such as MAP-RSeq (~126 h for 18 samples vs CIPHER's ~8 h)
- QC assessment flags no biases, contaminations, or errors (extensive quality control assessment per sample)
- CONFIG file follows the 6-column pair-ended schema (or 5-column single-ended) with no mixed single/pair-ended rows, otherwise the run fails

## Limitations

- RNA-seq pipeline is largely optimized for typical two-type experimental designs (e.g. WT versus KO); multi-design DEG and time-series analysis are not supported
- CIPHER currently only runs entire workflows — modular execution of tool subsets is not possible
- Do not mix single- and pair-ended data in the same CONFIG file; the workflow does not handle this case and will fail
- When running locally without a cluster executor, too many parallelized processes (missing -qs flag) cause the workflow to end abruptly from memory exhaustion
- README requires the user to alter example-data CONFIG fastq paths before running, otherwise the run will fail

## Evidence

- [results] CIPHER processed all ChIP-seq datasets (~30 million reads per dataset) in 7 h and 23 min: "CIPHER processed all ChIP-seq datasets (~30 million reads per dataset) in 7 h and 23 min"
- [results] serial comparison runtime: "we used data from the Gene Expression Omnibus repository (GEO) to re-create two previously published studies"
- [intro] parallelization rationale: "By combining Nextflow [11, 12] - a powerful workflow language based on the Unix pipe concept"
- [readme] readme invocation: [evidence span withheld — anchor status: external_doc]
- [readme] readme config warning: [evidence span withheld — anchor status: external_doc]
- [abstract] abstract workflow steps: "(1) raw sequence evaluation, (2) read trimming and adapter removal, (3) read mapping and quality filtering, (4) visualization track generation, and (5) extensive quality control assessment"
