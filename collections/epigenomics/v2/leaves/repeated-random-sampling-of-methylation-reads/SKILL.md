---
name: repeated-random-sampling-of-methylation-reads
description: Use when comparing phased mCpG/epiallele frequencies across bisulfite-sequenced
  amplicon samples with unequal read depths, where raw frequency comparisons are biased
  by library size.
when_to_use_negative:
- All samples already share the same sequencing depth, so size-comparison bias is
  absent and rarefaction only discards molecules
- The goal is quantitative methylation of individual CpGs or abundance of single epialleles
  (the scope of tools like EBA, ampliMethProfiler and methclone), not phased mCpG
  core/epiallele-family structure
- Input reads have ambiguous CpG calls (gaps or A/G at CpG dinucleotides) that have
  not yet been removed by the ampliMethProfiler quality filters
edam_topics:
- http://edamontology.org/topic_3295
tools:
- name: MethCoresProfiler
  role: R tool whose MethCores_Extractor module computes b4 and performs the y=1000
    iterative sampling before mCpG-combination significance testing
  repo: https://github.com/84AP/MethCoresProfiler/
- name: ampliMethProfiler
  role: Preprocessing tool that extracts per-molecule mCpG configurations and produces
    the binary epihaplotype matrix used as sampling input
  repo: https://sourceforge.net/projects/amplimethprofiler/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqaa096
    title: Tracing and tracking epiallele families in complex DNA populations
  source_content_hash: sha256:7100e8cd5049f7c3c0036ac6445272d98470af6d86a7678916511f6c2ebacc28
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/84AP/MethCoresProfiler
  repo_url: https://github.com/84AP/MethCoresProfiler
  tools:
  - MethCoresProfiler
  - R
  - ampliMethProfiler
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# Repeated random sampling of methylation reads

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reduces read-depth (size-comparison) bias in bisulfite epiallele analysis by iteratively subsampling each experimental sample y=1000 times at the average sample depth b4, as implemented in MethCoresProfiler's MethCores_Extractor before mCpG-combination significance testing.

## When to use

Use when comparing phased mCpG/epiallele frequencies across bisulfite-sequenced amplicon samples with unequal read depths, where raw frequency comparisons are biased by library size.

## When NOT to use

- All samples already share the same sequencing depth, so size-comparison bias is absent and rarefaction only discards molecules
- The goal is quantitative methylation of individual CpGs or abundance of single epialleles (the scope of tools like EBA, ampliMethProfiler and methclone), not phased mCpG core/epiallele-family structure
- Input reads have ambiguous CpG calls (gaps or A/G at CpG dinucleotides) that have not yet been removed by the ampliMethProfiler quality filters

## Inputs

- Tab-delimited text file of epihaplotypes in binary format (one row per DNA molecule, CpG columns as 0/1 methylated states)
- Tab-delimited CpG-position file describing CpG positions in the sequenced string
- Tab-delimited sample metadata file with columns #SampleID, Tissue, Description, Group, Rep, ID

## Outputs

- Size-matched (depth b4) resampled epiallele populations (y=1000 iterations per sample)
- Tables of single-CpG methylation frequencies, tetrachoric correlations and mCpG co-occurrence
- Shannon-Entropy index and summary/sample statistics and plots

## How to apply

Parse the tab-delimited binary epihaplotype matrix (produced e.g. by ampliMethProfiler), the CpG-position file and the sample metadata (#SampleID, Tissue, Description, Group, Rep, ID), then compute the average read depth of each sample (b4) inside MethCores_Extractor. Perform y=1000 repeated iterative samplings of each sample at depth b4 so that downstream frequency statistics are computed on size-matched populations. Feed the resampled populations to Module 1's statistics: chi-square tests of independence on pairs of mCpGs (P-value ≤ 10−9), single-CpG methylation frequencies, tetrachoric correlations, mCpG co-occurrence, and Shannon and Renyi entropies (Renyi weight α = 0.5) normalized to [0, 1]. The rationale is that sampling equalizes molecule counts before significance calls, reducing size-comparison errors between deep and shallow samples. Downstream, Module 2 (MethCores_Combinator) uses the same size-matched frequencies to cross significant two-mCpG combinations into cores (minimum two methylated cytosines).

## Related tools

- **MethCoresProfiler** (R tool whose MethCores_Extractor module computes b4 and performs the y=1000 iterative sampling before mCpG-combination significance testing) — https://github.com/84AP/MethCoresProfiler/
- **ampliMethProfiler** (Preprocessing tool that extracts per-molecule mCpG configurations and produces the binary epihaplotype matrix used as sampling input) — https://sourceforge.net/projects/amplimethprofiler/

## Examples

```
MethCoresProfiler=('/path-to-MethCoresProfiler-master/'); Data='testData/'; input='out/'; source('Run_MethCoresProfiler_setup.R')
```

## Evaluation signals

- Each resampled replicate matches the sample's average depth b4 (same molecule count as the average), confirming size matching
- Sampling is repeated y=1000 times per sample with summarized (averaged) statistics, not a single rarefaction
- Downstream mCpG-pair significance calls use chi-square independence tests with P-value ≤ 10−9 on the size-matched populations
- Reported Shannon and Renyi entropies (α = 0.5) fall within [0, 1] after normalization
- Reconstructed significant mCpG combinations and core frequencies match the original MethCoresProfiler Modules 1–2 reference outputs on the same input files

## Limitations

- Sampling to the average depth b4 discards reads from deeper samples, reducing effective data for those samples
- The 10% amplified epialleles in Cond 3 and Cond 4 escape detection by Modules 1 and 2 even after size-matched sampling; Module 3 is needed to identify them
- Default y = 1000 iterations is computationally heavy given the large R dependency list; results depend on the binary epihaplotype matrix passing upstream filters (≥98% bisulfite efficiency, ≥60% reference alignment, no ambiguous CpG calls)

## Evidence
