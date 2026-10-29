---
name: epiallele-family-trajectory-comparison
description: Use when comparing bisulfite amplicon-seq epihaplotype data across two
  or more conditions to test whether specific families of phased mCpG combinations
  (methylated cores) expand clonally or remain stochastic between time points or populations.
when_to_use_negative:
- The question concerns single-CpG methylation levels or single-epiallele abundance,
  which tools like EBA, ampliMethProfiler or methclone already quantify without phased-core
  tracking
- Input reads are shotgun bisulfite sequencing rather than amplicon-based reads matching
  the PEAR merge / 400-500 nt length filter workflow
- The comparison does not require distinguishing clonally expanded epiallele families
  from stochastic methylation (e.g., one-population descriptive methylation profiling
  only)
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0091
tools:
- name: MethCoresProfiler
  role: R tool (Modules 1-3) that extracts methylated cores, annotates significant
    mCpG combinations, and computes the clonality and stability/entanglement indices
    used for trajectory comparison
  repo: https://github.com/84AP/MethCoresProfiler/
- name: PEAR
  role: Merges paired-end bisulfite amplicon FASTQ reads with a minimum 40 nucleotides
    overlap before quality filtering
- name: PRINSEQ
  role: Converts quality-filtered reads to FASTA format for ampliMethProfiler processing
- name: ampliMethProfiler
  role: Produces the binary epihaplotype tables per sample that MethCoresProfiler
    consumes as input
  repo: https://sourceforge.net/projects/amplimethprofiler/
- name: R
  role: Language in which MethCoresProfiler and its three modules are implemented
    and executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqaa096
    title: Tracing and tracking epiallele families in complex DNA populations
  source_content_hash: sha256:53a166d72693dcfc520d1c2e481dcc198480803b5b97acaba00aac105c7d978b
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
yes_rate: 1.0
n_claims: 6
---

# epiallele-family-trajectory-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantify and compare epiallele family trajectories across populations (e.g., P1 vs P90 mouse gut) by extracting methylated cores with MethCoresProfiler and computing its clonality index (core frequency normalized to average population methylation) and stability/entanglement index to distinguish clonally expanded dominant families from stochastic methylation.

## When to use

Use when comparing bisulfite amplicon-seq epihaplotype data across two or more conditions to test whether specific families of phased mCpG combinations (methylated cores) expand clonally or remain stochastic between time points or populations.

## When NOT to use

- The question concerns single-CpG methylation levels or single-epiallele abundance, which tools like EBA, ampliMethProfiler or methclone already quantify without phased-core tracking
- Input reads are shotgun bisulfite sequencing rather than amplicon-based reads matching the PEAR merge / 400-500 nt length filter workflow
- The comparison does not require distinguishing clonally expanded epiallele families from stochastic methylation (e.g., one-population descriptive methylation profiling only)

## Inputs

- Binary-format epihaplotype tables (tab-delimited file.out from ampliMethProfiler) per sample
- Tab-delimited CpG position file mapping CpG positions within the amplicon sequence
- Sample metadata file with columns #SampleID, Tissue, Description, Group, Rep, ID
- Region list input for MethCores_Combinator and MethCores_Analyst

## Outputs

- ComposCore table reporting structure and frequency of significant methylated cores
- Tab_Epileleles table of extracted epialleles containing significant mCpG combinations
- Clonality index values per core per population
- MethCores Index (frequency of methylated cores in the population)
- Stability/Entanglement (in-phase CpGs) Index per CpG in the core
- Heatmaps and PCA of significant epialleles from MethCores_Analyst

## How to apply

Process bisulfite amplicon reads (ENA PRJEB28662) by merging paired-end FASTQ with PEAR (minimum 40 nt overlap), retaining reads with mean Phred >33 and 400-500 nt length, converting to FASTA with PRINSEQ, and calling binary epihaplotypes with ampliMethProfiler using quality filters (length ±50% of reference, ≥80% primer similarity, ≥98% bisulfite efficiency, ≥60% base alignment; ambiguous CpG calls removed). Run MethCoresProfiler Modules 1-3 (MethCores_Extractor, MethCores_Combinator, MethCores_Analyst) on the binary epihaplotype tables with CpG-position and metadata inputs, using default repeated sampling (y = 1000) and chi-square independence testing at adjusted P-value ≤ 10−9, requiring a minimum core of at least two methylated cytosines with statistically significant frequency. From the ComposCore and index outputs, compute the clonality index (core frequency normalized to average methylation of the population) and the entanglement/stability index (rate of coupling of ≥2 mCpGs in the core) for each condition, then compare indices between populations to identify a dominant, high-clonality epiallele family that emerges in one condition (e.g., P90 postnatal gut differentiation) versus the other (P1).

## Related tools

- **MethCoresProfiler** (R tool (Modules 1-3) that extracts methylated cores, annotates significant mCpG combinations, and computes the clonality and stability/entanglement indices used for trajectory comparison) — https://github.com/84AP/MethCoresProfiler/
- **PEAR** (Merges paired-end bisulfite amplicon FASTQ reads with a minimum 40 nucleotides overlap before quality filtering)
- **PRINSEQ** (Converts quality-filtered reads to FASTA format for ampliMethProfiler processing)
- **ampliMethProfiler** (Produces the binary epihaplotype tables per sample that MethCoresProfiler consumes as input) — https://sourceforge.net/projects/amplimethprofiler/
- **R** (Language in which MethCoresProfiler and its three modules are implemented and executed)

## Examples

```
Rscript Run_MethCoresProfiler_setup.R  # after setting MethCoresProfiler="/path-to-MethCoresProfiler-master/" and Data="testData/" with input="out/" binary epihaplotype tables
```

## Evaluation signals

- Results table exists and contains core structures plus clonality index values for each population
- All annotated cores satisfy the chi-square adjusted P-value ≤ 10−9 significance threshold and contain at least two methylated cytosines
- The comparison reproduces a dominant, high-clonality methylated core at P90 that is absent or much weaker at P1, consistent with Supplementary Figure S8
- Module outputs include the expected ComposCore and index files and MethCores_Analyst heatmaps/PCA of significant epialleles
- Repeated iterative sampling uses the default y = 1000 samplings at average depth (b4)

## Limitations

- 10% amplified epialleles under certain conditions can escape detection by Modules 1 and 2
- Key validation evidence (Supplementary Table S4, Figures S4A, S5C, S8, S14) is deferred to supplementary materials; no software version is stated in some sections
- Reads with ambiguous CpG calls (gaps, A or G at the CpG dinucleotide) must be removed, so heavily damaged or incompletely converted amplicons reduce usable depth
- In the absence of significant mCpGs combinations, all epialleles are passed to the next step, and if no significant epialleles are found the highest-frequency epiallele is annotated as the core, which may not reflect a true expanded family

## Evidence
