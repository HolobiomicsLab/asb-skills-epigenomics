---
name: tsa-seq-enrichment-quantification
description: Use when a Hi-C-derived sub-compartment inference (A1/B1/B3) needs quantitative
  validation against independent nuclear-body or lamina enrichment data such as SON
  TSA-seq, lamin-B1 cTSA-seq, or H3K27me3 ChIP-seq.
when_to_use_negative:
- The input is not Hi-C-derived sub-compartment assignments — e.g. the goal is de
  novo inference of sub-compartments rather than benchmarking existing probabilities
  against enrichment tracks
- The genome assembly lacks an hg19 mapping path or the enrichment tracks are unavailable
  for the target genome, since correlations are computed genome-wide on hg19
- Only a normalized bedgraph visualization is needed and no probability scores or
  benchmarking are required
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_3308
tools:
- name: CscoreTool-M
  role: Infers per-window sub-compartment probability scores (XXX_cscore.txt) from
    the Hi-C summary and Rabl bed, run with 5 sub-compartments and Blacklist.bed exclusion;
    these scores are the quantities correlated against SON TSA-seq, H3K27me3 ChIP-seq,
    and lamin-B1 cTSA-seq enrichment.
  repo: https://github.com/scoutzxb/CscoreTool-M
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad314
    title: CscoreTool-M infers 3D sub-compartment probabilities within cell population
  source_content_hash: sha256:dfeb215d8320155fc3c0b8c429e677560353bdcbb2cbc1e0b4ef8aa9c58cc3ad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scoutzxb/CscoreTool-M
  tools:
  - CscoreTool-M
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# tsa-seq-enrichment-quantification

## Summary

Quantifies sub-compartment assignments against independent TSA-seq/cTSA-seq protein-DNA interaction enrichment tracks (SON TSA-seq for A1, lamin-B1 cTSA-seq for B3, plus H3K27me3 ChIP-seq for B1) by computing genome-wide correlation coefficients, using CscoreTool-M compartment probabilities as the method under test.

## When to use

Use when a Hi-C-derived sub-compartment inference (A1/B1/B3) needs quantitative validation against independent nuclear-body or lamina enrichment data such as SON TSA-seq, lamin-B1 cTSA-seq, or H3K27me3 ChIP-seq.

## When NOT to use

- The input is not Hi-C-derived sub-compartment assignments — e.g. the goal is de novo inference of sub-compartments rather than benchmarking existing probabilities against enrichment tracks
- The genome assembly lacks an hg19 mapping path or the enrichment tracks are unavailable for the target genome, since correlations are computed genome-wide on hg19
- Only a normalized bedgraph visualization is needed and no probability scores or benchmarking are required

## Inputs

- Hi-C interaction summary file in HOMER runHiCpca.pl HiCsummary format (e.g. GM12878 Hi-C, GSE63525)
- Equal-length windows_Rabl.bed with Rabl position (relative position between centromere 0 and telomere 1) in column 4
- ENCODE Blacklist.bed (7th argument)
- Published sub-compartment annotations: Rao Gaussian-HMM, SNIPER, SCI, 10 kb Calder
- Enrichment tracks: SON TSA-seq, H3K27me3 ChIP-seq, lamin-B1 cTSA-seq (GEO GSE224282)

## Outputs

- XXX_cscore.txt: per-window sub-compartment probability scores
- XXX_cscore.bedgraph: normalized visualization file (c-scores add up to 1 per region)
- subcompartment_benchmark_correlations.csv: correlation table of A1/B1/B3 vs enrichment benchmarks
- XXX_rabl.txt: estimated Rabl effect matrix

## How to apply

Run CscoreTool-M on a GM12878 Hi-C Rabl-format summary file with 5 sub-compartments (N_Rablwindow=20, N_session=12), passing the ENCODE Blacklist.bed as the 7th argument to exclude blacklist regions possibly affected by copy number variations. Read the per-window probabilities from the output XXX_cscore.txt (compartment scores represent the probability of a genomic region locating in a specific sub-compartment) and map coordinates to hg19 as required. Build binary assignment vectors for each sub-compartment from published annotations of the comparison methods (Rao Gaussian-HMM, SNIPER, SCI, 10 kb Calder). Load the enrichment benchmarks: SON TSA-seq (A1), H3K27me3 ChIP-seq (B1), and lamin-B1 cTSA-seq from GEO GSE224282 (B3). Compute genome-wide correlations of each method's A1/B1/B3 scores against each enrichment track, assemble into a comparison table (subcompartment_benchmark_correlations.csv), and verify CscoreTool-M attains the highest correlation in every comparison.

## Related tools

- **CscoreTool-M** (Infers per-window sub-compartment probability scores (XXX_cscore.txt) from the Hi-C summary and Rabl bed, run with 5 sub-compartments and Blacklist.bed exclusion; these scores are the quantities correlated against SON TSA-seq, H3K27me3 ChIP-seq, and lamin-B1 cTSA-seq enrichment.) — https://github.com/scoutzxb/CscoreTool-M

## Examples

```
./CscoreTool-M hg38_500k_Rabl.bed Test.summary Test_500k_2 5 20 12 hg38_blacklisted.bed
```

## Evaluation signals

- XXX_cscore.bedgraph c-scores are normalized to add up to 1 for each genomic region; regions with no reads are not shown
- The comparison table contains correlations for all three benchmarks: A1 vs SON TSA-seq, B1 vs H3K27me3 ChIP-seq, B3 vs lamin-B1 cTSA-seq, for each method (Gaussian-HMM, SNIPER, SCI, Calder, CscoreTool-M)
- CscoreTool-M attains the highest correlation coefficient in every A1/B1/B3 comparison before the table is saved as subcompartment_benchmark_correlations.csv
- Blacklist regions from ENCODE Blacklist.bed are absent from the analysis (excluded via the 7th argument)
- Per-window scores in XXX_cscore.txt are interpretable as probabilities of a genomic region locating in a specific sub-compartment

## Limitations

- Regions with no reads are not shown in the normalized bedgraph output, so enrichment correlations cover only windows with Hi-C data
- Blacklist regions (ENCODE Blacklist.bed) are excluded, possibly due to copy number variations, which restricts the genomic space of the benchmark
- Coordinates must be mapped to hg19 for the genome-wide correlation step, introducing a potential coordinate-conversion dependency
- Benchmark validity depends on availability of independent enrichment datasets (e.g. GSE224282) and published annotations for each comparison method

## Evidence

- [abstract] The compartment scores inferred using CscoreTool-M represents the probability of a genomic region locating in a specific sub-compartment.: "The compartment scores inferred using CscoreTool-M represents the probability of a genomic region locating in a specific sub-compartment."
- [abstract] Compared to published methods, CscoreTool-M is more accurate in inferring sub-compartments corresponding to both active and repressed chromatin.: "Compared to published methods, CscoreTool-M is more accurate in inferring sub-compartments corresponding to both active and repressed chromatin."
- [methods] Blacklist regions excluded from ChIP-seq analysis, possibly due to copy number variations.: "This is the encode blacklist file which lists the blacklist regions excluded from ChIP-seq analysis, possibly due to copy number variations."
- [methods] Cscore output per genomic window.: "XXX_cscore.txt This is the Cscore estimated for each genomic window."
- [readme] Normalized bedgraph c-scores sum to 1 per region.: "XXX_cscore.bedgraph This is the bedgraph file made for visualization. Ths c-scores are normalized to add up to 1 for each genomic region."
