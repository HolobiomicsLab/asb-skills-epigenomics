---
name: computational-resource-profiling
description: Use when comparing the speed, memory footprint, mapping efficiency, or
  accuracy of methylation read-alignment tools (e.g., Msuite vs. Bismark vs. BWA-meth)
  on a controlled read set.
when_to_use_negative:
- The goal is methylation calling or biological interpretation rather than comparing
  tool performance — this skill only profiles computational metrics and mapping statistics.
- Input is real clinical/experimental data where ground-truth mapping accuracy cannot
  be computed; simulated reads with known origins are needed for the accuracy column.
- 'The aligner is not methylation-aware (plain BWA/STAR): the TAPS/BS conversion modes
  (-3/-4, -m TAPS/BS) and the comparison framing do not apply.'
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
tools:
- name: Msuite
  role: Tool under test; run in 3-letter (-3 -m TAPS) and 4-letter (-4 -m TAPS) modes
    after building indices with build.index.sh
  repo: https://github.com/hellosunking/Msuite/
- name: SHERMAN script
  role: Generates the in-silico paired-/single-end 36-/100-bp TAPS-protocol simulation
    reads used as the controlled benchmark input
- name: Bismark
  role: Baseline 3-letter bisulfite aligner (v0.22.3) run with default settings for
    the speed/memory comparison
- name: BWA-meth
  role: Baseline 3-letter aligner (v0.2.2) run with default settings for the speed/memory
    comparison
- name: samtools
  role: Supporting utility for handling SAM/BAM alignment outputs during benchmarking
  repo: http://samtools.sourceforge.net/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2020.100127
    title: 'Msuite: A High-Performance and Versatile DNA Methylation Data-Analysis
      Toolkit'
  source_content_hash: sha256:0b3ff4242b987f822577cb3d71aab314da4f305a926b1456a5a4e6fcf264ceac
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/hellosunking/Msuite
  repo_url: https://github.com/hellosunking/Msuite
  tools:
  - BWA-meth
  - Bismark
  - Msuite
  - SHERMAN script
  - samtools
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# computational-resource-profiling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarking skill for comparing alignment/tool performance on identical in-silico datasets: it measures mapping efficiency, mapping accuracy, total running time, and peak memory usage per tool/mode (e.g., Msuite 3- and 4-letter modes vs. Bismark v0.22.3 and BWA-meth v0.2.2 on simulated TAPS reads).

## When to use

Use when comparing the speed, memory footprint, mapping efficiency, or accuracy of methylation read-alignment tools (e.g., Msuite vs. Bismark vs. BWA-meth) on a controlled read set.

## When NOT to use

- The goal is methylation calling or biological interpretation rather than comparing tool performance — this skill only profiles computational metrics and mapping statistics.
- Input is real clinical/experimental data where ground-truth mapping accuracy cannot be computed; simulated reads with known origins are needed for the accuracy column.
- The aligner is not methylation-aware (plain BWA/STAR): the TAPS/BS conversion modes (-3/-4, -m TAPS/BS) and the comparison framing do not apply.

## Inputs

- in-silico paired-end 100-bp and 36-bp TAPS-protocol FASTQ read pairs (1 million read pairs each) simulated with SHERMAN
- reference genome FASTA (e.g., GRCh38) and RefSeq annotation for index building
- tool installations: Msuite, Bismark v0.22.3, BWA-meth v0.2.2, bowtie2, samtools

## Outputs

- aggregated benchmark table (table2_benchmark.tsv) with one row per tool/mode and columns for read length, mapping efficiency (%), accuracy (%), running time, and peak memory
- per-run profiling records from /usr/bin/time (wall time and peak memory)

## How to apply

Generate identical in-silico test reads (e.g., 1 million paired-end 100-bp and 36-bp TAPS-protocol read pairs with SHERMAN, using 20% C->T conversion at CpG sites, 0.5% at CpH sites, and 0.1% sequencing error rate, restricted to CT/GA-rich regions with CT or GA proportion >=80%) so the comparison is controlled. Build genome indices once (Msuite's build.index.sh with reference FASTA and RefSeq annotation) and run each tool on the same read sets under default settings, including Msuite 3-letter (-3 -m TAPS) and 4-letter (-4 -m TAPS) modes by launching the generated makefile. Include overhead steps in the timing: Msuite's adapter/quality trimming is counted, so its measured time is comparable end-to-end. Profile time and memory with standard system tools such as /usr/bin/time, then aggregate mapping efficiency (%), accuracy (%), running time, and peak memory per tool/mode into a table (table2_benchmark.tsv) and validate against expected reference values (Msuite 3-letter 96.19%, Msuite 4-letter 95.75%, Bismark 95.79%, BWA-meth 95.58% mapping efficiency on 100-bp reads).

## Related tools

- **Msuite** (Tool under test; run in 3-letter (-3 -m TAPS) and 4-letter (-4 -m TAPS) modes after building indices with build.index.sh) — https://github.com/hellosunking/Msuite/
- **SHERMAN script** (Generates the in-silico paired-/single-end 36-/100-bp TAPS-protocol simulation reads used as the controlled benchmark input)
- **Bismark** (Baseline 3-letter bisulfite aligner (v0.22.3) run with default settings for the speed/memory comparison)
- **BWA-meth** (Baseline 3-letter aligner (v0.2.2) run with default settings for the speed/memory comparison)
- **samtools** (Supporting utility for handling SAM/BAM alignment outputs during benchmarking) — http://samtools.sourceforge.net/

## Examples

```
/usr/bin/time -v msuite -x hg38 -1 reads_R1.fq -2 reads_R2.fq -3 -m TAPS -o msuite_3l_out
```

## Evaluation signals

- Mapping-efficiency values reproduce the reported Table 2 numbers (Msuite 3-letter 96.19%, Msuite 4-letter 95.75%, Bismark 95.79%, BWA-meth 95.58% on 100-bp reads) within rounding tolerance
- Msuite shows faster total running time than Bismark and BWA-meth even though its adapter/quality-trimming time is included in the measurement
- Msuite's peak memory usage (from /usr/bin/time) is lower than both baselines
- table2_benchmark.tsv has exactly one row per tool/mode with complete read-length, efficiency, accuracy, time, and memory columns
- All tools are run on the same simulated read sets (1M pairs, 100-bp and 36-bp, 20%/0.5% conversion, 0.1% error) so differences are attributable to the aligners

## Limitations

- Simulation-based benchmarking does not capture real-chemistry artifacts: on a real TAPS dataset with imperfect chemical treatment, results may differ even though the 4-letter mode still showed high-quality results
- Higher benchmark mapping accuracy does not prove superiority everywhere: there is no strong evidence that the 4-letter mode is more accurate on inconsistent CpG sites
- Comparison is protocol-specific (TAPS, CT/GA-rich regions); BS-seq data must use 3-letter mode and -m BS, so the 4-letter benchmark does not transfer to bisulfite libraries

## Evidence
