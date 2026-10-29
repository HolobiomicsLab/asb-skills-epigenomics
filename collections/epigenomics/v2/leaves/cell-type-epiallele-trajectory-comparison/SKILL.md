---
name: cell-type-epiallele-trajectory-comparison
description: Use when comparing methylated-core/epiallele structures across purified
  cell populations (e.g., astrocytes, neurons, MOEs from cerebellum and cortex) to
  test for shared precursor cores during postnatal differentiation.
when_to_use_negative:
- 'Input is raw bisulfite FASTQ: reads must first be merged with PEAR (≥40 nt overlap),
  quality-filtered (Phred >33, 400-500 nt) and converted to binary epihaplotype files
  with ampliMethProfiler before MethCoresProfiler can run.'
- The goal is single-CpG methylation quantification or single-epiallele abundance
  rather than shared phased mCpG combinations — the paper positions existing tools
  (EBA, ampliMethProfiler, methclone) in that space.
- Samples lack cell-type metadata (Tissue/Group/Rep columns) or replicates, so cross-population
  heatmaps, PCA and index comparisons cannot be meaningfully interpreted.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3295
tools:
- name: MethCoresProfiler
  role: Three R modules (MethCores_Extractor, MethCores_Combinator, MethCores_Analyst)
    that extract significant mCpG cores, annotate epialleles, and perform the cross-cell-type
    clustering, PCA and index comparison
  repo: https://github.com/84AP/MethCoresProfiler/
- name: ampliMethProfiler
  role: Extracts single-molecule CpG methylation configurations (binary epihaplotypes)
    from quality-filtered FASTA reads of each purified cell type, applying the length/primer/bisulfite/alignment
    filters
  repo: https://sourceforge.net/projects/amplimethprofiler/
- name: PEAR
  role: Merges paired-end FASTQ reads per cell-type sample, setting a minimum of 40
    nucleotides as the overlapping region
- name: PRINSEQ
  role: Applies quality filtering (mean Phred >33, 400-500 nt) and converts merged
    reads to FASTA format for ampliMethProfiler
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqaa096
    title: Tracing and tracking epiallele families in complex DNA populations
  source_content_hash: sha256:1e7ed1be703066474872652902b2362f026cf78a32262a9d065d93dae40115e1
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# cell-type-epiallele-trajectory-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Cell-type-resolved comparison of epiallele families: MethCoresProfiler (R) extracts significant phased mCpG cores from bisulfite amplicons of each purified cell population, then clusters and compares them (heatmaps, PCA, MethCores/Clonality/E indices) to reveal shared precursor cores versus divergence across cell types during postnatal differentiation.

## When to use

Use when comparing methylated-core/epiallele structures across purified cell populations (e.g., astrocytes, neurons, MOEs from cerebellum and cortex) to test for shared precursor cores during postnatal differentiation.

## When NOT to use

- Input is raw bisulfite FASTQ: reads must first be merged with PEAR (≥40 nt overlap), quality-filtered (Phred >33, 400-500 nt) and converted to binary epihaplotype files with ampliMethProfiler before MethCoresProfiler can run.
- The goal is single-CpG methylation quantification or single-epiallele abundance rather than shared phased mCpG combinations — the paper positions existing tools (EBA, ampliMethProfiler, methclone) in that space.
- Samples lack cell-type metadata (Tissue/Group/Rep columns) or replicates, so cross-population heatmaps, PCA and index comparisons cannot be meaningfully interpreted.

## Inputs

- Paired-end FASTQ reads per purified cell-type sample (ENA accession PRJEB28662)
- Tab-delimited text file of EpiHaplotypes in BINARY format (from ampliMethProfiler)
- Tab-delimited text file of CpG positions in the sequence
- Sample metadata file with columns #SampleID, Tissue, Description, Group, Rep, ID

## Outputs

- ComposCore table: structure and frequency of significant mCpG combination cores per cell type
- Tab_Epialleles table: annotated epialleles per sample
- Hierarchical-clustering heatmaps and PCA plots of significant epialleles across cell types (red=high, blue=low, white=neutral)
- Per-cell-type MethCores Index, Clonality Index, and E (stability/entanglement) index values

## How to apply

Process amplicon bisulfite reads per cell type: merge paired-end FASTQ with PEAR (minimum 40 nucleotides overlap), retain reads with mean Phred >33 and 400-500 nt length, convert to FASTA with PRINSEQ, and call single-molecule CpG configurations with ampliMethProfiler using filters of length ±50% of the reference, ≥80% primer similarity, ≥98% bisulfite efficiency, ≥60% base alignment, and removal of reads with ambiguous CpG calls. Run MethCores_Extractor (Module 1) on the binary epihaplotype, CpG-position and metadata files (#SampleID, Tissue, Description, Group, Rep, ID), with y=1000 iterative samplings at average depth to annotate significant two-mCpG combinations (chi-square independence test, P-value ≤ 10−9). Run MethCores_Combinator (Module 2) to cross significant pairs, test higher-order mCpG combinations, and report core structure/frequency (ComposCore) and epialleles (Tab_Epialleles) per cell type. Run MethCores_Analyst (Module 3) to generate hierarchical-clustering heatmaps and PCA across cell-type samples, testing each epiallele against a random theoretical control (R) with the same number of CpGs and depth, and compute the MethCores, Clonality, and E (stability/entanglement) indices per population. Judge success by whether core structures reproduce the reported cell-type organization, e.g., astrocytes and MOEs sharing a common precursor core distinct from the neuron/whole-cerebellum epiallele family (Figure 3H).

## Related tools

- **MethCoresProfiler** (Three R modules (MethCores_Extractor, MethCores_Combinator, MethCores_Analyst) that extract significant mCpG cores, annotate epialleles, and perform the cross-cell-type clustering, PCA and index comparison) — https://github.com/84AP/MethCoresProfiler/
- **ampliMethProfiler** (Extracts single-molecule CpG methylation configurations (binary epihaplotypes) from quality-filtered FASTA reads of each purified cell type, applying the length/primer/bisulfite/alignment filters) — https://sourceforge.net/projects/amplimethprofiler/
- **PEAR** (Merges paired-end FASTQ reads per cell-type sample, setting a minimum of 40 nucleotides as the overlapping region)
- **PRINSEQ** (Applies quality filtering (mean Phred >33, 400-500 nt) and converts merged reads to FASTA format for ampliMethProfiler)

## Examples

```
Rscript Run_MethCoresProfiler_setup.R  # after setting MethCoresProfiler="/path-to-MethCoresProfiler-master/", Data="testData/", input="out/"
```

## Evaluation signals

- Every annotated two-mCpG combination passes chi-square independence testing with P-value ≤ 10−9; the minimum core contains at least two methylated cytosines with significant frequency.
- Module 2 outputs ComposCore and Tab_Epialleles for every cell type, with higher-order (≥3 mCpG) combinations tested against the expected frequency for independent events.
- Module 3 compares each epiallele's frequency to a random control (R) with the same number of CpGs and depth, marking significant epialleles in the heatmap (red=high, blue=low, white=neutral).
- Astrocyte and MOE samples cluster with a shared precursor core distinct from the neuron/whole-cerebellum epiallele family (Figure 3H), reproducing the reported cell-type separation.
- Per-sample outputs include all three indices (MethCores, Clonality, E) computed on the decreasing frequency scale starting from 0.9 (minimum 2 mCpGs).

## Limitations

- Rare amplified epialleles (~10% under synthetic Cond 3 and Cond 4 mixes) escape detection by Modules 1 and 2.
- If no significant epialleles are found at Module 3, only the highest-frequency epiallele is annotated and treated as the stable core signature, weakening cross-cell-type trajectory claims.
- Key validation comparisons with existing tools (EBA, ampliMethProfiler, methclone) are deferred to supplementary material (Supplementary Table S4; Figures S4A, S5C, S8, S14) rather than detailed in the main text.

## Evidence
