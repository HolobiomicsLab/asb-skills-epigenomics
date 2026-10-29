---
name: desktop-hardware-benchmarking
description: Use when reproducing or updating the execution time and memory footprint
  of a command-line bioinformatics tool (here DiNAMO) on the article's evaluation
  datasets, and when comparing its tractability against alternatives like MEME-CHIP,
  HOMER or Discrover.
when_to_use_negative:
- The goal is motif discovery quality (sensitivity, nCC, predicted cofactors) rather
  than hardware/tractability benchmarking — use the motif-discovery workflow instead
- Only a single small dataset is available and no scanning-mode/fixed-position-mode
  comparison is intended, making a Table 3-style performance table meaningless
- The target machine is not desktop-class (e.g. an HPC node), so the '< 8 GB of RAM'
  desktop-machine claim is not being tested
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: DiNAMO
  role: The benchmarked executable, run in command-line mode in both scanning (default)
    and fixed-position ('-p') modes to measure execution time and memory space
  repo: https://github.com/bonsai-team/DiNAMO
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-018-2215-1
    title: 'DiNAMO: highly sensitive DNA motif discovery in high-throughput sequencing
      data'
  source_content_hash: sha256:40027697376f867d057ef1cea37a28ead113113e4fdcbc8a043bf4881dc2cfbe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bonsai-team/DiNAMO
  tools:
  - DiNAMO
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# desktop-hardware-benchmarking

## Summary

Reproduces the paper's Table 3 by instrumenting DiNAMO runs on the evaluation ChIP-seq datasets (GATA1, SOX2, OCT4, STAT3, KLF1) to record wall-clock execution time and peak resident memory on a desktop-class machine, then validates the measurements against the reported values (e.g. 100 sec / 3400 MB scanning vs 0.75 sec / 41 MB fixed-position on GATA1).

## When to use

Use when reproducing or updating the execution time and memory footprint of a command-line bioinformatics tool (here DiNAMO) on the article's evaluation datasets, and when comparing its tractability against alternatives like MEME-CHIP, HOMER or Discrover.

## When NOT to use

- The goal is motif discovery quality (sensitivity, nCC, predicted cofactors) rather than hardware/tractability benchmarking — use the motif-discovery workflow instead
- Only a single small dataset is available and no scanning-mode/fixed-position-mode comparison is intended, making a Table 3-style performance table meaningless
- The target machine is not desktop-class (e.g. an HPC node), so the '< 8 GB of RAM' desktop-machine claim is not being tested

## Inputs

- Positive signal FASTA files for the evaluation datasets (GATA1: 14,351 sequences; SOX2: 4,526; OCT4: 3,761; STAT3: 2,546; KLF1: 945)
- Matching negative control FASTA files
- DiNAMO binary (Linux/OS X/Windows binaries from the GitHub releases, or built from source with Boost)
- Timing/memory instrumentation (time or /usr/bin/time -v)

## Outputs

- performance_table.csv with per-dataset execution time and peak memory following the structure of Table 3
- Validation check confirming peak memory < 8 GB and consistency with Table 3 values

## How to apply

Obtain the evaluation FASTA pairs (positive signal and negative control files, e.g. the GATA1 dataset with 14,351 sequences) from the article's data sources. Run the DiNAMO binary in command-line mode ('dinamo -pf signal.fa -nf control.fa -l 6') with the parameters reported in the paper for each application — scanning mode is the default, and fixed-position mode is selected with the optional '-p <i>' parameter, which only processes motifs at a given offset from the end of each sequence. Instrument each run with the time utility or '/usr/bin/time -v' on a desktop-class machine (the paper used a single Intel Xeon 1.96 GHz CPU) to capture wall-clock time and peak resident set size. Assemble the per-dataset measurements into performance_table.csv following the structure of Table 3. Validate that every recorded peak memory value stays below the paper's stated '< 8 GB of RAM' desktop-machine threshold and that runtimes/footprints are consistent with Table 3 (GATA1 scanning: 100 sec and 3400 MB; fixed-position: 0.75 sec and 41 MB).

## Related tools

- **DiNAMO** (The benchmarked executable, run in command-line mode in both scanning (default) and fixed-position ('-p') modes to measure execution time and memory space) — https://github.com/bonsai-team/DiNAMO

## Examples

```
/usr/bin/time -v dinamo -pf signal.fa -nf control.fa -l 6
```

## Evaluation signals

- Every recorded peak memory value is below the paper's stated '< 8 GB of RAM' desktop-machine threshold
- GATA1 scanning-mode measurements are in the ballpark of Table 3 (100 sec, 3400 MB RAM)
- GATA1 fixed-position-mode measurements are in the ballpark of Table 3 (0.75 sec, 41 MB RAM), demonstrating the large mode-dependent gap
- performance_table.csv covers every evaluation dataset (GATA1, SOX2, OCT4, STAT3, KLF1) with both time and memory columns populated
- Instrumentation captures both wall-clock execution time and peak resident memory (not just user time) per run

## Limitations

- Table 3's contents are referenced but not fully reproduced in the machine-readable article text, so exact per-dataset values beyond the reported finding may require consulting the published PDF
- The paper's timings were obtained on a single Intel Xeon 1.96 GHz CPU; reproductions on different hardware are expected to deviate in absolute runtime
- No software version for DiNAMO is stated in the article, so version drift between the paper's binary and current GitHub releases may affect measured footprints

## Evidence

- [methods] Desktop threshold claim: "It can be easily installed (binaries for different Linux, MacOS and Windows are available) and used on a desktop machine (< 8 GB of RAM, see Table 3)."
- [discussion] Table 3 purpose: "In Table 3, we report the execution time and the memory space required to run DiNAMO."
- [readme] Input format and CLI: "DiNAMO takes as input two fasta files, which contain respectively the positive dataset (signal.fa) and the negative dataset (control.fa)."
- [readme] Mode distinction: "It has two modes: *scanning* mode (default), where all windows are parsed, or *fixed-position* mode (see optional parameter *-p* )"
