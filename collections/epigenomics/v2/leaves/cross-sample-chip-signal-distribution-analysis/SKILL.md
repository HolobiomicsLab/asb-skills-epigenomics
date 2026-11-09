---
name: cross-sample-chip-signal-distribution-analysis
description: Use when comparing ChIP-seq signal across ≥2 conditions with spike-in
  chromatin (e.g. drug titration series), especially when total-read normalization
  would mask a global occupancy reduction, and peak/background classification (MACS2
  broad-peak BEDs) is available.
when_to_use_negative:
- ChIP-seq experiments lack exogenous spike-in chromatin — the correction factor cannot
  be computed from spike-in material
- The question concerns only a pre-selected subset of loci rather than genome-wide
  signal comparison — spikChIP normalizes over the complete genome, not just selected
  regions
- No peak files (e.g. MACS2 broad-peak BEDs) are available for the experimental or
  spike-in genomes, since bin classification into peaks/background is required for
  the local regression
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0769
tools:
- name: spikChIP
  role: Perl pipeline that orchestrates binning, read counting, multi-approach normalization
    and the LOESS-based spike-in correction across all samples
  repo: https://github.com/eblancoga/spikChIP
- name: SeqCode
  role: Counts reads of BAM files overlapping bins and peaks of BED files during segmentation/counting
  repo: https://github.com/eblancoga/seqcode
- name: Samtools
  role: Reports the number of reads of BAM files and performs downsampling
- name: MACS2
  role: Generates the broad-peak BED files that define peak versus background bins
  repo: https://pypi.org/project/MACS2
- name: gawk
  role: Manages plain-text columnar files used by the pipeline
- name: R (affy, MASS)
  role: Runs the LOESS local regression that gradually applies the spike-in correction
    factor from background to peaks
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nargab/lqab064
    title: 'SpikChIP: a novel computational methodology to compare multiple ChIP-seq
      using spike-in chromatin'
  source_content_hash: sha256:3e2ade307b37da33f182e2ade7f6e11695ac42e137b5ae36cea936fb5c8e7ab3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/eblancoga/spikChIP
  repo_url: https://github.com/eblancoga/spikChIP
  tools:
  - MACS2
  - R
  - R (affy, MASS)
  - Samtools
  - SeqCode
  - gawk
  - spikChIP
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# cross-sample-chip-signal-distribution-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Genome-wide comparison of ChIP-seq signal distributions across multiple experimental conditions using spikChIP's local regression spike-in normalization, which increasingly applies the spike-in correction factor from background to peak regions so that global occupancy changes (e.g. gradual H3K79me2 reduction across 0%–100% EPZ5676 mixtures) become detectable.

## When to use

Use when comparing ChIP-seq signal across ≥2 conditions with spike-in chromatin (e.g. drug titration series), especially when total-read normalization would mask a global occupancy reduction, and peak/background classification (MACS2 broad-peak BEDs) is available.

## When NOT to use

- ChIP-seq experiments lack exogenous spike-in chromatin — the correction factor cannot be computed from spike-in material
- The question concerns only a pre-selected subset of loci rather than genome-wide signal comparison — spikChIP normalizes over the complete genome, not just selected regions
- No peak files (e.g. MACS2 broad-peak BEDs) are available for the experimental or spike-in genomes, since bin classification into peaks/background is required for the local regression

## Inputs

- Configuration file (plain text: #sample, bam_sample, bam_spike, peaks_sample, peaks_spike per condition)
- ChromInfo file listing chromosomes and sizes of sample and spike-in genomes (e.g. hg19 + dm3 with FLY suffix)
- Mapped BAM files for sample and spike-in genomes per condition (e.g. GSM1465004-GSM1465008 from GSE60104)
- MACS2 broad-peak BED files for both sample and spike-in genomes per condition
- Bin size in bp (e.g. -k 1000 for 1000-bp bins)

## Outputs

- Normalized bin-level signal files per approach (RAW, TRADITIONAL, CHIPRX, TAG REMOVAL, SPIKCHIP), e.g. FINAL_<series>_SPIKCHIP_1000_avg_normalized_sample.txt.gz in results/
- Automatic PDF boxplots in plots/ (with and without labels) comparing normalized distributions for peaks versus background in sample and spike-in genomes across conditions
- Run log (e.g. logs/<series>.log)
- Intermediate files (removable with -c)

## How to apply

Build a plain-text configuration file listing, per condition, the sample and spike-in BAM files plus user-provided MACS2 broad-peak BED files, and a ChromInfo file enumerating chromosomes of both genomes (e.g. hg19 plus dm3 chromosomes with a 'FLY' suffix). Run spikChIP with a bin size such as 1000 bp (-k), counting reads per bin (average and maximum) with SeqCode and Samtools, and classify bins as peaks or background from the supplied BEDs. The program computes RAW, TRADITIONAL, CHIPRX and TAG REMOVAL normalizations in addition to SPIKCHIP, the latter applying a LOESS local regression (R with affy and MASS libraries) that gradually applies the spike-in correction factor from background to positive ChIP signal regions. Inspect the normalized result files (e.g. FINAL_*_SPIKCHIP_1000_avg_normalized_sample.txt.gz) and the automatic PDF boxplots in plots/ comparing peak versus background distributions across all conditions, with logging to logs/. Judge success when peak-region normalized signal declines progressively across the treatment series while background distributions remain flat, and a control histone mark (e.g. H3K4me3) shows equivalent peak and background levels across samples.

## Related tools

- **spikChIP** (Perl pipeline that orchestrates binning, read counting, multi-approach normalization and the LOESS-based spike-in correction across all samples) — https://github.com/eblancoga/spikChIP
- **SeqCode** (Counts reads of BAM files overlapping bins and peaks of BED files during segmentation/counting) — https://github.com/eblancoga/seqcode
- **Samtools** (Reports the number of reads of BAM files and performs downsampling)
- **MACS2** (Generates the broad-peak BED files that define peak versus background bins) — https://pypi.org/project/MACS2
- **gawk** (Manages plain-text columnar files used by the pipeline)
- **R (affy, MASS)** (Runs the LOESS local regression that gradually applies the spike-in correction factor from background to peaks)

## Examples

```
./spikChIP -vcp 1 -e FLY -k 1 config/H3K79me2_0-25-50-75-100_config.txt ChromInfo.txt
```

## Evaluation signals

- Normalized H3K79me2 peak-region signal declines progressively from 0% to 100% EPZ5676 across the five Jurkat samples, including the 25:75, 50:50 and 75:25 mixtures
- Background-region normalized distributions remain flat across all conditions in the boxplots (no overcorrection of non-occupied regions)
- A control mark such as H3K4me3 shows equivalent peak and background signal levels across all samples (no spurious changes)
- The spike-in genome distributions confirm the expected correction behavior (local regression applied gradually from background to peaks)
- The FINAL_<series>_SPIKCHIP_1000_avg_normalized_sample.txt.gz output exists in results/ and the log in logs/ reports successful processing of all five conditions

## Limitations

- Normalization relies on user-supplied peak caller output (e.g. MACS2 broad peaks); poor peak calls propagate into the peaks/background classification
- The number of spike-in reads in the final sequencing step should be at least one million reads (~2%–5% of the experimental genome) to minimize changes in overall material used
- Cross-reactivity of the experimental antibody when capturing spike-in material is a constraint; a second antibody against a fly-specific histone variant (H2Av) is recommended
- Alternative spike-in normalization strategies (RAW, TRADITIONAL, CHIPRX, TAG REMOVAL) can introduce inconsistencies in final results, as each has its own benefits and limitations

## Evidence
