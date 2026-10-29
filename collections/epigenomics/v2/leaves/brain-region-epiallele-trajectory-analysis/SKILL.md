---
name: brain-region-epiallele-trajectory-analysis
description: Use when bisulfite amplicon sequencing of phased CpG regions must resolve
  stable epiallele families and methylated cores in complex DNA populations, e.g.
  tracking mCpG signatures across brain regions or developmental time points.
when_to_use_negative:
- The question is single-CpG methylation quantification or abundance of single epialleles
  without phased combinations — tools like EBA, ampliMethProfiler or methclone are
  designed for that
- Input reads are not bisulfite-converted amplicon data with phased CpGs, so no binary
  epihaplotype table can be generated
- The population lacks detectable significant mCpG combinations, in which case Modules
  1 and 2 may miss rare amplified epialleles (as with the 10% amplified epialleles
  in Cond 3 and Cond 4)
edam_operation: http://edamontology.org/operation_3208
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3295
tools:
- name: PEAR
  role: Merges paired-end FASTQ reads with a minimum 40 nucleotide overlapping region
- name: PRINSEQ
  role: Converts quality-filtered merged reads to FASTA format
- name: ampliMethProfiler
  role: Extracts per-molecule mCpG configurations applying quality filters to produce
    binary epihaplotype tables
  repo: https://sourceforge.net/projects/amplimethprofiler/
- name: MethCoresProfiler
  role: Three R modules (Extractor, Combinator, Analyst) that annotate significant
    mCpG combinations, build methylated cores, and compute MethCores, clonality and
    entanglement indices
  repo: https://github.com/84AP/MethCoresProfiler/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqaa096
    title: Tracing and tracking epiallele families in complex DNA populations
  source_content_hash: sha256:8a98010e3c0035a07ffc9dd5b8dfdd199831e77cc0b1109793c75aa7b3397241
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/84AP/MethCoresProfiler
  tools:
  - MethCoresProfiler
  - PEAR
  - PRINSEQ
  - R
  - ampliMethProfiler
  edam_operation: http://edamontology.org/operation_3208
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

# brain-region-epiallele-trajectory-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A MethCoresProfiler-based workflow that extracts significant combinations of methylated phased CpGs (methylated cores), traces rare epiallele families, and computes MethCores, clonality and entanglement (E) indices across mouse brain areas (CB, CX, HIPP) during postnatal differentiation (P1–P60).

## When to use

Use when bisulfite amplicon sequencing of phased CpG regions must resolve stable epiallele families and methylated cores in complex DNA populations, e.g. tracking mCpG signatures across brain regions or developmental time points.

## When NOT to use

- The question is single-CpG methylation quantification or abundance of single epialleles without phased combinations — tools like EBA, ampliMethProfiler or methclone are designed for that
- Input reads are not bisulfite-converted amplicon data with phased CpGs, so no binary epihaplotype table can be generated
- The population lacks detectable significant mCpG combinations, in which case Modules 1 and 2 may miss rare amplified epialleles (as with the 10% amplified epialleles in Cond 3 and Cond 4)

## Inputs

- Paired-end FASTQ reads from bisulfite amplicon libraries (ENA accession PRJEB28662)
- Tab delimited text file of EpiHaplotypes in BINARY format (file.out) from ampliMethProfiler
- Tab delimited text file of CpG positions in the sequence (cgpos)
- Sample metadata file with columns #SampleID, Tissue, Description, Group, Rep, ID
- List of target regions (e.g. DDO and DAO amplicons)

## Outputs

- ComposCore table reporting the structure and frequency of significant methylated cores
- Tab_Epialleles table of individual significant epialleles
- Heatmaps of epiallele structure and frequency plus PCA of significant epialleles
- Per-sample MethCores index, Clonality index and Entanglement (E) index (e.g. Supplementary Table S3 values for CB, CX, HIPP across P1–P60)

## How to apply

Merge paired-end FASTQ reads from the bisulfite amplicon libraries (ENA PRJEB28662) with PEAR using a minimum 40 nt overlap, retain reads with mean Phred >33 and length 400–500 nt, and convert to FASTA with PRINSEQ. Extract per-molecule mCpG configurations with ampliMethProfiler applying its filters (length ±50% of reference, ≥80% primer similarity, ≥98% bisulfite efficiency, ≥60% base alignment, removal of reads with ambiguous CpG calls) to yield binary epihaplotype tables. Run MethCores_Extractor (Module 1) with iterative sampling (default y = 1000) at average depth to annotate significant two-mCpG combinations by chi-square independence testing (P ≤ 1e-9), then MethCores_Combinator (Module 2) to cross combinations into higher-order cores (chi-square, P ≤ 1e-9) and report ComposCore and Tab_Epialleles tables. Finally run MethCores_Analyst (Module 3) for hierarchical clustering, comparison against the random theoretical control (R), and extraction of the most frequent mCpGs shared by significant epialleles from a 0.9 decreasing percentage scale (minimum two mCpGs), computing the MethCores index, clonality index and entanglement (E) index per sample and time point. Rationale: single-nucleotide methylation techniques cannot resolve phased CpG strings because heterogeneous 5'-3' ends and stochastic mCpGs dilute stable epialleles, whereas cores mark families of epialleles derived from common ancestors.

## Related tools

- **PEAR** (Merges paired-end FASTQ reads with a minimum 40 nucleotide overlapping region)
- **PRINSEQ** (Converts quality-filtered merged reads to FASTA format)
- **ampliMethProfiler** (Extracts per-molecule mCpG configurations applying quality filters to produce binary epihaplotype tables) — https://sourceforge.net/projects/amplimethprofiler/
- **MethCoresProfiler** (Three R modules (Extractor, Combinator, Analyst) that annotate significant mCpG combinations, build methylated cores, and compute MethCores, clonality and entanglement indices) — https://github.com/84AP/MethCoresProfiler/

## Examples

```
Rscript Run_MethCoresProfiler_setup.R  # set MethCoresProfiler="/path-to-MethCoresProfiler-master/", Data="testData/", input="out/" before running
```

## Evaluation signals

- The stable '−336|−330' two-mCpG association is recovered in cerebellum (CB) and hippocampus (HIPP) across P1–P60
- Cortex (CX) shows the reported developmental switch from '−242|−175' to '−336|−318|−242'
- Computed E index values match those listed in Supplementary Table S3 for CB, CX and HIPP
- Module 1 output contains tetrachoric correlations, two-mCpG co-occurrence tables and Shannon-entropy summaries as documented in the README
- All annotated cores contain at least two methylated cytosines with chi-square adjusted P-value ≤ 1e-9

## Limitations

- Rare amplified epialleles (e.g. 10% in Cond 3 and Cond 4) escape detection by Modules 1 and 2
- EBA, ampliMethProfiler and methclone cannot distinguish the Cond 4 epiallele population from R1 and R2 controls; only MethCoresProfiler does
- If significant epialleles are not found by Module 3, only the highest-frequency epiallele is annotated as the stable signature/core
- Stable epialleles cannot be identified by current single-nucleotide techniques due to heterogeneity of 5'-3' ends and dilution by random unstable methylated CpGs

## Evidence
