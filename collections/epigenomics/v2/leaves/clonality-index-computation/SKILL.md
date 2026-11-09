---
name: clonality-index-computation
description: Use when bisulfite amplicon epihaplotype tables are available and you
  need to distinguish clonally expanded (dominant) epiallele families from stochastic
  methylation by scoring each methylated core with a clonality index and E index.
when_to_use_negative:
- Input is raw FASTQ/FASTA or per-read methylation calls rather than Module-2-annotated
  epialleles/cores — run PEAR, PRINSEQ, ampliMethProfiler and Modules 1–2 first.
- Goal is quantifying methylation of single CpGs or abundance of single epialleles
  without assessing phased core clonality — single-nucleotide methods or tools like
  EBA/methclone cover that scope.
- The population contains rare amplified epialleles at ~10% frequency of the kind
  in Cond 3/Cond 4 — these escape detection by Modules 1 and 2, so clonality values
  derived from them would be misleading without Module 3-based identification.
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_0092
tools:
- name: MethCoresProfiler
  role: R package whose MethCores_Analyst (Module 3) computes the MethCores, clonality
    and entanglement (E) indices for each sample from Module-2-annotated epialleles
  repo: https://github.com/84AP/MethCoresProfiler/
- name: ampliMethProfiler
  role: Upstream tool producing the binary epihaplotype tables that Module 3 consumes
  repo: https://sourceforge.net/projects/amplimethprofiler/
- name: R
  role: Language in which MethCoresProfiler (and hence the index computations) is
    implemented
  repo: https://github.com/84AP/MethCoresProfiler/
provenance:
  source_task_ids:
  - task_005
  - task_001
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqaa096
    title: Tracing and tracking epiallele families in complex DNA populations
  source_content_hash: sha256:ee10f7dd6ee9fa09e41cdda7a590da1ac59778b72b27e3f0c2f03e41897aad65
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# clonality-index-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes the clonality index — the frequency of methylated cores normalized to the average methylation of the population — alongside the MethCores Index and the entanglement (E) index, as implemented in MethCoresProfiler Module 3 (MethCores_Analyst) to quantify clonal dominance of epiallele families in complex DNA populations.

## When to use

Use when bisulfite amplicon epihaplotype tables are available and you need to distinguish clonally expanded (dominant) epiallele families from stochastic methylation by scoring each methylated core with a clonality index and E index.

## When NOT to use

- Input is raw FASTQ/FASTA or per-read methylation calls rather than Module-2-annotated epialleles/cores — run PEAR, PRINSEQ, ampliMethProfiler and Modules 1–2 first.
- Goal is quantifying methylation of single CpGs or abundance of single epialleles without assessing phased core clonality — single-nucleotide methods or tools like EBA/methclone cover that scope.
- The population contains rare amplified epialleles at ~10% frequency of the kind in Cond 3/Cond 4 — these escape detection by Modules 1 and 2, so clonality values derived from them would be misleading without Module 3-based identification.

## Inputs

- Tab-delimited binary epihaplotype matrix (file.out) from ampliMethProfiler processing of FASTA reads
- Tab-delimited CpG position file (cgpos) describing CpG positions in the sequence
- Tab-delimited sample metadata file with columns #SampleID, Tissue, Description, Group, Rep, ID (meta.map)
- List of regions and epiallele tables annotated by MethCores_Combinator (Tab_Epialleles, ComposCore)

## Outputs

- Clonality Index per methylated core (core frequency normalized to average population methylation)
- MethCores Index per core (frequency of methylated cores in the population)
- Entanglement / Stability / in-phase CpGs Index (E) per mCpG in the core
- Heatmaps of epiallele structure and frequency with significant epialleles marked, plus PCA of significant epialleles

## How to apply

Feed the binary epihaplotype matrix, CpG-position file and sample metadata (#SampleID, Tissue, Description, Group, Rep, ID) through MethCores_Extractor (Module 1) with repeated iterative sampling (default y = 1000 at average depth b4) and chi-square independence testing at P ≤ 10−9 to annotate significant mCpG combinations; then run MethCores_Combinator (Module 2) to extract significant cores (minimum two methylated cytosines) and produce the ComposCore and Tab_Epialleles tables. Module 3 (MethCores_Analyst) compares each epiallele's frequency to a random theoretical control (R) by chi-square test, then extracts the most frequent mCpGs shared by significant epialleles on a decreasing percentage scale starting from 0.9 (minimum two mCpGs). It then reports three indices: the MethCores Index (core frequency in the population), the Clonality Index (core frequency normalized to the average population methylation), and the in-phase CpGs / Entanglement Index E (rate of coupling of at least two mCpGs, mCpG1 and mCpG2, in the core). If no significant epialleles are found, the epiallele with the highest frequency is annotated and its structure treated as the stable signature/core. High clonality indices at later time points relative to earlier ones (e.g. P90 vs P1 gut, or across CB, CX, HIPP at P1–P60) indicate dominant epigenetic families.

## Related tools

- **MethCoresProfiler** (R package whose MethCores_Analyst (Module 3) computes the MethCores, clonality and entanglement (E) indices for each sample from Module-2-annotated epialleles) — https://github.com/84AP/MethCoresProfiler/
- **ampliMethProfiler** (Upstream tool producing the binary epihaplotype tables that Module 3 consumes) — https://sourceforge.net/projects/amplimethprofiler/
- **R** (Language in which MethCoresProfiler (and hence the index computations) is implemented) — https://github.com/84AP/MethCoresProfiler/

## Examples

```
Rscript Run_MethCoresProfiler_setup.R
```

## Evaluation signals

- Outputs include all three indices — MethCores Index, Clonality Index, and E (entanglement/stability) index — for every core in each sample.
- Reported cores contain at least two methylated cytosines with significant frequency (chi-square adjusted P ≤ 10−9), matching the minimum-core rule.
- Clonality index values reproduce published patterns (e.g. dominant high-clonality core at P90 absent/weaker at P1; E index values consistent with Supplementary Table S3), including stable cores like '−336|−330' in CB and HIPP and the CX '−242|−175' → '−336|−318|−242' switch.
- Reconstructed index values from a reimplementation match the original MethCoresProfiler Modules 1–2 outputs on the same inputs.
- The 0.9 decreasing percentage scale (minimum two mCpGs) is used to extract the most frequent mCpGs shared by significant epialleles; if none are significant, the highest-frequency epiallele is annotated as the core.

## Limitations

- Rare amplified epialleles (~10% in Cond 3 and Cond 4) escape detection by Modules 1 and 2; Module 3 is required to identify them, so index values from Modules 1–2 alone are incomplete in such populations.
- If significant epialleles are not found, the pipeline falls back to annotating the single highest-frequency epiallele as the core, which is a heuristic rather than a statistically supported signature.
- The iterative sampling default (y = 1000 at depth b4) must be honored to reduce size-comparison errors between samples of different depth.

## Evidence
