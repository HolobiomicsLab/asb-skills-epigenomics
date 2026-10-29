---
name: parameter-sensitivity-analysis
description: Use when validating that a method's scientific conclusions do not depend
  on its tunable parameters — e.g., sweeping DNA fragment size and window size in
  RECOGNICER and checking whether total domain length and read counts remain stable.
when_to_use_negative:
- The goal is assessing sensitivity to sequencing depth rather than model parameters
  — the paper handles depth robustness by random downsampling of reads, not by parameter
  sweeps.
- The analysis goal is calling domains with one fixed, recommended configuration rather
  than testing robustness — RECOGNICER defaults (FRAGMENT_SIZE=150, WINDOW_SIZE=200)
  should simply be used.
- The user intends to tune the many internal coarse-graining parameters beyond fragment/window
  size, which the README strongly discourages ('strongly NOT recommended to change
  other parameters').
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3674
tools:
- name: RECOGNICER
  role: Broad domain caller whose FRAGMENT_SIZE and WINDOW_SIZE parameters are swept;
    the full coarse-graining pipeline is rerun per setting and domain length/read-count
    totals are compared
  repo: github.com/zanglab/recognicer
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s40484-020-0225-2
    title: 'RECOGNICER: A coarse‐graining approach for identifying broad domains from
      ChIP‐seq data'
  source_content_hash: sha256:2e6e30102dc845bcdeaeaa558ebc6482be07e1e83c22560847184c33d6f90f6b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - RECOGNICER
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3674
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# parameter-sensitivity-analysis

## Summary

Sweeps tunable model parameters of a computational method (here RECOGNICER's FRAGMENT_SIZE and WINDOW_SIZE for ChIP-seq broad domain calling) and tabulates an output summary metric per setting to demonstrate robustness or sensitivity, as done for human CD4+ T cell H3K27me3 Chromosome 1 data.

## When to use

Use when validating that a method's scientific conclusions do not depend on its tunable parameters — e.g., sweeping DNA fragment size and window size in RECOGNICER and checking whether total domain length and read counts remain stable.

## When NOT to use

- The goal is assessing sensitivity to sequencing depth rather than model parameters — the paper handles depth robustness by random downsampling of reads, not by parameter sweeps.
- The analysis goal is calling domains with one fixed, recommended configuration rather than testing robustness — RECOGNICER defaults (FRAGMENT_SIZE=150, WINDOW_SIZE=200) should simply be used.
- The user intends to tune the many internal coarse-graining parameters beyond fragment/window size, which the README strongly discourages ('strongly NOT recommended to change other parameters').

## Inputs

- ChIP-seq mapped read file in BED format (e.g., human CD4+ T cell H3K27me3, Chromosome 1)
- Control sample mapped read file in BED format
- FDR threshold (README example uses 0.01)
- Sweep grid of FRAGMENT_SIZE and WINDOW_SIZE values (default 150 bp and 200 bp; alternatives up to 1 kb windows)

## Outputs

- parameter_sweep_table.csv with columns: fragment size, window size, total domain length, total read count within domains (one row per parameter combination)
- Per-setting RECOGNICER outputs such as $SAMPLE-fdr0.01_broadPeak.bed and $SAMPLE.cgsummary files

## How to apply

Load the mapped-read BED file (human CD4+ T cell H3K27me3 ChIP-seq, Chromosome 1) and, for each (FRAGMENT_SIZE, WINDOW_SIZE) combination in the sweep — from RECOGNICER's defaults of 150 bp fragment size and 200 bp window size up to the 1 kb window size — partition the chromosome into non-overlapping windows of that size and shift each single-end read from its 5' end by half the fragment size. Run the full RECOGNICER coarse-graining pipeline per setting: recursive block transformation with block transformation factor L=3 and phase optimization, auto-correlation-based termination, candidate domain retrieval with boundary determination, and Poisson-model significance determination. For each run, sum the lengths of all significantly enriched domains and the ChIP-seq read counts within them. Assemble the per-setting totals into a single table (fragment size, window size, total domain length, total read count within domains) and confirm the metrics remain approximately stable across settings. Run instances in separate directories because RECOGNICER's temporary files have hard-coded names.

## Related tools

- **RECOGNICER** (Broad domain caller whose FRAGMENT_SIZE and WINDOW_SIZE parameters are swept; the full coarse-graining pipeline is rerun per setting and domain length/read-count totals are compared) — github.com/zanglab/recognicer

## Examples

```
sh RECOGNICER.sh ChIP.bed control.bed 0.01
```

## Evaluation signals

- parameter_sweep_table.csv contains exactly one row per tested (FRAGMENT_SIZE, WINDOW_SIZE) combination.
- Total length of identified domains is approximately invariant across fragment-size and window-size settings (consistent with Fig. 3E).
- Total read count within domains is approximately invariant across settings (consistent with Fig. 3F).
- Each run used the full pipeline with block transformation factor L=3, phase optimization, and the same FDR threshold, so differences in the table are attributable to the swept parameters only.
- Identified domains carry Poisson-model significance assessment (P-value and Q-value columns in $SAMPLE.cgsummary).

## Limitations

- RECOGNICER may falsely merge two domains separated by fine-scale gaps into one coarse-scale domain, so merged domains in any sweep run should be individually monitored.
- Multiple RECOGNICER instances from the same directory cannot run in parallel because temporary files have hard-coded names; sweep runs must use separate directories.
- Tuning beyond FRAGMENT_SIZE, WINDOW_SIZE, GENOME_FRACTION, and CHIPTHRESHOLD is strongly not recommended by the README, limiting the sweep to these parameters.

## Evidence

- [results] RECOGNICER is robust to the DNA fragment size and Initial window size parameters: the total length of identified domains and total read count in domains do not change appreciably: [evidence span withheld — anchor status: not_found]
