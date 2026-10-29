---
name: benchmark-table-compilation
description: Use when a study reports execution time and memory footprint of a tool
  on named datasets and the task is to reproduce or extend that performance table
  on a local machine.
when_to_use_negative:
- The task is to evaluate motif discovery quality (e.g. nCC or JASPAR comparison)
  rather than runtime and memory tractability
- The hardware is not desktop-class or the comparison target values were measured
  on a different architecture, making direct runtime comparison meaningless without
  re-baselining
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: DiNAMO
  role: Tool being benchmarked; invoked in command-line mode in scanning (default)
    and fixed-position (-p) modes on signal/control FASTA pairs
  repo: https://github.com/bonsai-team/DiNAMO
- name: MEME-CHIP
  role: Comparison tool whose tractability is contrasted with DiNAMO in Table 3
- name: HOMER
  role: Comparison tool whose tractability is contrasted with DiNAMO in Table 3
- name: Discrover
  role: Comparison tool whose tractability is contrasted with DiNAMO in Table 3
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-018-2215-1
    title: 'DiNAMO: highly sensitive DNA motif discovery in high-throughput sequencing
      data'
  source_content_hash: sha256:1fa083455033d288e0de987f60007f98441800fef537245c0904f2d85587e3c8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bonsai-team/DiNAMO
  tools:
  - DiNAMO
  - Discrover
  - HOMER
  - MEME-CHIP
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# benchmark-table-compilation

## Summary

Reproduce and assemble a performance benchmark table (like Table 3 of the DiNAMO paper) by instrumenting tool runs to record wall-clock execution time and peak resident memory per dataset, then validating measurements against the published values.

## When to use

Use when a study reports execution time and memory footprint of a tool on named datasets and the task is to reproduce or extend that performance table on a local machine.

## When NOT to use

- The task is to evaluate motif discovery quality (e.g. nCC or JASPAR comparison) rather than runtime and memory tractability
- The hardware is not desktop-class or the comparison target values were measured on a different architecture, making direct runtime comparison meaningless without re-baselining

## Inputs

- Signal and negative-control FASTA files for each evaluated dataset (GATA1 14,351 sequences; SOX2 4526; OCT4 3761; STAT3 2546; KLF1 945)
- Published benchmark table values to compare against (DiNAMO Table 3: execution time and memory space)
- Tool binary or source build (DiNAMO binaries for Linux/OS X/Windows from GitHub releases, or build with `make` requiring Boost)

## Outputs

- performance_table.csv with per-dataset execution time and peak memory, structured like Table 3 of the article
- Per-run instrumentation logs (wall-clock time and peak resident memory from /usr/bin/time -v)

## How to apply

Obtain the exact evaluation FASTA datasets cited in the table (e.g. the GATA1 ChIP-seq dataset with 14,351 sequences, plus SOX2 (4526), OCT4 (3761), STAT3 (2546) and KLF1 (945) datasets). Run the tool once per dataset pair and per reported mode using the paper's command-line parameters — for DiNAMO, `dinamo -pf signal.fa -nf control.fa -l 6` in default scanning mode, and with the `-p` offset parameter for fixed-position mode. Instrument each run with `/usr/bin/time -v` (or the `time` utility) to capture wall-clock time and peak resident set size on a single desktop-class CPU (the paper used one Intel Xeon 1.96 GHz core). Assemble the per-dataset measurements into performance_table.csv mirroring the structure of Table 3, distinguishing the two DiNAMO modes: the paper reports 100 sec and 3400 MB RAM for GATA1 in scanning mode versus only 0.75 sec and 41 MB in fixed-position mode. Finally validate that every peak memory value stays below the paper's stated desktop threshold ('< 8 GB of RAM') and that measured runtimes are in the same order of magnitude as the published values.

## Related tools

- **DiNAMO** (Tool being benchmarked; invoked in command-line mode in scanning (default) and fixed-position (-p) modes on signal/control FASTA pairs) — https://github.com/bonsai-team/DiNAMO
- **MEME-CHIP** (Comparison tool whose tractability is contrasted with DiNAMO in Table 3)
- **HOMER** (Comparison tool whose tractability is contrasted with DiNAMO in Table 3)
- **Discrover** (Comparison tool whose tractability is contrasted with DiNAMO in Table 3)

## Examples

```
/usr/bin/time -v dinamo -pf GATA1_signal.fa -nf GATA1_control.fa -l 6 -o dinamo_gata1_scanning.txt
```

## Evaluation signals

- Every recorded peak resident memory value is below the paper's stated desktop threshold of 8 GB RAM
- GATA1 scanning-mode measurements are in the same order of magnitude as 100 sec and 3400 MB on a comparable single-core CPU
- Fixed-position-mode measurements show the reported orders-of-magnitude speedup and memory reduction (0.75 sec, 41 MB) relative to scanning mode
- performance_table.csv contains a row for every dataset and mode combination reported in Table 3 with no missing cells
- Run configuration (motif length -l 6, same FASTA inputs, single CPU) matches the parameters reported in the paper

## Limitations

- Table 3's contents are not included in the article text extract, so exact values for all datasets may need to be recovered from the published PDF
- Published timings were obtained on a single Intel Xeon 1.96 GHz CPU; runtimes are not directly transferable to different hardware
- Scanning mode memory (3400 MB for GATA1) approaches desktop limits for larger datasets, so scaling beyond the evaluated datasets may exceed the 8 GB threshold

## Evidence

- [methods] It can be easily installed (binaries for different Linux, MacOS and Windows are available) and used on a desktop machine (< 8 GB of RAM, see Table 3).: "It can be easily installed (binaries for different Linux, MacOS and Windows are available) and used on a desktop machine (< 8 GB of RAM, see Table 3)."
- [discussion] In Table 3, we report the execution time and the memory space required to run DiNAMO.: "In Table 3, we report the execution time and the memory space required to run DiNAMO."
- [readme] dinamo -pf signal.fa  -nf control.fa -l 6: "dinamo -pf signal.fa  -nf control.fa -l 6"
- [readme] Only process motifs at a offset i (0..N) related to the end of each sequence: "Only process motifs at a offset i (0..N)  related to the **end** of each sequence"
- [methods] DiNAMO is implemented in C ++ using the libraries Sparsepp [26] and boost [27]. It is freely available under the GNU Affero General Public License, version 3.: "DiNAMO is implemented in C ++ using the libraries Sparsepp [26] and boost [27]. It is freely available under the GNU Affero General Public License, version 3."
