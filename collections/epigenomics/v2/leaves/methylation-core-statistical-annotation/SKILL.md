---
name: methylation-core-statistical-annotation
description: Use when amplicon bisulfite reads have been converted to binary epihaplotype
  matrices of phased CpG methylation states and the goal is to extract significant
  mCpG combinations, cores, clonality/entanglement indices, or track epiallele families
  across cell types or time points.
when_to_use_negative:
- The analysis target is quantitative methylation of individual CpGs or abundance
  of single epialleles — MethCoresProfiler was designed specifically for stable combinations
  of phased mCpGs shared by epiallele families
- Input reads have not been reduced to binary epihaplotype matrices and contain ambiguous
  CpG calls, low bisulfite efficiency (<98%), or insufficient overlap after merging
  (<40 nt)
- Only a handful of CpGs are in the locus, so a minimum core of two significantly
  co-occurring mCpGs cannot be established
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0091
tools:
- name: MethCoresProfiler
  role: R-based tool whose three modules (MethCores_Extractor, MethCores_Combinator,
    MethCores_Analyst) extract, statistically annotate, and visualize methylated cores
    and epiallele families
  repo: https://github.com/84AP/MethCoresProfiler/
- name: ampliMethProfiler
  role: Upstream tool to extract mCpG configurations in single DNA molecules from
    FASTA reads and generate the binary epihaplotype files consumed by Module 1
  repo: https://sourceforge.net/projects/amplimethprofiler/
- name: PEAR
  role: Merges paired-end FASTQ reads (minimum 40 nucleotides overlap) from ENA accession
    PRJEB28662 before epihaplotype extraction
- name: PRINSEQ
  role: Quality-filters and converts reads to FASTA format (mean Phred >33, length
    400–500 nt) prior to ampliMethProfiler
- name: R
  role: Language in which MethCoresProfiler is implemented; requires packages such
    as vegan, Hmisc, data.table, FunChisq, ComplexHeatmap installed via Install_Dependencies.R
  repo: https://github.com/84AP/MethCoresProfiler/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqaa096
    title: Tracing and tracking epiallele families in complex DNA populations
  source_content_hash: sha256:ce33017f24b7b7863b1a9150588c3e705faac43f47fbafe92ecb10c524d8d186
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9565217391304348
n_claims: 23
---

# methylation-core-statistical-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Statistically annotate stable combinations of methylated phased CpGs ('methylated cores') that mark epiallele families in complex bisulfite-sequencing populations, using the three MethCoresProfiler R modules (MethCores_Extractor, MethCores_Combinator, MethCores_Analyst) with chi-square significance testing at P ≤ 10−9.

## When to use

Use when amplicon bisulfite reads have been converted to binary epihaplotype matrices of phased CpG methylation states and the goal is to extract significant mCpG combinations, cores, clonality/entanglement indices, or track epiallele families across cell types or time points.

## When NOT to use

- The analysis target is quantitative methylation of individual CpGs or abundance of single epialleles — MethCoresProfiler was designed specifically for stable combinations of phased mCpGs shared by epiallele families
- Input reads have not been reduced to binary epihaplotype matrices and contain ambiguous CpG calls, low bisulfite efficiency (<98%), or insufficient overlap after merging (<40 nt)
- Only a handful of CpGs are in the locus, so a minimum core of two significantly co-occurring mCpGs cannot be established

## Inputs

- Tab-delimited text file of epihaplotypes in binary format (per-molecule CpG methylation states)
- Tab-delimited text file of CpG positions in the sequence
- Sample metadata file with columns #SampleID, Tissue, Description, Group, Rep, ID
- List of target regions

## Outputs

- Annotated significant two-mCpG and higher-order mCpG combinations (cores) with chi-square P-values
- ComposCore (core structure) and Tab_Epialleles (epiallele structure and frequency) tables
- Hierarchical-clustering heatmaps and PCA of significant epialleles
- MethCores Index, Clonality Index, and E (stability/entanglement) Index per sample
- Shannon/Renyi entropy statistics and tetrachoric correlations of CpGs

## How to apply

First produce per-molecule binary epihaplotype files (e.g. with ampliMethProfiler after PEAR merging with ≥40 nt overlap and PRINSEQ FASTA conversion), retaining reads with mean Phred >33, length 400–500 nt, ±50% reference length, ≥80% primer similarity, ≥98% bisulfite efficiency, ≥60% base alignment, and no ambiguous CpG calls. Run Module 1 (MethCores_Extractor) on the binary epihaplotype, CpG-position, and metadata files, performing y = 1000 iterative samplings at the average read depth (b4) and annotating significant two-mCpG combinations by chi-square independence testing at P ≤ 10−9. Run Module 2 (MethCores_Combinator) to cross significant pairs, test higher-order (≥3 mCpGs) combinations against the independent-events expectation, and report core structure and epiallele frequencies (ComposCore and Tab_Epialleles tables); the minimum core contains at least two methylated cytosines with adjusted P ≤ 10−9. Run Module 3 (MethCores_Analyst) to generate heatmaps, PCA, compare each epiallele's frequency against a randomly generated theoretical control of the same CpG number and depth, and compute the MethCores, clonality, and E (stability/entanglement) indices per sample.

## Related tools

- **MethCoresProfiler** (R-based tool whose three modules (MethCores_Extractor, MethCores_Combinator, MethCores_Analyst) extract, statistically annotate, and visualize methylated cores and epiallele families) — https://github.com/84AP/MethCoresProfiler/
- **ampliMethProfiler** (Upstream tool to extract mCpG configurations in single DNA molecules from FASTA reads and generate the binary epihaplotype files consumed by Module 1) — https://sourceforge.net/projects/amplimethprofiler/
- **PEAR** (Merges paired-end FASTQ reads (minimum 40 nucleotides overlap) from ENA accession PRJEB28662 before epihaplotype extraction)
- **PRINSEQ** (Quality-filters and converts reads to FASTA format (mean Phred >33, length 400–500 nt) prior to ampliMethProfiler)
- **R** (Language in which MethCoresProfiler is implemented; requires packages such as vegan, Hmisc, data.table, FunChisq, ComplexHeatmap installed via Install_Dependencies.R) — https://github.com/84AP/MethCoresProfiler/

## Examples

```
Rscript Run_MethCoresProfiler_setup.R  # with MethCoresProfiler="/path-to-MethCoresProfiler-master/" and Data="testData/", input="out/" set in the script
```

## Evaluation signals

- Significant mCpG combinations pass chi-square independence testing with P-value ≤ 10−9, and the minimum core contains at least two methylated cytosines with adjusted P ≤ 10−9
- Module 2 reports ComposCore and Tab_Epialleles tables; if no significant mCpG combinations exist, all epialleles are forwarded to the next step rather than the run failing
- Each epiallele frequency is compared by chi-square test to a random theoretical control (R) with the same number of CpGs and depth, and only significant epialleles are marked in the heatmap (red = high, blue = low, white = neutral)
- Per-sample indices are produced: MethCores Index (core frequency), Clonality Index (core frequency normalized to average methylation), and E index (rate of coupling of at least 2 mCpGs)
- Downstream validation reproduces known biology, e.g. astrocytes and MOEs sharing a common precursor core that differs from the neuron/whole-cerebellum epiallele family (Figure 3H)

## Limitations

- Amplified rare epialleles (~10%) under certain conditions escape detection by Modules 1 and 2
- If no significant epialleles are found at Module 3, only the single highest-frequency epiallele is annotated as the stable core signature
- Stable epialleles cannot be identified by current single-nucleotide techniques due to heterogeneity of 5'-3' ends of molecules and dilution by random unstable methylated CpGs; results depend on the stringent read-filtering chain upstream
- The strict P ≤ 10−9 threshold and binary epihaplotype requirement mean noisy or short reads with ambiguous CpG calls (gaps, A or G) must be removed beforehand

## Evidence
