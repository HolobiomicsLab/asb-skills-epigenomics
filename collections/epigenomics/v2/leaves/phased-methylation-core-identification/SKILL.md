---
name: phased-methylation-core-identification
description: Use when bisulfite amplicon reads have been converted to a binary epihaplotype
  matrix and the goal is to identify statistically significant phased mCpG combinations
  (cores) and track epiallele families across populations.
when_to_use_negative:
- The analysis goal is quantitative methylation of single CpGs or abundance of individual
  epialleles without phased-CpG structure — tools like EBA, ampliMethProfiler or methclone
  address this and cannot distinguish cores (e.g. they fail to separate Cond 4 from
  R1/R2 epialleles).
- The input is raw FASTQ/FASTA rather than a processed binary epihaplotype matrix
  — preprocessing (PEAR merge, PRINSEQ conversion, ampliMethProfiler mCpG extraction)
  must be completed first.
- The DNA segment contains fewer than two CpGs, since the minimum core requires at
  least two methylated cytosines with statistically significant frequency.
edam_operation: http://edamontology.org/operation_3208
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0654
tools:
- name: MethCoresProfiler
  role: R-based tool that extracts and identifies combinations of methylated phased
    CpGs (methylated cores), assigning clonality and stability/entanglement indices
    to each CpG in the core
  repo: https://github.com/84AP/MethCoresProfiler/
- name: ampliMethProfiler
  role: Upstream processing of FASTA bisulfite reads to extract mCpG configurations
    in single DNA molecules, producing the binary epihaplotype matrix input
  repo: https://sourceforge.net/projects/amplimethprofiler/
- name: PEAR
  role: Merges paired-end FASTQ reads (minimum 40 nucleotides overlap) before methylation
    extraction
- name: PRINSEQ
  role: Converts quality-filtered reads to FASTA format for ampliMethProfiler
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqaa096
    title: Tracing and tracking epiallele families in complex DNA populations
  source_content_hash: sha256:67aaa5988e0e7ffda0368b1ee16293e48067dfc93fd3da1f7267f5898147b5e1
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
  - PEAR
  - PRINSEQ
  - R
  - ampliMethProfiler
  edam_operation: http://edamontology.org/operation_3208
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 22
---

# phased-methylation-core-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts stable combinations of phased methylated CpGs (methylated cores) shared by epiallele families from bisulfite-sequencing epihaplotypes using MethCoresProfiler, and quantifies each core with clonality and stability/entanglement indices in each population.

## When to use

Use when bisulfite amplicon reads have been converted to a binary epihaplotype matrix and the goal is to identify statistically significant phased mCpG combinations (cores) and track epiallele families across populations.

## When NOT to use

- The analysis goal is quantitative methylation of single CpGs or abundance of individual epialleles without phased-CpG structure — tools like EBA, ampliMethProfiler or methclone address this and cannot distinguish cores (e.g. they fail to separate Cond 4 from R1/R2 epialleles).
- The input is raw FASTQ/FASTA rather than a processed binary epihaplotype matrix — preprocessing (PEAR merge, PRINSEQ conversion, ampliMethProfiler mCpG extraction) must be completed first.
- The DNA segment contains fewer than two CpGs, since the minimum core requires at least two methylated cytosines with statistically significant frequency.

## Inputs

- Tab-delimited binary EpiHaplotype matrix (file.out) produced by ampliMethProfiler
- Tab-delimited CpG-position file (positions of CpGs in the sequence/string)
- Sample metadata file with columns #SampleID, Tissue, Description, Group, Rep, ID
- List of regions for MethCores_Combinator and MethCores_Analyst

## Outputs

- Tables of single-CpG methylation frequencies, tetrachoric correlations, and two-mCpG co-occurrence
- Shannon-entropy index and summary/sample statistics and plots
- ComposCore table (structure and frequency of significant mCpG cores)
- Tab_Epialleles table (extracted epialleles containing all significant mCpG combinations)
- Heatmaps of epiallele structure and frequency plus PCA of significant epialleles
- MethCores Index, Clonality Index, and Stability/Entanglement Index (E) per core per population

## How to apply

Process reads with ampliMethProfiler (after PEAR merging and PRINSEQ FASTA conversion) to obtain a tab-delimited binary EpiHaplotype matrix, a CpG-position file, and a sample metadata file (#SampleID, Tissue, Description, Group, Rep, ID). Run MethCores_Extractor (Module 1) to compute average depth b4, perform y = 1000 iterative samplings at depth b4, and annotate significant two-mCpG combinations via chi-square test of independence at P-value ≤ 10−9, also reporting single-CpG frequencies, tetrachoric correlations, mCpG co-occurrence, and Shannon/Renyi entropies (α = 0.5, normalized to [0,1]). Then run MethCores_Combinator (Module 2), which crosses significant two-mCpG combinations, tests combinations of three or more mCpGs against expected independent-event frequencies (chi-square, P ≤ 10−9), extracts the epialleles containing all significant combinations, and reports core structures and frequencies (ComposCore and Tab_Epialleles), enforcing a minimum core of at least two methylated cytosines. MethCores_Analyst (Module 3) compares each epiallele's frequency to a random control (R) with the same number of CpGs and depth, performs hierarchical clustering/PCA, and derives three indices: MethCores Index (core frequency in the population), Clonality Index (core frequency normalized to average population methylation), and the Stability/Entanglement Index E (rate of coupling of at least two mCpGs in the core). The rationale is that cores are stable signatures marking families of epialleles from common ancestors, with high information content and low stochasticity.

## Related tools

- **MethCoresProfiler** (R-based tool that extracts and identifies combinations of methylated phased CpGs (methylated cores), assigning clonality and stability/entanglement indices to each CpG in the core) — https://github.com/84AP/MethCoresProfiler/
- **ampliMethProfiler** (Upstream processing of FASTA bisulfite reads to extract mCpG configurations in single DNA molecules, producing the binary epihaplotype matrix input) — https://sourceforge.net/projects/amplimethprofiler/
- **PEAR** (Merges paired-end FASTQ reads (minimum 40 nucleotides overlap) before methylation extraction)
- **PRINSEQ** (Converts quality-filtered reads to FASTA format for ampliMethProfiler)

## Examples

```
Rscript Run_MethCoresProfiler_setup.R  # after setting MethCoresProfiler="/path-to-MethCoresProfiler-master/", Data="testData/", input="out/"
```

## Evaluation signals

- Re-running Module 1 (MethCores_Extractor) and Module 2 (MethCores_Combinator) on the same epihaplotype matrix, CpG-position file and metadata should reproduce the same significant mCpG combinations, core structures, and clonality/E index values as the reference outputs.
- Every annotated significant mCpG combination must satisfy the chi-square test of independence with adjusted P-value ≤ 10−9.
- Each reported core must contain at least two methylated cytosines with statistically significant frequency.
- In the synthetic validation, the 10% amplified epialleles in Cond 3 and Cond 4 should escape detection by Modules 1 and 2 while Module 3 (MethCores_Analyst) identifies them, confirming the sensitivity hierarchy.
- Clonality Index must equal core frequency normalized to the average methylation of the population; the entanglement index E must start from the 5′-most CpG pair and equal each mCpG's frequency in the core normalized to its population frequency.

## Limitations

- Reads with ambiguous calls (gaps or A or G) at the CpG dinucleotide are removed, so noisy molecules are dropped rather than imputed.
- Combinations below the P ≤ 10−9 significance threshold are not annotated; in the absence of significant mCpG combinations, all epialleles are carried to the next step instead of cores.
- Many R dependencies (vegan, Hmisc, FunChisq, entropy, ComplexHeatmap, etc.) are required; installation via pacman, docker, or Install_Dependencies.R is recommended.
- If significant epialleles are not found in Module 3, only the highest-frequency epiallele is annotated and its structure taken as the stable signature/core.

## Evidence
