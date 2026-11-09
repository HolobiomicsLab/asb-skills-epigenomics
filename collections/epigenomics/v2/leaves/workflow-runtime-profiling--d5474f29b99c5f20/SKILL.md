---
name: workflow-runtime-profiling
description: Use when comparing the execution time and parameter/setup burden of NGS
  pipelines on the same FASTQ inputs, e.g. validating claims that a parallelized workflow
  (CIPHER) outperforms serial alternatives on ChIP-seq or RNA-seq data.
when_to_use_negative:
- The comparison requires running a subset of individual tools, since CIPHER only
  runs entire workflows rather than modular tool subsets
- The experimental design needs multi-factor or time-series differential expression,
  as CIPHER's RNA-seq pipeline is optimized only for two-type designs (e.g. WT versus
  KO)
- Mixing single-ended and pair-ended data in one run, which CIPHER's CONFIG format
  does not support
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: CIPHER
  role: Workflow platform being benchmarked; runs ChIP-seq/RNA-seq pipelines in parallel
    via Nextflow
  repo: https://github.com/c-guzman/cipher-workflow-platform
- name: Nextflow
  role: Workflow engine used to execute and parallelize CIPHER runs ('nextflow run
    cipher.nf')
  repo: https://www.nextflow.io/
- name: HiChIP
  role: ChIP-seq comparison pipeline whose serial runtime is measured as the baseline
- name: MAP-RSeq
  role: RNA-seq comparison pipeline whose serial runtime is measured as the baseline
- name: bcbio-nextgen
  role: Additional comparison pipeline for runtime/ease-of-use benchmarking
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-017-1770-1
    title: 'CIPHER: a flexible and extensive workflow platform for integrative next-generation
      sequencing data analysis and genomic regulatory element prediction'
  source_content_hash: sha256:483b4f0a87e5e4f10511ab295a4cfe1326f22695cf092c88d7b4cfaf7bceb9be
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
  - bcbio-nextgen
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# workflow-runtime-profiling

## Summary

Benchmark wall-clock runtime and user-input burden of an NGS workflow platform (CIPHER) against competing pipelines (HiChIP for ChIP-seq, MAP-RSeq for RNA-seq) on identical public datasets, to quantify speed and ease-of-use differences.

## When to use

Use when comparing the execution time and parameter/setup burden of NGS pipelines on the same FASTQ inputs, e.g. validating claims that a parallelized workflow (CIPHER) outperforms serial alternatives on ChIP-seq or RNA-seq data.

## When NOT to use

- The comparison requires running a subset of individual tools, since CIPHER only runs entire workflows rather than modular tool subsets
- The experimental design needs multi-factor or time-series differential expression, as CIPHER's RNA-seq pipeline is optimized only for two-type designs (e.g. WT versus KO)
- Mixing single-ended and pair-ended data in one run, which CIPHER's CONFIG format does not support

## Inputs

- Public single-ended FASTQ files (e.g. ENCODE ChIP-seq H3K4me1 HCT116, ~30 million reads per dataset; GEO RNA-seq)
- Tab-separated CIPHER CONFIG file (5 columns single-ended, 6 columns pair-ended: MergeID, SampleID, FASTQ paths, InputID, Mark)
- Ensembl/Gencode reference genome FASTA and GTF annotation files
- Comparison pipeline inputs (same FASTQ files for HiChIP and MAP-RSeq)

## Outputs

- Benchmark table with one row per pipeline (CIPHER, HiChIP, MAP-RSeq) and columns for wall-clock runtime and user-input burden
- Per-pipeline finished outputs for verification (QC reports, BAM files, bigWig visualization tracks, peak calls or DGE results)

## How to apply

Obtain identical public input FASTQ files (e.g. ENCODE ChIP-seq and GEO RNA-seq), plus Ensembl/Gencode reference FASTA and GTF files, for every pipeline being compared. Run CIPHER via Nextflow with containerization, supplying the required flags ('--mode chip'/'--mode rna', '--config', '--fasta', '--gtf', '--lib', '--readLen'); on a single machine cap parallelism with '-qs <INT>' to avoid running out of memory. Run the comparison pipelines (HiChIP, MAP-RSeq, or bcbio-nextgen) on the same FASTQ inputs under equivalent settings, recording any additional installation, configuration, or scripting effort. For each pipeline, record total wall-clock runtime and the number of user-supplied parameters/steps needed to reach finished outputs (QC reports, BAM files, bigWig tracks, peak or differential-expression results). Compile measurements into a benchmark table with one row per pipeline and columns for runtime and per-step execution. Rationale: CIPHER parallelizes dataset processing while HiChIP and MAP-RSeq run serially, explaining the observed ~4X and ~15X speedups.

## Related tools

- **CIPHER** (Workflow platform being benchmarked; runs ChIP-seq/RNA-seq pipelines in parallel via Nextflow) — https://github.com/c-guzman/cipher-workflow-platform
- **Nextflow** (Workflow engine used to execute and parallelize CIPHER runs ('nextflow run cipher.nf')) — https://www.nextflow.io/
- **HiChIP** (ChIP-seq comparison pipeline whose serial runtime is measured as the baseline)
- **MAP-RSeq** (RNA-seq comparison pipeline whose serial runtime is measured as the baseline)
- **bcbio-nextgen** (Additional comparison pipeline for runtime/ease-of-use benchmarking)

## Examples

```
nextflow run cipher.nf -with-singularity cipher.img --mode chip --config CONFIG --fa hg38.fa --gtf genes.gtf --lib LIB --readLen 50 -qs 8
```

## Evaluation signals

- CIPHER runtime should be consistent with the paper's report that six ChIP-seq datasets (~30 million reads each) completed in 7 h and 23 min
- The benchmark table contains one row for each of the three pipelines (CIPHER, HiChIP, MAP-RSeq) with populated runtime and input-burden fields
- HiChIP runtime measures ~4X longer than CIPHER on 6 ChIP-seq samples (~30 h vs ~8 h)
- MAP-RSeq runtime measures ~15X longer than CIPHER on 18 RNA-seq samples (~126 h estimated vs ~8 h)
- CIPHER requires only the six mandatory flags ('--mode', '--config', '--fasta', '--gtf', '--lib', '--readLen') plus an optional '--download_data' shortcut

## Limitations

- Speed advantage depends on parallel execution; when not on a cluster, the '-qs <INT>' flag must limit process count or the workflow may abruptly end because it runs out of memory
- CIPHER does not handle mixing single-ended and pair-ended data in the same CONFIG file
- Runtime comparisons assume equivalent hardware and identical input FASTQ files across pipelines; MAP-RSeq runtime was partly estimated (~126 h)

## Evidence

- [results] CIPHER processed six ChIP-seq datasets in ~8 h versus ~30 h for HiChIP: "CIPHER processed all ChIP-seq datasets (~30 million reads per dataset) in 7 h and 23 min"
- [intro] Parallelization rationale: "By combining Nextflow [11, 12] - a powerful workflow language based on the Unix pipe concept"
- [readme] README command showing flags needed for a run: [evidence span withheld — anchor status: external_doc]
- [readme] README memory caveat for local runs: [evidence span withheld — anchor status: external_doc]
