---
name: entanglement-index-computation
description: Use when bisulfite-sequencing epiallele data has been reduced to significant
  methylated cores and you must quantify per-CpG coupling (entanglement/stability
  index E) and core clonality in each population.
when_to_use_negative:
- Data are single-CpG methylation levels or unphased bulk methylation calls — phased
  epihaplotype matrices from bisulfite reads are required
- No statistically significant mCpG combinations exist after Module 1/2 filtering
  (chi-square adjusted P-value ≤ 10−9) — cores and their indices cannot be defined
- The goal is merely profiling single-epiallele abundance, which tools like ampliMethProfiler
  or methclone already provide without phased-core indices
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3295
tools:
- name: MethCoresProfiler
  role: R tool whose Modules 1 and 2 extract significant phased mCpG combinations
    (cores) from which the clonality index and entanglement/stability index E are
    computed
  repo: https://github.com/84AP/MethCoresProfiler/
- name: ampliMethProfiler
  role: Upstream tool that extracts mCpG configurations in single DNA molecules to
    produce the epihaplotype matrix input
  repo: https://sourceforge.net/projects/amplimethprofiler/
- name: R
  role: Language in which MethCoresProfiler modules and the index computations are
    implemented
  repo: https://github.com/84AP/MethCoresProfiler/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqaa096
    title: Tracing and tracking epiallele families in complex DNA populations
  source_content_hash: sha256:aaea8981e1f84763f5915d36e440251e005ca292ce39b487e2d28a878572e073
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# entanglement-index-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes the MethCoresProfiler clonality index and the stability/entanglement index (E) that quantify, per CpG and per core, how strongly phased mCpGs are coupled within epiallele families in a bisulfite-sequenced DNA population.

## When to use

Use when bisulfite-sequencing epiallele data has been reduced to significant methylated cores and you must quantify per-CpG coupling (entanglement/stability index E) and core clonality in each population.

## When NOT to use

- Data are single-CpG methylation levels or unphased bulk methylation calls — phased epihaplotype matrices from bisulfite reads are required
- No statistically significant mCpG combinations exist after Module 1/2 filtering (chi-square adjusted P-value ≤ 10−9) — cores and their indices cannot be defined
- The goal is merely profiling single-epiallele abundance, which tools like ampliMethProfiler or methclone already provide without phased-core indices

## Inputs

- Tab-delimited text file of epihaplotypes in BINARY format (file.out) from ampliMethProfiler
- Tab-delimited CpG-position file describing CpG positions in the sequence
- Sample metadata file with columns #SampleID, Tissue, Description, Group, Rep, ID
- Significant two-mCpG combinations annotated by MethCores_Extractor (Module 1)

## Outputs

- Clonality index per methylated core per population (core frequency normalized to average population methylation)
- Entanglement/stability index E per CpG in each core (in-phase CpGs index)
- ComposCore table of core structures and frequencies
- Tab_Epialleles table of epiallele structures and frequencies

## How to apply

Run MethCoresProfiler Modules 1 and 2 (MethCores_Extractor and MethCores_Combinator) on the binary epihaplotype matrix, CpG-position file and sample metadata, first performing repeated iterative sampling (default y = 1000) of each sample at the average read depth b4 to reduce size-comparison errors. Two mCpGs are annotated as significant when their observed frequency exceeds the expected independent-event frequency by chi-square test with P-value ≤ 10−9, and higher-order cores are extracted from epialleles containing all significant combinations, with the minimum core defined as at least two methylated cytosines with adjusted P ≤ 10−9. The clonality index is the frequency of the methylated core normalized to the average methylation of the population. The entanglement/stability index E measures the rate of coupling or association of at least two mCpGs (mCpG1 and mCpG2) in the core, computed as each mCpG's frequency within the core normalized to its population frequency, starting from the 5′-most CpG pair. Verify outputs by reproducing the ComposCore and Tab_Epialleles tables and comparing reconstructed clonality and E values against reference MethCoresProfiler runs.

## Related tools

- **MethCoresProfiler** (R tool whose Modules 1 and 2 extract significant phased mCpG combinations (cores) from which the clonality index and entanglement/stability index E are computed) — https://github.com/84AP/MethCoresProfiler/
- **ampliMethProfiler** (Upstream tool that extracts mCpG configurations in single DNA molecules to produce the epihaplotype matrix input) — https://sourceforge.net/projects/amplimethprofiler/
- **R** (Language in which MethCoresProfiler modules and the index computations are implemented) — https://github.com/84AP/MethCoresProfiler/

## Examples

```
Rscript Run_MethCoresProfiler_setup.R  # with MethCoresProfiler="/path-to-MethCoresProfiler-master/", Data="testData/", input="out/"
```

## Evaluation signals

- Reconstructed clonality index matches the reference MethCoresProfiler output (core frequency normalized to average population methylation)
- Entanglement index E values reproduce the expected coupling rates of mCpG1–mCpG2 pairs starting from the 5′-most CpG pair
- All annotated cores satisfy the minimum definition: at least two methylated cytosines with statistically significant frequency (chi-square adjusted P-value ≤ 10−9)
- ComposCore and Tab_Epialleles tables from Module 2 match reference structures and frequencies for the same input files
- Behavioral check: 10% amplified epialleles in Cond 3 and Cond 4 escape detection by Modules 1 and 2 while Module 3 identifies them

## Limitations

- Iterative sampling uses default y = 1000 iterations at depth b4; different depth regimes change the size-comparison error reduction
- Index computation depends on the significance threshold P ≤ 10−9, so weakly coupled mCpGs are excluded from cores and have no E value
- If no significant mCpG combinations are found, all epialleles pass through Module 2 and index coverage is limited to cores annotated downstream

## Evidence
