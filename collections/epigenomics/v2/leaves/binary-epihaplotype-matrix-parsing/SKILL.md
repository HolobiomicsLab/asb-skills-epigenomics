---
name: binary-epihaplotype-matrix-parsing
description: Use when bisulfite-sequencing reads have been processed by ampliMethProfiler
  into a binary epiallele/epihaplotype matrix and you need to load CpG positions and
  sample metadata for methylated-core analysis.
when_to_use_negative:
- Input is still raw FASTQ/FASTA bisulfite reads — these must first be merged with
  PEAR, quality-filtered (Phred >33, 400–500 nt), converted with PRINSEQ, and processed
  by ampliMethProfiler before a binary epihaplotype matrix exists
- The data lack phased CpG information (e.g. only per-CpG bulk methylation levels),
  since methylated cores require phased methylation configurations in single DNA molecules
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3295
tools:
- name: MethCoresProfiler
  role: R tool whose Module 1 (MethCores_Extractor) parses the binary epihaplotype
    matrix, computes average depth b4, and runs repeated iterative sampling
  repo: https://github.com/84AP/MethCoresProfiler/
- name: ampliMethProfiler
  role: Upstream tool that extracts mCpG configurations in single DNA molecules and
    generates the reads used to build the binary epihaplotype matrix
  repo: https://sourceforge.net/projects/amplimethprofiler/
- name: R
  role: Language in which the parsing and downstream modules of MethCoresProfiler
    are implemented
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqaa096
    title: Tracing and tracking epiallele families in complex DNA populations
  source_content_hash: sha256:a3c21bd1a4a59451a8dea6d675131adb76972d796115de647f905a293e8585ff
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# binary-epihaplotype-matrix-parsing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parse the tab-delimited binary epihaplotype matrix (file.out) plus CpG-position and sample-metadata files that MethCoresProfiler requires as input, so that phased mCpG configurations per DNA molecule can be extracted, sampled at depth b4, and analyzed for methylated cores.

## When to use

Use when bisulfite-sequencing reads have been processed by ampliMethProfiler into a binary epiallele/epihaplotype matrix and you need to load CpG positions and sample metadata for methylated-core analysis.

## When NOT to use

- Input is still raw FASTQ/FASTA bisulfite reads — these must first be merged with PEAR, quality-filtered (Phred >33, 400–500 nt), converted with PRINSEQ, and processed by ampliMethProfiler before a binary epihaplotype matrix exists
- The data lack phased CpG information (e.g. only per-CpG bulk methylation levels), since methylated cores require phased methylation configurations in single DNA molecules

## Inputs

- Tab-delimited text file of epihaplotypes in BINARY format (file.out) from ampliMethProfiler
- Tab-delimited text file containing CpG positions in the sequence (cgpos)
- Tab-delimited sample metadata file with columns #SampleID, Tissue, Description, Group, Rep, ID (meta.map)

## Outputs

- Tables of single-mCpG methylation frequencies
- Tetrachoric correlation table of CpGs
- mCpG co-occurrence table
- Shannon-Entropy index and summary/sample statistics and plots
- Random control (R) epiallele distribution with the same number of CpGs and depth
- Significant mCpG combinations and core structures (ComposCore and Tab_Epialleles tables from Module 2)

## How to apply

First, after ampliMethProfiler extracts mCpG configurations in single DNA molecules, encode each read as a row of a tab-delimited text file of EpiHaplotypes in BINARY format (file.out), and prepare two companion tab-delimited files: one with CpG positions in the sequence and one metadata file with columns #SampleID, Tissue, Description, Group, Rep, ID. Load all three into MethCores_Extractor (Module 1), which computes the average read depth b4 and performs y = 1000 repeated iterative samplings of each sample at depth b4 to reduce size-comparison errors. The parsed matrix then feeds Module 1 statistics (single-mCpG frequencies, tetrachoric correlations, mCpG co-occurrence, Shannon and Renyi entropies) and Module 2 (MethCores_Combinator), which tests mCpG combinations against expected independent-event frequencies (chi-square, P ≤ 10−9). A random control (R) epiallele set with the same number of CpGs and depth is generated automatically by Module 1 from the parsed matrix for downstream comparison in Module 3.

## Related tools

- **MethCoresProfiler** (R tool whose Module 1 (MethCores_Extractor) parses the binary epihaplotype matrix, computes average depth b4, and runs repeated iterative sampling) — https://github.com/84AP/MethCoresProfiler/
- **ampliMethProfiler** (Upstream tool that extracts mCpG configurations in single DNA molecules and generates the reads used to build the binary epihaplotype matrix) — https://sourceforge.net/projects/amplimethprofiler/
- **R** (Language in which the parsing and downstream modules of MethCoresProfiler are implemented)

## Examples

```
Rscript 1_MethCores_Extractor.R  # after setting MethCoresProfiler="/path-to-MethCoresProfiler-master/" and input="out/" in Run_MethCoresProfiler_setup.R
```

## Evaluation signals

- Module 1 reports a single average depth value b4 consistent with the number of rows (epialleles) and CpG columns in the binary matrix
- Repeated iterative sampling completes the default y = 1000 iterations per sample at depth b4
- Single-mCpG frequency, tetrachoric correlation, and co-occurrence tables have row counts matching the CpG positions in the cgpos file
- Re-running Modules 1 and 2 on the same input reproduces the same significant mCpG combinations, core structures, and clonality/E index values
- The random control (R) distribution generated by Module 1 has the same number of CpGs and depth as the parsed sample

## Limitations

- Reads with ambiguous calls (gaps, A or G) at CpG dinucleotides must be removed upstream; parsing assumes clean binary calls
- In the reported validation, the 10% amplified epialleles in Cond 3 and Cond 4 escape detection by Modules 1 and 2 (only Module 3 identifies them)
- In the absence of significant mCpGs combinations, all epialleles are passed forward by Module 2, so the matrix alone may not yield methylated cores

## Evidence

- [readme] requires three types of input files: i. a tab delimited text file of EpiHaplotypes in BINARY format; ii. a tab delimited text file containing information on the CpGs position in the sequence (or string), and; iii. a tab delimited text file containing metadata: "requires three types of input files: i. a tab delimited text file of EpiHaplotypes in BINARY format; ii. a tab delimited text file containing information on the CpGs position"
- [methods] performs and summarizes (y) repeated and iterative sampling (default y = 1000) of each experimental sample using b4 as a depth: "performs and summarizes (y) repeated and iterative sampling (default y = 1000) of each experimental sample using b4 as a depth"
- [methods] The MethCores_Extractor, or Module 1, calculates the average depth of the sample reads (named b4): "The MethCores_Extractor, or Module 1, calculates the average depth of the sample reads (named b4)"
- [methods] Under these conditions, the 10% amplified epiallele(s) in Cond 3 and Cond 4 escape(s) detection by Modules 1 and 2.: "Under these conditions, the 10% amplified epiallele(s) in Cond 3 and Cond 4 escape(s) detection by Modules 1 and 2."
- [methods] Reads with ambiguous calls (including gaps or A or G) at the CpG dinucleotide were removed: "Reads with ambiguous calls (including gaps or A or G) at the CpG dinucleotide were removed"
