---
name: epiallele-methylation-state-extraction
description: Use when bisulfite amplicon sequencing reads (FASTQ, e.g. from ENA PRJEB28662)
  must be converted into phased single-molecule CpG methylation states and stable
  mCpG combinations (cores) traced in complex DNA populations.
when_to_use_negative:
- The goal is only quantitative methylation assessment of single CpGs or abundance
  of single epialleles — tools like EBA, ampliMethProfiler alone, or methclone suffice
  and will not resolve phased-core structure
- Reads have ambiguous CpG calls (gaps, A or G at CpG dinucleotides), fail the ≥98%
  bisulfite efficiency or ≥60% reference-alignment filters, or fall outside 400–500
  nt / ±50% of reference length
- 10% rare amplified epialleles must be detected — under the tested conditions such
  low-abundance epialleles escape detection by Modules 1 and 2
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0654
tools:
- name: MethCoresProfiler
  role: Three R modules (MethCores_Extractor, MethCores_Combinator, MethCores_Analyst)
    that extract, test, and analyze significant phased mCpG combinations and epiallele
    families
  repo: https://github.com/84AP/MethCoresProfiler/
- name: ampliMethProfiler
  role: Extracts mCpG configurations in single DNA molecules from FASTA reads and
    produces the binary epihaplotype files consumed by MethCoresProfiler Module 1
  repo: https://sourceforge.net/projects/amplimethprofiler/
- name: PEAR
  role: Merges paired-end FASTQ reads with a minimum 40 nucleotide overlapping region
- name: PRINSEQ
  role: Converts quality-filtered reads to FASTA format prior to ampliMethProfiler
- name: R
  role: Language in which MethCoresProfiler and its three modules are implemented
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqaa096
    title: Tracing and tracking epiallele families in complex DNA populations
  source_content_hash: sha256:c24d0e92149cc7bf1446a621849915d62fc206c8ff7a1c31dc12b02258695c12
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
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# epiallele-methylation-state-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts single-molecule CpG methylation configurations (binary epihaplotypes) from bisulfite amplicon reads and identifies stable combinations of phased methylated CpGs (methylated cores) shared by all components of epiallele families, using MethCoresProfiler's three R modules plus ampliMethProfiler for read-level methylation calling.

## When to use

Use when bisulfite amplicon sequencing reads (FASTQ, e.g. from ENA PRJEB28662) must be converted into phased single-molecule CpG methylation states and stable mCpG combinations (cores) traced in complex DNA populations.

## When NOT to use

- The goal is only quantitative methylation assessment of single CpGs or abundance of single epialleles — tools like EBA, ampliMethProfiler alone, or methclone suffice and will not resolve phased-core structure
- Reads have ambiguous CpG calls (gaps, A or G at CpG dinucleotides), fail the ≥98% bisulfite efficiency or ≥60% reference-alignment filters, or fall outside 400–500 nt / ±50% of reference length
- 10% rare amplified epialleles must be detected — under the tested conditions such low-abundance epialleles escape detection by Modules 1 and 2

## Inputs

- Paired-end FASTQ bisulfite amplicon reads (ENA accession PRJEB28662)
- FASTA-format reads after PEAR merging, Phred>33 quality filtering (400–500 nt), and PRINSEQ conversion
- Tab-delimited binary EpiHaplotype file (file.out) from ampliMethProfiler
- Tab-delimited CpG position file for the sequenced region
- Sample metadata file with columns #SampleID, Tissue, Description, Group, Rep, ID

## Outputs

- Frequency tables of single-mCpG methylation profiles and two-mCpG co-occurrence
- Significant methylated-core compositions and epiallele tables (ComposCore, Tab_Epialleles)
- Hierarchical clustering heatmaps and PCA of significant epialleles
- Per-sample MethCores Index, Clonality Index, and E (stability/entanglement) index
- Shannon and Renyi entropy statistics of epiallele populations

## How to apply

Merge paired-end FASTQ reads with PEAR (minimum 40 nt overlapping region), retain reads with mean Phred >33 and length 400–500 nt, convert to FASTA with PRINSEQ, then run ampliMethProfiler keeping reads with length ±50% of the reference, ≥80% primer similarity, ≥98% bisulfite efficiency, ≥60% base alignment, and removing reads with ambiguous CpG calls. Feed the resulting binary EpiHaplotype file, a CpG-position file, and a sample-metadata file (#SampleID, Tissue, Description, Group, Rep, ID) into MethCoresProfiler Module 1 (MethCores_Extractor), which performs y=1000 repeated iterative samplings at the average depth (b4) and annotates significant two-mCpG combinations by chi-square independence testing (P ≤ 10−9). Module 2 (MethCores_Combinator) crosses significant pairs, tests higher-order (≥3 mCpGs) combinations against independent-event expectations, and reports core and epiallele structure/frequency (ComposCore, Tab_Epialleles). Module 3 (MethCores_Analyst) generates heatmaps, PCA, compares each epiallele's frequency to a random theoretical control by chi-square test, and computes the MethCores, Clonality, and E (stability/entanglement) indices per population.

## Related tools

- **MethCoresProfiler** (Three R modules (MethCores_Extractor, MethCores_Combinator, MethCores_Analyst) that extract, test, and analyze significant phased mCpG combinations and epiallele families) — https://github.com/84AP/MethCoresProfiler/
- **ampliMethProfiler** (Extracts mCpG configurations in single DNA molecules from FASTA reads and produces the binary epihaplotype files consumed by MethCoresProfiler Module 1) — https://sourceforge.net/projects/amplimethprofiler/
- **PEAR** (Merges paired-end FASTQ reads with a minimum 40 nucleotide overlapping region)
- **PRINSEQ** (Converts quality-filtered reads to FASTA format prior to ampliMethProfiler)
- **R** (Language in which MethCoresProfiler and its three modules are implemented)

## Examples

```
Rscript Run_MethCoresProfiler_setup.R  # set MethCoresProfiler=("/path-to-MethCoresProfiler-master/") and Data="testData/" first
```

## Evaluation signals

- ampliMethProfiler output passes all filters: read length ±50% of reference, ≥80% primer similarity, ≥98% bisulfite efficiency, ≥60% base alignment, no ambiguous CpG calls
- Significant two-mCpG combinations annotated at chi-square independence P-value ≤ 10−9 after y=1000 iterative samplings at the average read depth b4
- Minimum core contains at least two methylated cytosines with statistically significant frequency (adjusted P ≤ 10−9)
- Each epiallele's frequency is compared by chi-square test to a random control (R) with the same number of CpGs and depth, and significant epialleles are marked in the heatmap
- MethCores, Clonality, and E indices are produced per sample; validation expects astrocytes and MOEs to share a common precursor core distinct from the neuron/whole-cerebellum epiallele family (Figure 3H)

## Limitations

- 10% amplified epialleles in Cond 3 and Cond 4 escape detection by Modules 1 and 2
- Stable epialleles with phased methylated CpGs cannot be identified by current single-nucleotide techniques due to 5'-3' end heterogeneity and dilution by random unstable methylated CpGs
- MethCoresProfiler requires extensive R dependencies (vegan, ComplexHeatmap, FunChisq, etc.); installation via Install_Dependencies.R, pacman, or docker is recommended
- Software version for MethCoresProfiler (2.0) and ampliMethProfiler is not stated within the article text

## Evidence
