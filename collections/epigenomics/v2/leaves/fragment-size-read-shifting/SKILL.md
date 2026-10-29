---
name: fragment-size-read-shifting
description: Use when calling broad ChIP-seq domains (e.g. H3K27me3/H3K9me3) with
  RECOGNICER from BED-format mapped reads and you must set or sensitivity-test the
  fragment-size shift and window-size parameters.
when_to_use_negative:
- The goal is calling punctate/narrow ChIP-seq peaks rather than broad kb-to-Mb enrichment
  domains (RECOGNICER is designed for broad-domain identification)
- Input reads are already strand-shifted or paired-end fragments with inferred centers,
  so an additional half-fragment 5' shift would double-shift reads
- You need to vary parameters beyond FRAGMENT_SIZE/WINDOW_SIZE such as the block transformation
  factor or termination criteria — the README strongly recommends not changing other
  parameters
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: RECOGNICER
  role: Broad-domain caller whose RECOGNICER.sh configuration exposes WINDOW_SIZE
    (default 200 bp) and FRAGMENT_SIZE (default 150 bp, shift from 5' read end toward
    fragment center); used to run the parameter sweep and produce broadPeak domain
    calls
  repo: github.com/zanglab/recognicer
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s40484-020-0225-2
    title: 'RECOGNICER: A coarse‐graining approach for identifying broad domains from
      ChIP‐seq data'
  source_content_hash: sha256:b470371632c1fab4a4a994ed6279b98db3598b01b15bd121712e7862a7f67252
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - RECOGNICER
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# fragment-size-read-shifting

## Summary

Shift each mapped single-end ChIP-seq read from its 5' end by half the DNA fragment size (RECOGNICER default FRAGMENT_SIZE=150 bp, mononucleosome) within equal-sized non-overlapping windows (default WINDOW_SIZE=200 bp) before broad-domain calling, and verify that resulting domain calls are robust to these two parameters.

## When to use

Use when calling broad ChIP-seq domains (e.g. H3K27me3/H3K9me3) with RECOGNICER from BED-format mapped reads and you must set or sensitivity-test the fragment-size shift and window-size parameters.

## When NOT to use

- The goal is calling punctate/narrow ChIP-seq peaks rather than broad kb-to-Mb enrichment domains (RECOGNICER is designed for broad-domain identification)
- Input reads are already strand-shifted or paired-end fragments with inferred centers, so an additional half-fragment 5' shift would double-shift reads
- You need to vary parameters beyond FRAGMENT_SIZE/WINDOW_SIZE such as the block transformation factor or termination criteria — the README strongly recommends not changing other parameters

## Inputs

- ChIP-sample mapped-read BED file (e.g. human CD4+ T cell H3K27me3 ChIP-seq, Chromosome 1)
- Control-sample mapped-read BED file (RECOGNICER requires ChIP and control BED inputs)
- FRAGMENT_SIZE and WINDOW_SIZE parameter values for the sweep (150 bp/200 bp defaults up to 1 kb window)

## Outputs

- Significantly enriched domain calls per parameter setting ($SAMPLE-fdr0.01_broadPeak.bed in ENCODE broadPeak format)
- Per-setting totals of domain length and ChIP read count within domains, assembled into a parameter-sweep table (e.g. parameter_sweep_table.csv)
- $SAMPLE.cgsummary coarse-graining summary of candidate domains (chrom, start, end, ChIP count, control count, P-value, fold change, Q-value)

## How to apply

Partition each chromosome into equal-sized non-overlapping windows (WINDOW_SIZE, default 200 bp, mononucleosome+linker) and shift each single-end read from its 5' start location toward the center of the DNA fragment it represents, using shift = FRAGMENT_SIZE/2 (default FRAGMENT_SIZE=150 bp for mononucleosomes). To test parameter sensitivity, sweep (FRAGMENT_SIZE, WINDOW_SIZE) combinations from the 150 bp/200 bp defaults up to the 1 kb window size used in the paper, rerunning the full RECOGNICER pipeline (recursive block transformation with L=3 and phase optimization, auto-correlation-based termination, candidate domain retrieval with boundary determination, Poisson significance determination) per setting. Rationale: broad-domain calls should be insensitive to these parameters, so total domain length and total read count within domains serve as the stability check. Run each parameter setting in a separate working directory, since RECOGNICER temporary files have hard-coded names.

## Related tools

- **RECOGNICER** (Broad-domain caller whose RECOGNICER.sh configuration exposes WINDOW_SIZE (default 200 bp) and FRAGMENT_SIZE (default 150 bp, shift from 5' read end toward fragment center); used to run the parameter sweep and produce broadPeak domain calls) — github.com/zanglab/recognicer

## Examples

```
sh RECOGNICER.sh ChIP.bed control.bed 0.01  # with WINDOW_SIZE and FRAGMENT_SIZE set in RECOGNICER.sh (defaults 200 and 150)
```

## Evaluation signals

- The sweep table contains exactly one row per tested (FRAGMENT_SIZE, WINDOW_SIZE) combination
- Total length of identified significant domains remains approximately stable across fragment-size and window-size settings (robustness as in Fig. 3E)
- Total ChIP-seq read count within identified domains remains approximately stable across settings (Fig. 3F)
- Each output run yields the expected RECOGNICER artifacts: $SAMPLE.cgsummary with chrom/start/end/counts/P-value/fold change/Q-value and a broadPeak.bed with fold change in column 5 and ChIP read count as signalValue in column 7
- Parallel runs of different parameter settings were executed in separate directories so hard-coded temporary files did not interfere

## Limitations

- RECOGNICER temporary files have hard-coded names, so multiple instances from the same directory cannot run in parallel without interfering
- RECOGNICER could falsely merge two domains separated by fine-scale gaps into one coarse-scale domain; users must monitor individual merging cases
- The README strongly recommends not changing parameters other than WINDOW_SIZE, FRAGMENT_SIZE, GENOME_FRACTION, and CHIPTHRESHOLD
- The full coarse-graining process may take several hours per run, which constrains sweep size

## Evidence

- [results] the total length of identified domains and total read count in domains do not change appreciably when these parameters are varied: [evidence span withheld — anchor status: not_found]
