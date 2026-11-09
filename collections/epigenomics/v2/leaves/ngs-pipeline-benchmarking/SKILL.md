---
name: ngs-pipeline-benchmarking
description: Use when comparing pipeline runtime and configuration burden across NGS
  analysis platforms (ChIP-seq/RNA-seq) on the same fastq inputs, or when validating
  claims that a parallelized workflow outperforms serial competitors.
when_to_use_negative:
- When the goal is accuracy or output-quality comparison rather than runtime and configuration
  burden (this benchmark does not validate biological result equivalence)
- When datasets mix single-end and pair-ended reads in one CIPHER CONFIG file — CIPHER
  does not handle this use-case and the run will fail
- When comparing RNA-seq pipelines for complex experimental designs (multi-design
  DEG or time-series), since CIPHER's RNA-seq pipeline is optimized only for two-type
  designs (e.g., WT versus KO)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: CIPHER
  role: Workflow platform under test; run via Nextflow with containerization using
    --mode, --config, --fasta, --gtf, --lib, --readLen flags
  repo: https://github.com/c-guzman/cipher-workflow-platform
- name: Nextflow
  role: Workflow language/engine used to launch CIPHER and provide parallel execution
    of datasets
  repo: https://www.nextflow.io/
- name: Docker
  role: Container technology reported in the article for reproducible CIPHER runs
    during benchmarking
- name: Singularity
  role: 'Alternative containerization per the README: build cipher.img, then run with
    ''nextflow run cipher.nf -with-singularity'''
  repo: http://singularity.lbl.gov/index.html
- name: HiChIP
  role: ChIP-seq comparison pipeline; ran ~4X longer (~30 h vs ~8 h) on the same six
    samples
- name: MAP-RSeq
  role: RNA-seq comparison pipeline; took ~15X longer (~126 h estimated vs ~8 h) on
    18 RNA-seq samples
- name: bcbio-nextgen
  role: Optional additional comparison pipeline run on identical fastq inputs
- name: MultiQC
  role: Aggregates QC output used to confirm all pipelines reached finished output
    state
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-017-1770-1
    title: 'CIPHER: a flexible and extensive workflow platform for integrative next-generation
      sequencing data analysis and genomic regulatory element prediction'
  source_content_hash: sha256:6184e9e5cdd8b4228c0c6e4b42fde238b3a1f345a801639357e0c4f1912892e8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/c-guzman/cipher-workflow-platform
  tools:
  - CIPHER
  - Docker
  - HiChIP
  - MAP-RSeq
  - MultiQC
  - Nextflow
  - Singularity
  - bcbio-nextgen
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# ngs-pipeline-benchmarking

## Summary

Benchmark the wall-clock runtime and user-input burden of an NGS workflow platform (e.g., CIPHER) against competing pipelines such as HiChIP (ChIP-seq) and MAP-RSeq (RNA-seq) on identical public datasets, to quantify speed and ease-of-use differences.

## When to use

Use when comparing pipeline runtime and configuration burden across NGS analysis platforms (ChIP-seq/RNA-seq) on the same fastq inputs, or when validating claims that a parallelized workflow outperforms serial competitors.

## When NOT to use

- When the goal is accuracy or output-quality comparison rather than runtime and configuration burden (this benchmark does not validate biological result equivalence)
- When datasets mix single-end and pair-ended reads in one CIPHER CONFIG file — CIPHER does not handle this use-case and the run will fail
- When comparing RNA-seq pipelines for complex experimental designs (multi-design DEG or time-series), since CIPHER's RNA-seq pipeline is optimized only for two-type designs (e.g., WT versus KO)

## Inputs

- Paired benchmark FASTQ files (e.g., six ~30M-read ENCODE ChIP-seq H3K4me1 HCT116 datasets; GEO RNA-seq fastq files for 18 samples)
- Ensembl/Gencode reference genome FASTA
- Ensembl/Gencode GTF annotation file
- CIPHER tab-separated CONFIG file (5 columns single-end, 6 columns pair-ended: MergeID, SampleID, fastq path(s), InputID, Mark)
- Alternative pipeline installations (HiChIP, MAP-RSeq, bcbio-nextgen)

## Outputs

- Benchmark table with one row per pipeline and columns for total wall-clock runtime and user-input/parameter burden
- Per-pipeline finished outputs used as completion evidence: QC reports, BAM files, bigWig tracks, peak calls (ChIP-seq) or DGE results (RNA-seq)
- Runtime comparison ratios (e.g., ~4X for HiChIP, ~15X for MAP-RSeq vs CIPHER)

## How to apply

Obtain the same public NGS inputs for all pipelines (e.g., six ENCODE ChIP-seq H3K4me1 HCT116 datasets of ~30 million reads each, plus GEO RNA-seq fastq files) and the Ensembl/Gencode reference FASTA and GTF files. Run CIPHER via Nextflow with containerization using only the required flags ('--mode chip' or '--mode rna', '--config', '--fasta'/'--fa', '--gtf', '--lib', '--readLen'), noting the minimal parameter count as the ease-of-use metric. Run the comparison pipelines (HiChIP for ChIP-seq, MAP-RSeq for RNA-seq, optionally bcbio-nextgen) on identical fastq inputs under equivalent settings, recording any extra installation, configuration, or scripting effort. For each pipeline, record total wall-clock runtime and the number of user-supplied parameters/steps needed to reach finished outputs (QC reports, BAM files, bigWig tracks, peak calls or DGE results). Compile measurements into a benchmark table with one row per pipeline and columns for runtime and input burden; speed differences are largely attributable to CIPHER parallelizing datasets while competitors run serially.

## Related tools

- **CIPHER** (Workflow platform under test; run via Nextflow with containerization using --mode, --config, --fasta, --gtf, --lib, --readLen flags) — https://github.com/c-guzman/cipher-workflow-platform
- **Nextflow** (Workflow language/engine used to launch CIPHER and provide parallel execution of datasets) — https://www.nextflow.io/
- **Docker** (Container technology reported in the article for reproducible CIPHER runs during benchmarking)
- **Singularity** (Alternative containerization per the README: build cipher.img, then run with 'nextflow run cipher.nf -with-singularity') — http://singularity.lbl.gov/index.html
- **HiChIP** (ChIP-seq comparison pipeline; ran ~4X longer (~30 h vs ~8 h) on the same six samples)
- **MAP-RSeq** (RNA-seq comparison pipeline; took ~15X longer (~126 h estimated vs ~8 h) on 18 RNA-seq samples)
- **bcbio-nextgen** (Optional additional comparison pipeline run on identical fastq inputs)
- **MultiQC** (Aggregates QC output used to confirm all pipelines reached finished output state)

## Examples

```
nextflow run cipher.nf -with-singularity cipher.img --mode chip --config config.tsv --fa hg38.fa --gtf genes.gtf --lib lib.txt --readLen 50
```

## Evaluation signals

- CIPHER runtime for six ChIP-seq datasets (~30 million reads each) should be ~7 h and 23 min, consistent with the paper's report
- Benchmark table contains exactly one row per pipeline (CIPHER, HiChIP, MAP-RSeq) with populated runtime and input-burden fields
- Expected speedup ratios: HiChIP ~4X slower (~30 h vs ~8 h on 6 ChIP-seq samples) and MAP-RSeq ~15X slower (~126 h estimated vs ~8 h on 18 RNA-seq samples)
- CIPHER run requires only six user-supplied flags (mode, config, fasta/fa, gtf, lib, readLen) plus optional --download_data; any additional required scripting counts against ease-of-use
- All pipelines reach comparable finished outputs: QC reports, BAM files, bigWig tracks, and peak (ChIP-seq) or DGE (RNA-seq) results

## Limitations

- Single-end and pair-ended data must not be mixed in the same CIPHER CONFIG file — CIPHER does not handle this use-case yet
- When running CIPHER without containers, all pipeline software must be installed manually (tools are listed in cipher.nf); on a non-cluster machine, the -qs flag must limit parallel processes or the workflow will end abruptly from memory exhaustion
- The README documents Singularity containers while the article reports Docker; the benchmark environment must match whichever containerization is available
- Timing comparisons depend on parallelization: CIPHER processes datasets in parallel while competitors run serially, so identical hardware/thread counts must be enforced for a fair benchmark

## Evidence

- [full_text] HiChIP took ~4X longer on 6 ChIP-seq samples (~30 h vs ~8 h) and MAP-RSeq took ~15X longer on 18 RNA-seq samples: [evidence span withheld — anchor status: not_found]
- [results] CIPHER processed six ChIP-seq datasets in 7 h 23 min: "CIPHER processed all ChIP-seq datasets (~30 million reads per dataset) in 7 h and 23 min"
- [readme] README run command with flags: [evidence span withheld — anchor status: external_doc]
- [readme] DO NOT MIX AND MATCH SINGLE AND PAIR ENDED DATA INTO THE SAME CONFIG FILE: [evidence span withheld — anchor status: external_doc]
- [full_text] likely because CIPHER processes datasets in parallel while the others run serially: [evidence span withheld — anchor status: not_found]
