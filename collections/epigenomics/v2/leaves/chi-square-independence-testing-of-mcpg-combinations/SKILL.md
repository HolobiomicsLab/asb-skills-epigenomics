---
name: chi-square-independence-testing-of-mcpg-combinations
description: Use when bisulfite-sequencing epihaplotype matrices (binary mCpG per
  molecule) must be mined for statistically significant co-methylated CpG combinations
  underlying stable epiallele families.
when_to_use_negative:
- The goal is quantifying methylation of single CpGs or abundance of individual epialleles
  without testing phased co-occurrence — tools like ampliMethProfiler, methclone or
  EBA are designed for that, and MethCoresProfiler's independence testing is unnecessary.
- Input data are not single-molecule (phased) bisulfite reads; unphased bulk methylation
  levels cannot support co-occurrence testing of mCpG pairs within molecules.
- 'Randomized/shuffled epihaplotype controls are the input: by construction mCpGs
  are independent, so all combinations should be non-significant and the test would
  flag nothing meaningful.'
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0622
tools:
- name: MethCoresProfiler
  role: Provides Modules 1 (MethCores_Extractor) and 2 (MethCores_Combinator), which
    run the chi-square independence tests on mCpG combinations and assemble significant
    cores
  repo: https://github.com/84AP/MethCoresProfiler/
- name: ampliMethProfiler
  role: Upstream preprocessing that extracts per-molecule mCpG configurations from
    bisulfite reads, producing the binary epihaplotype matrix the chi-square tests
    consume
  repo: https://sourceforge.net/projects/amplimethprofiler/
- name: R
  role: Language in which the modules and the chi-square independence statistics are
    implemented
  repo: https://github.com/84AP/MethCoresProfiler/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqaa096
    title: Tracing and tracking epiallele families in complex DNA populations
  source_content_hash: sha256:f63a9a2ddf1ee5b379f6e06259ed4181eb08d302ce342d4b4d1c4071f65d50ba
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
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# chi-square-independence-testing-of-mcpg-combinations

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Statistical core of MethCoresProfiler: identify which combinations of phased methylated CpGs (mCpGs) co-occur within single DNA molecules more frequently than expected under independence, using chi-square tests of independence at P ≤ 10−9, to define the 'methylated cores' shared by epiallele families.

## When to use

Use when bisulfite-sequencing epihaplotype matrices (binary mCpG per molecule) must be mined for statistically significant co-methylated CpG combinations underlying stable epiallele families.

## When NOT to use

- The goal is quantifying methylation of single CpGs or abundance of individual epialleles without testing phased co-occurrence — tools like ampliMethProfiler, methclone or EBA are designed for that, and MethCoresProfiler's independence testing is unnecessary.
- Input data are not single-molecule (phased) bisulfite reads; unphased bulk methylation levels cannot support co-occurrence testing of mCpG pairs within molecules.
- Randomized/shuffled epihaplotype controls are the input: by construction mCpGs are independent, so all combinations should be non-significant and the test would flag nothing meaningful.

## Inputs

- Tab-delimited text file of epihaplotypes in BINARY format (one row per DNA molecule, one column per CpG)
- Tab-delimited text file with CpG positions in the sequence
- Sample metadata file with columns #SampleID, Tissue, Description, Group, Rep, ID
- List of regions to analyze (required by MethCores_Combinator)
- Filtering parameters for the Combinator step

## Outputs

- Annotated table of significant two-mCpG combinations (chi-square independence test, P ≤ 10−9)
- Significant mCpG combinations of three or more CpGs from MethCores_Combinator
- ComposCore table: structure and frequency of significant mCpG combinations (cores)
- Tab_Epialleles table: epialleles containing all significant mCpG combinations
- Single-CpG methylation frequencies, tetrachoric correlations, two-mCpG co-occurrence table, and Shannon/Renyi entropy indices normalized to [0, 1]
- Clonality index and entanglement/stability index E per CpG in each core

## How to apply

Parse the tab-delimited binary epihaplotype matrix, CpG-position file and sample metadata, and first compute the average read depth (b4), performing repeated iterative sampling of each sample (default y = 1000 iterations at depth b4) to reduce size-comparison errors. In Module 1 (MethCores_Extractor), compare for every pair of mCpGs the observed frequency n(mCpG1–mCpG2)/N against the expected frequency for independent events p(mCpG1)×p(mCpG2) using a chi-square test of independence and annotate as significant only combinations with P-value ≤ 10−9; single-CpG frequencies, tetrachoric correlations, mCpG co-occurrence, and Shannon/Renyi entropies (Renyi weight α = 0.5, normalized to [0, 1]) are also reported. In Module 2 (MethCores_Combinator), cross all significant two-mCpG combinations, test combinations of three or more mCpGs against the expected independent-event frequency (chi-square, P ≤ 10−9), extract the epialleles containing all significant combinations, and report core structures and frequencies (ComposCore and Tab_Epialleles tables); the minimum core contains at least two methylated cytosines with statistically significant frequency. If no significant mCpG combinations are found, all epialleles are carried to the next step. Downstream indices (clonality index = core frequency normalized to average population methylation; entanglement/stability index E = ratio of each mCpG's frequency in the core normalized to its population frequency, starting from the 5′-most CpG pair) depend directly on the significance calls made here.

## Related tools

- **MethCoresProfiler** (Provides Modules 1 (MethCores_Extractor) and 2 (MethCores_Combinator), which run the chi-square independence tests on mCpG combinations and assemble significant cores) — https://github.com/84AP/MethCoresProfiler/
- **ampliMethProfiler** (Upstream preprocessing that extracts per-molecule mCpG configurations from bisulfite reads, producing the binary epihaplotype matrix the chi-square tests consume) — https://sourceforge.net/projects/amplimethprofiler/
- **R** (Language in which the modules and the chi-square independence statistics are implemented) — https://github.com/84AP/MethCoresProfiler/

## Examples

```
source("Run_MethCoresProfiler_setup.R")  # sets MethCoresProfiler path and Data="testData/", input="out/", then runs 1_MethCores_Extractor.R and 2_MethCores_Combinator.R
```

## Evaluation signals

- Re-running the original MethCoresProfiler Modules 1 and 2 on the same input files should reproduce the same significant mCpG combinations, core structures, and clonality/E index values.
- Every annotated combination must satisfy the chi-square test of independence with P-value ≤ 10−9; combinations at conventional thresholds (e.g. 0.05) are incorrectly permissive.
- Reported cores must contain at least two methylated cytosines with statistically significant frequency; any single-CpG 'core' is a schema violation.
- In validation datasets, the 10% amplified epialleles in Cond 3 and Cond 4 escape detection by Modules 1 and 2 while Module 3 (MethCores_Analyst) identifies them — a known negative control for Modules 1–2 sensitivity.
- The random control (R) generated by Module 1 with the same number of CpGs and depth should yield no significant mCpG combinations, confirming the test rejects independent co-occurrence.

## Limitations

- The very strict P ≤ 10−9 threshold means low-frequency or low-depth mCpG combinations escape detection by Modules 1 and 2 (e.g. the 10% amplified epialleles in Cond 3 and Cond 4), and only Module 3 can recover them.
- MethCoresProfiler distinguishes Cond 4 from R1 and R2 epialleles whereas EBA, ampliMethProfiler and methclone do not; results depend on having phased single-molecule data, not aggregate methylation.
- Repeated iterative sampling (y = 1000) at depth b4 is required to reduce size-comparison errors when comparing samples of different depths; skipping it biases frequency comparisons.
- In the absence of significant mCpG combinations, all epialleles are carried forward, so downstream indices may be computed on unfiltered, statistically unsupported combinations.

## Evidence
