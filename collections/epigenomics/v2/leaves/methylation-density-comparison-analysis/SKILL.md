---
name: methylation-density-comparison-analysis
description: Use when you have Msuite methylation-call outputs (CpG/CpH meth.call
  files) from simulated benchmark reads with known preset methylation densities and
  need to quantify call accuracy per cytosine context.
when_to_use_negative:
- Data is BS-seq generated — must use 3-letter mode with '-m BS'; 4-letter mode only
  supports TAPS/5hmC-CATCH data with very low non-CpG methylation
- Real-data samples with unknown ground-truth methylation densities — there is no
  preset density to compare against; use cross-mode agreement (e.g., 4- vs 3-letter
  Pearson R) instead
- You only need alignment output — use '--align-only' rather than running the full
  methylation call pipeline
edam_operation: http://edamontology.org/operation_3207
edam_topics:
- http://edamontology.org/topic_3295
tools:
- name: Msuite
  role: All-in-one toolkit performing quality control, 4-letter alignment, and methylation
    call that produces the CpG/CpH meth.call files analyzed here
  repo: https://github.com/hellosunking/Msuite/
- name: SHERMAN script
  role: Generates the in silico simulated benchmark paired-end reads with preset methylation/conversion
    rates
- name: R
  role: Used to plot preset-vs-deduced methylation density differences and write the
    comparison figure
- name: samtools
  role: Converts Msuite SAM alignment output into BAM format during the pipeline
  repo: http://samtools.sourceforge.net/
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2020.100127
    title: 'Msuite: A High-Performance and Versatile DNA Methylation Data-Analysis
      Toolkit'
  source_content_hash: sha256:b9969cd1cbb6b90ea50bd87d5ed47c00dda3d038e9490a9cfd7a9ff07e11a95b
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
  - Msuite
  - R
  - SHERMAN script
  - samtools
  edam_operation: http://edamontology.org/operation_3207
  edam_topics:
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# methylation-density-comparison-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validate Msuite methylation-call accuracy by parsing Msuite.CpG.meth.call and Msuite.CpH.meth.call outputs, computing deduced methylation densities per context, and comparing them against preset densities (e.g., simulated 20% CpG and 0.5% CpH conversion rates) with R visualization.

## When to use

Use when you have Msuite methylation-call outputs (CpG/CpH meth.call files) from simulated benchmark reads with known preset methylation densities and need to quantify call accuracy per cytosine context.

## When NOT to use

- Data is BS-seq generated — must use 3-letter mode with '-m BS'; 4-letter mode only supports TAPS/5hmC-CATCH data with very low non-CpG methylation
- Real-data samples with unknown ground-truth methylation densities — there is no preset density to compare against; use cross-mode agreement (e.g., 4- vs 3-letter Pearson R) instead
- You only need alignment output — use '--align-only' rather than running the full methylation call pipeline

## Inputs

- Msuite.CpG.meth.call file from a Msuite run
- Msuite.CpH.meth.call file from a Msuite run
- SHERMAN-simulated paired-end TAPS FASTQ files with preset methylation densities (e.g., 20% CpG, 0.5% CpH conversion rates)
- hg19 genome index built with Msuite's build.index.sh (auto-incorporates Lambda genome for conversion-rate estimation)

## Outputs

- Deduced CpG and CpH methylation density values per context
- Per-context difference table of preset vs deduced methylation densities
- methylation_density_comparison.png R plot of preset-vs-deduced differences

## How to apply

Run Msuite on the benchmark FASTQ data (e.g., SHERMAN-simulated paired-end TAPS reads aligned to hg19 in 4-letter mode with '-m TAPS', then 'make' in the output directory) to generate Msuite.CpG.meth.call and Msuite.CpH.meth.call. Parse these call files to compute overall deduced methylation densities for the CpG and CpH contexts separately. Compare each deduced density against the preset value used during simulation (e.g., 20% C->T CpG conversion rate and 0.5% CpH conversion rate), computing per-context differences. Use R to plot preset-vs-deduced differences and write the difference table to a figure such as methylation_density_comparison.png. Accept results when residuals are near zero — at the level of the simulated sequencing error rate (0.1%).

## Related tools

- **Msuite** (All-in-one toolkit performing quality control, 4-letter alignment, and methylation call that produces the CpG/CpH meth.call files analyzed here) — https://github.com/hellosunking/Msuite/
- **SHERMAN script** (Generates the in silico simulated benchmark paired-end reads with preset methylation/conversion rates)
- **R** (Used to plot preset-vs-deduced methylation density differences and write the comparison figure)
- **samtools** (Converts Msuite SAM alignment output into BAM format during the pipeline) — http://samtools.sourceforge.net/

## Examples

```
Msuite -1 read1.fq -2 read2.fq -x hg19 -4 -m TAPS -o out.dir && cd out.dir && make && Rscript plot_meth_density_comparison.R
```

## Evaluation signals

- Deduced-vs-preset density differences are near zero for both CpG and CpH contexts, at the level of the simulated 0.1% sequencing error rate
- Deduced CpG density approximates the preset 20% C->T CpG conversion rate
- Deduced CpH density approximates the preset 0.5% CpH conversion rate
- methylation_density_comparison.png is produced with the accompanying difference table
- Msuite run completed both Msuite.CpG.meth.call and Msuite.CpH.meth.call outputs and the 'make' step succeeded

## Limitations

- No strong evidence that 4-letter mode is more accurate on inconsistent CpG sites; accuracy should be further validated with additional methods such as microarrays
- On real TAPS data with imperfect chemical treatment, densities deviate from ideal — this skill is best grounded on simulated benchmarks with known preset densities
- Msuite methylation call requires reads to meet the minimum alignment score (-Q, default 0) and quality thresholds (-q 20 default, i.e., 1% per-cycle error rate)

## Evidence

- [results] On the simulated benchmark data, the overall methylation densities deduced by Msuite are in close approximation to the preset methylation densities used during simulation.: [evidence span withheld — anchor status: not_found]
