---
name: chipseq-fasta-dataset-handling
description: Use when you need to benchmark DiNAMO's execution time and memory footprint
  on the article's ChIP-seq FASTA signal/control dataset pairs (e.g. GATA1) or compare
  its runtime with MEME-CHIP, HOMER and Discrover as in Table 3.
when_to_use_negative:
- When the goal is motif-discovery accuracy (nCC, JASPAR matrix comparison) rather
  than runtime/memory benchmarking — use the synthetic implanted-motif evaluation
  instead
- When the input is not a paired signal/control FASTA dataset but, e.g., a sequencing-error
  coordinate set (the monocyte dataset with 3,272 Hg18 coordinates), which targets
  a different DiNAMO application
- When sequences lack a common fixed motif position — fixed-position mode (-p) is
  only appropriate when motifs occur at a specific offset relative to the sequence
  end
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: DiNAMO
  role: IUPAC motif-discovery software being benchmarked; run in scanning mode (default)
    and fixed-position mode (-p) with '-pf signal.fa -nf control.fa -l 6'
  repo: https://github.com/bonsai-team/DiNAMO
- name: HOMER
  role: Comparison tool for tractability against DiNAMO's execution time and memory
    footprint in Table 3
- name: MEME
  role: EM-based motif discovery tool referenced as a comparison point for the performance
    table
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-018-2215-1
    title: 'DiNAMO: highly sensitive DNA motif discovery in high-throughput sequencing
      data'
  source_content_hash: sha256:078dd57c50e70baa5f01a21eab0cb32b21ee4e1731599ea09aae74eafabfedd3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bonsai-team/DiNAMO
  tools:
  - DiNAMO
  - HOMER
  - MEME
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# chipseq-fasta-dataset-handling

## Summary

Reproduce the DiNAMO benchmark of Table 3 by running the DiNAMO binary (github.com/bonsai-team/DiNAMO) on the ChIP-seq FASTA dataset pairs (GATA1 14,351 sequences; SOX2 4,526; OCT4 3,761; STAT3 2,546; KLF1 945) in both scanning and fixed-position modes, recording wall-clock time and peak resident memory. This validates the paper's claim that DiNAMO runs on a desktop machine with < 8 GB of RAM and quantifies its tractability versus MEME-CHIP, HOMER and Discrover.

## When to use

Use when you need to benchmark DiNAMO's execution time and memory footprint on the article's ChIP-seq FASTA signal/control dataset pairs (e.g. GATA1) or compare its runtime with MEME-CHIP, HOMER and Discrover as in Table 3.

## When NOT to use

- When the goal is motif-discovery accuracy (nCC, JASPAR matrix comparison) rather than runtime/memory benchmarking — use the synthetic implanted-motif evaluation instead
- When the input is not a paired signal/control FASTA dataset but, e.g., a sequencing-error coordinate set (the monocyte dataset with 3,272 Hg18 coordinates), which targets a different DiNAMO application
- When sequences lack a common fixed motif position — fixed-position mode (-p) is only appropriate when motifs occur at a specific offset relative to the sequence end

## Inputs

- Positive-signal FASTA file (e.g. GATA1 ChIP-seq dataset, 14,351 sequences)
- Negative-control FASTA file for each dataset pair
- DiNAMO binary (Linux/OS X/Windows release from github.com/bonsai-team/DiNAMO) or source build requiring Boost
- Motif length parameter (L = 6 in the paper's ChIP-seq application)

## Outputs

- Per-run measurements of execution time and peak resident memory (via /usr/bin/time -v)
- performance_table.csv reproducing the structure of Table 3 (time and memory per dataset and per mode)
- Optional comparison row set against MEME-CHIP, HOMER and Discrover runtimes

## How to apply

Obtain the positive-signal and negative-control FASTA files for each dataset pair (GATA1: 14,351 sequences; SOX2: 4,526; OCT4: 3,761; STAT3: 2,546; KLF1: 945). Install a DiNAMO binary for Linux, OS X or Windows from the github.com/bonsai-team/DiNAMO releases (or build with make against the Boost library). Run the scanning mode (default) as 'dinamo -pf signal.fa -nf control.fa -l 6' for motif length 6, and the fixed-position mode using the optional '-p <i>' parameter that only processes motifs at a given offset relative to the end of each sequence. Instrument each run with '/usr/bin/time -v' (or the time utility) to capture wall-clock time and peak resident memory, expecting roughly the Table 3 profile (e.g. GATA1 scanning mode: ~100 sec and ~3400 MB RAM vs fixed-position mode: ~0.75 sec and ~41 MB on an Intel Xeon 1.96 GHz CPU). Assemble measurements into performance_table.csv mirroring Table 3 and check that every peak-memory value stays below the stated desktop threshold of < 8 GB of RAM.

## Related tools

- **DiNAMO** (IUPAC motif-discovery software being benchmarked; run in scanning mode (default) and fixed-position mode (-p) with '-pf signal.fa -nf control.fa -l 6') — https://github.com/bonsai-team/DiNAMO
- **HOMER** (Comparison tool for tractability against DiNAMO's execution time and memory footprint in Table 3)
- **MEME** (EM-based motif discovery tool referenced as a comparison point for the performance table)

## Examples

```
/usr/bin/time -v dinamo -pf gata1_signal.fa -nf gata1_control.fa -l 6
```

## Evaluation signals

- Every recorded peak resident memory value is below the paper's desktop-machine threshold of < 8 GB of RAM
- Scanning mode on the GATA1 dataset (14,351 sequences, L=6) falls in the order of ~100 sec and ~3400 MB RAM on a single Intel Xeon 1.96 GHz CPU
- Fixed-position mode (-p) runs are orders of magnitude cheaper (~0.75 sec, ~41 MB) than scanning mode on the same dataset
- performance_table.csv contains one time/memory row per dataset per mode, matching the Table 3 structure
- Each run produces a MEME-format output file of over-represented motifs, confirming the run completed rather than was killed (OOM or timeout)

## Limitations

- Table 3's contents are not included in the parsed article text, so expected values must be taken from the finding card (e.g. 100 sec / 3400 MB for GATA1 scanning mode) and validated approximately
- Measured runtimes depend on the host hardware; the paper's reference machine is a single Intel Xeon 1.96 GHz CPU desktop, so absolute times will differ on other machines
- Building DiNAMO from source requires the Boost library (use -I /path/to/boost in the Makefile if not in standard includes); no software version is stated in the article
- Fixed-position mode only processes motifs at a specified offset related to the end of each sequence, so it is not interchangeable with the default scanning mode for benchmarking generality

## Evidence

- [methods] It can be easily installed (binaries for different Linux, MacOS and Windows are available) and used on a desktop machine (< 8 GB of RAM, see Table 3).: "It can be easily installed (binaries for different Linux, MacOS and Windows are available) and used on a desktop machine (< 8 GB of RAM, see Table 3)."
- [discussion] In Table 3, we report the execution time and the memory space required to run DiNAMO.: "In Table 3, we report the execution time and the memory space required to run DiNAMO."
- [readme] dinamo -pf signal.fa -nf control.fa -l 6: "dinamo -pf signal.fa  -nf control.fa -l 6"
- [readme] DiNAMO takes as input two fasta files, which contain respectively the positive dataset (signal.fa) and the negative dataset (control.fa).: "DiNAMO takes as input two fasta files, which contain respectively the positive dataset (signal.fa) and the negative dataset (control.fa)."
- [readme] scanning mode (default), where all windows are parsed, or fixed-position mode (see optional parameter -p): "It has two modes: *scanning* mode (default), where all windows are parsed, or *fixed-position* mode (see optional parameter *-p* )"
