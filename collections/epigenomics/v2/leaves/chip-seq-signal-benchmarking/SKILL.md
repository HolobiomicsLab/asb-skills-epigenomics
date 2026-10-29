---
name: chip-seq-signal-benchmarking
description: Use when validating Hi-C-derived sub-compartment probabilities against
  independent enrichment tracks (TSA-seq, ChIP-seq, cTSA-seq) and comparing correlation
  performance against published compartment callers.
when_to_use_negative:
- Input Hi-C interactions are not in HOMER HiCsummary format — reformat or run runHiCpca.pl
  first, as the tool explicitly only accepts that format
- You need per-region visualization rather than benchmarking — use the XXX_cscore.bedgraph
  output instead of computing correlations
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: CscoreTool-M
  role: Infers multiple 3D genome sub-compartment probabilities from Hi-C data; its
    compartment scores are benchmarked against SON TSA-seq, H3K27me3 ChIP-seq, and
    lamin-B1 cTSA-seq correlations
  repo: https://github.com/scoutzxb/CscoreTool-M
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad314
    title: CscoreTool-M infers 3D sub-compartment probabilities within cell population
  source_content_hash: sha256:404370280f5af74314687e2fb8ef1b3c103ff904457256c74a0995b55231a944
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scoutzxb/CscoreTool-M
  tools:
  - CscoreTool-M
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# chip-seq-signal-benchmarking

## Summary

Benchmark inferred 3D sub-compartment scores (A1, B1, B3) from CscoreTool-M against independent protein-DNA interaction datasets (SON TSA-seq for A1, H3K27me3 ChIP-seq for B1, lamin-B1 cTSA-seq for B3) by computing genome-wide correlation coefficients against competing methods (Rao's Gaussian-HMM, SNIPER, SCI, Calder).

## When to use

Use when validating Hi-C-derived sub-compartment probabilities against independent enrichment tracks (TSA-seq, ChIP-seq, cTSA-seq) and comparing correlation performance against published compartment callers.

## When NOT to use

- Input Hi-C interactions are not in HOMER HiCsummary format — reformat or run runHiCpca.pl first, as the tool explicitly only accepts that format
- You need per-region visualization rather than benchmarking — use the XXX_cscore.bedgraph output instead of computing correlations

## Inputs

- GM12878 Hi-C interactions summary file (HOMER runHiCpca.pl HiCsummary format, GSE63525)
- Rabl-format equal-length bed file with relative centromere-telomere position in column 4
- ENCODE blacklist regions bed file
- Published sub-compartment annotations from Rao's Gaussian-HMM, SNIPER, SCI, and 10 kb Calder
- Benchmark enrichment tracks: SON TSA-seq (A1), H3K27me3 ChIP-seq (B1), lamin-B1 cTSA-seq (B3) from GEO GSE224282

## Outputs

- XXX_cscore.txt per-window compartment scores (sub-compartment probabilities)
- Genome-wide correlation coefficients per method per sub-compartment (A1/B1/B3)
- Comparison table (subcompartment_benchmark_correlations.csv)

## How to apply

Run CscoreTool-M on the GM12878 Hi-C summary file (HOMER HiCsummary format from GSE63525) using a Rabl-format bed file with 5 sub-compartments, N_Rablwindow=20, N_session=12, passing the ENCODE Blacklist.bed as the 7th argument to exclude blacklist regions. Read per-window compartment scores from the XXX_cscore.txt output and map coordinates to hg19. Build binary assignment vectors for each sub-compartment from the published annotations of Rao Gaussian-HMM, SNIPER, SCI, and 10 kb Calder. Load the benchmark enrichment tracks (SON TSA-seq, H3K27me3 ChIP-seq, lamin-B1 cTSA-seq; GEO GSE224282) and compute genome-wide correlations of each method's A1/B1/B3 scores against each track. The decision rule from the paper is that CscoreTool-M attains the highest correlation coefficient in every A1/B1/B3 comparison; assemble the results into a comparison table (subcompartment_benchmark_correlations.csv).

## Related tools

- **CscoreTool-M** (Infers multiple 3D genome sub-compartment probabilities from Hi-C data; its compartment scores are benchmarked against SON TSA-seq, H3K27me3 ChIP-seq, and lamin-B1 cTSA-seq correlations) — https://github.com/scoutzxb/CscoreTool-M

## Examples

```
./CscoreTool-M hg38_500k_Rabl.bed Test.summary Test_500k_2 5 20 12 hg38_blacklisted.bed
```

## Evaluation signals

- XXX_cscore.txt exists and contains a compartment score (probability) for each genomic window
- Correlation coefficients are computed genome-wide on hg19 for all three benchmark tracks (A1/SON TSA-seq, B1/H3K27me3 ChIP-seq, B3/lamin-B1 cTSA-seq)
- CscoreTool-M achieves the highest correlation in all A1, B1, and B3 comparisons relative to Gaussian-HMM, SNIPER, SCI, and Calder
- Blacklisted regions (ENCODE Blacklist.bed, 7th argument) are excluded from the analysis before correlation computation
- Results are assembled into a single subcompartment_benchmark_correlations.csv table

## Limitations

- Benchmarking requires mapping output coordinates to hg19 to match the published annotations and enrichment tracks; coordinate mismatch will invalidate correlations
- The executable is compiled on a CentOS linux machine and may not work on other systems; recompile from CscoreTool-M.cpp, twister.h and twister.cpp with g++ -fopenmp -O3 if needed
- Excluding regions via the 8th argument (ExcludedInteractions.txt) also excludes expectations from the calculation, so it is not equivalent to simply deleting interactions from the input file

## Evidence

- [abstract] Compared to published methods, CscoreTool-M is more accurate in inferring sub-compartments corresponding to both active and repressed chromatin.: "Compared to published methods, CscoreTool-M is more accurate in inferring sub-compartments corresponding to both active and repressed chromatin."
- [readme] Blacklist.bed is the encode blacklist file listing regions excluded from ChIP-seq analysis.: "This is the encode blacklist file which lists the blacklist regions excluded from ChIP-seq analysis, possibly due to copy number variations."
- [abstract] The compartment scores inferred using CscoreTool-M represents the probability of a genomic region locating in a specific sub-compartment.: "The compartment scores inferred using CscoreTool-M represents the probability of a genomic region locating in a specific sub-compartment."
- [readme] We accept the same format as the HiCsummary file format for HOMER runHiCpca.pl.: "We accept the same format as the HiCsummary file format for HOMER runHiCpca.pl."
