---
name: runtime-and-memory-profiling
description: Use when a command-line motif discovery tool must be benchmarked for
  runtime and peak memory on defined FASTA datasets, and results compared to published
  performance numbers.
when_to_use_negative:
- The goal is accuracy or motif-quality evaluation (e.g. nCC or JASPAR matrix comparison)
  rather than computational tractability benchmarking.
- The tool must be run on a machine far below the paper's desktop-class specification,
  making the profiled numbers incomparable to Table 3.
- The workflow targets motif discovery itself rather than measuring the cost of running
  it.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: DiNAMO
  role: Motif discovery binary whose command-line runs are instrumented to record
    execution time and peak memory (scanning and fixed-position modes)
  repo: https://github.com/bonsai-team/DiNAMO
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-018-2215-1
    title: 'DiNAMO: highly sensitive DNA motif discovery in high-throughput sequencing
      data'
  source_content_hash: sha256:a8d5ce17e37aded4348a2e76a5329d89faacc44ab1ae2ca287ae5005568155a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bonsai-team/DiNAMO
  tools:
  - DiNAMO
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# runtime-and-memory-profiling

## Summary

Profiles the wall-clock execution time and peak resident memory of DiNAMO runs on the evaluation datasets (e.g. GATA1, SOX2, OCT4, STAT3, KLF1 ChIP-seq pairs) and assembles the results into a Table 3-style performance table, benchmarking the motif discovery pipeline's tractability against a stated desktop-machine budget.

## When to use

Use when a command-line motif discovery tool must be benchmarked for runtime and peak memory on defined FASTA datasets, and results compared to published performance numbers.

## When NOT to use

- The goal is accuracy or motif-quality evaluation (e.g. nCC or JASPAR matrix comparison) rather than computational tractability benchmarking.
- The tool must be run on a machine far below the paper's desktop-class specification, making the profiled numbers incomparable to Table 3.
- The workflow targets motif discovery itself rather than measuring the cost of running it.

## Inputs

- Positive-signal FASTA files (e.g. GATA1 with 14,351 sequences; SOX2, OCT4, STAT3, KLF1 datasets)
- Negative-control FASTA files paired with each signal dataset
- Motif length parameter (e.g. -l 6) and mode selection as reported in the paper
- DiNAMO binary from github.com/bonsai-team/DiNAMO releases

## Outputs

- Per-run measurements of wall-clock execution time and peak resident memory
- performance_table.csv reproducing the structure of Table 3 (time and memory space per dataset and mode)

## How to apply

Obtain the evaluation FASTA pairs (positive signal and negative control files, e.g. the GATA1 dataset with 14,351 sequences) from the article's data sources. Invoke the DiNAMO binary in command-line mode with the parameters reported in the paper for the corresponding application (e.g. scanning mode 'dinamo -pf signal.fa -nf control.fa -l 6', or fixed-position mode with the '-p' offset option), and instrument each run with a timing/memory wrapper such as '/usr/bin/time -v' to record wall-clock time and peak resident set size. Run both modes on a desktop-class machine (single Intel Xeon 1.96 GHz CPU class) since the paper claims the tool runs under 8 GB of RAM. Assemble per-dataset measurements into performance_table.csv following the structure of Table 3, which reports e.g. 100 sec and 3400 MB in scanning mode versus 0.75 sec and 41 MB in fixed-position mode on GATA1. Validate that every recorded peak memory value stays below the 8 GB threshold and that measured values are consistent with the published table.

## Related tools

- **DiNAMO** (Motif discovery binary whose command-line runs are instrumented to record execution time and peak memory (scanning and fixed-position modes)) — https://github.com/bonsai-team/DiNAMO

## Examples

```
/usr/bin/time -v dinamo -pf signal.fa -nf control.fa -l 6 -o results.txt
```

## Evaluation signals

- Every recorded peak resident memory value is below the paper's stated desktop threshold '< 8 GB of RAM'.
- Measured scanning-mode GATA1 runtime and memory are in the same order as Table 3 (100 sec, 3400 MB) on comparable hardware.
- Fixed-position mode is dramatically cheaper than scanning mode (paper: 0.75 sec vs 100 sec; 41 MB vs 3400 MB on GATA1).
- performance_table.csv contains one time/memory pair per dataset-and-mode combination matching Table 3's structure.
- Timing wrapper (e.g. /usr/bin/time -v) is used consistently across all runs so wall-clock and peak RSS are comparable.

## Limitations

- Published Table 3 values were obtained on a single Intel Xeon 1.96 GHz CPU; absolute numbers will differ on other hardware, so only order-of-magnitude consistency is expected.
- The article text as indexed does not include Table 3's full contents, so validation for datasets other than GATA1 relies on the external PDF.
- DiNAMO requires ~3.4 GB RAM in scanning mode on the largest dataset, which bounds its use on very small-memory machines.

## Evidence

- [methods] It can be easily installed (binaries for different Linux, MacOS and Windows are available) and used on a desktop machine (< 8 GB of RAM, see Table 3).: "It can be easily installed (binaries for different Linux, MacOS and Windows are available) and used on a desktop machine (< 8 GB of RAM, see Table 3)."
- [results] DiNAMO needing 100 sec and 3400 MB RAM on the GATA1 ChIP-seq dataset in scanning mode, versus only 0.75 sec and 41 MB in fixed-position mode: [evidence span withheld — anchor status: not_found]
- [readme] DiNAMO takes as input two fasta files, which contain respectively the positive dataset (signal.fa) and the negative dataset (control.fa).: "DiNAMO takes as input two fasta files, which contain respectively the positive dataset (signal.fa) and the negative dataset (control.fa)."
- [readme] it has two modes: *scanning* mode (default), where all windows are parsed, or *fixed-position* mode (see optional parameter *-p ): "It has two modes: *scanning* mode (default), where all windows are parsed, or *fixed-position* mode (see optional parameter *-p* )"
