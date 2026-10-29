---
name: atac-seq-replicate-sample-sheet-configuration
description: Use when setting up a bulk ATAC-seq differential accessibility comparison
  from perturbation (e.g., cKO-vs-WT) experiments with ≥2 replicates per condition
  and processed BAM plus MACS2 narrowPeak files available.
when_to_use_negative:
- Single-replicate per condition designs — minMembers = 2 and DESeq2 dispersion estimation
  require at least 2 members per contrast group
- Single-cell ATAC-seq data, where cell-type-specific accessible regions are instead
  generated with tools like SnapATAC2 or ArchR
- Data with no per-replicate peak files — the samplesheet requires MACS2 narrowPeak
  files alongside BAM files
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0607
tools:
- name: DiffBind
  role: Consumes the replicate samplesheet to build a DBA object, count reads over
    consensus peaks, and set up the WT-vs-cKO contrast
- name: MACS2
  role: Produces the narrowPeak files referenced by each sample row in the samplesheet
- name: DESeq2
  role: Underlying normalization and differential analysis engine that requires the
    replicate structure defined in the samplesheet
- name: R
  role: Language in which the samplesheet-driven DiffBind workflow is executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-60269-4
    title: 'BIT: Bayesian Identification of Transcriptional regulators from epigenomics-based
      query region sets'
  source_content_hash: sha256:5d8227cd99a0cc03ca324714db2fd9f8864ec1c0f91020d801f5a535151f9bbd
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ZeyuL01/BIT
  tools:
  - BIT
  - DESeq2
  - DiffBind
  - MACS2
  - R
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0607
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# ATAC-seq replicate sample sheet configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assembles a samplesheet of biological replicate ATAC-seq samples (e.g., 2 WT and 2 cKO replicates) with pointers to processed BAM files and MACS2 narrowPeak files, so that differential accessibility analysis tools like DiffBind can build a contrast-ready object carrying the perturbed-TR ground truth.

## When to use

Use when setting up a bulk ATAC-seq differential accessibility comparison from perturbation (e.g., cKO-vs-WT) experiments with ≥2 replicates per condition and processed BAM plus MACS2 narrowPeak files available.

## When NOT to use

- Single-replicate per condition designs — minMembers = 2 and DESeq2 dispersion estimation require at least 2 members per contrast group
- Single-cell ATAC-seq data, where cell-type-specific accessible regions are instead generated with tools like SnapATAC2 or ArchR
- Data with no per-replicate peak files — the samplesheet requires MACS2 narrowPeak files alongside BAM files

## Inputs

- Processed ATAC-seq BAM files (one per replicate)
- MACS2 narrowPeak files called per replicate (q-value 0.01 threshold)
- Sample metadata: condition labels (WT vs cKO), replicate identifiers, mm10 genome build

## Outputs

- Four-sample samplesheet (samplesheet CSV) with condition and file-path columns
- DiffBind object (e.g., RUNX1_KO) configured for the cKO-vs-WT contrast

## How to apply

Create a four-sample samplesheet (2 WT replicates and 2 cKO replicates) with columns identifying the sample condition (DBA_CONDITION) and file paths to the processed BAM files and MACS2 narrowPeak files. Feed this samplesheet into dba() to build a DiffBind object, then count reads over consensus peaks with dba.count using minOverlap = 2 and remove ENCODE blacklist regions with dba.blacklist (DBA_BLACKLIST_MM10 for mm10). Define the cKO-vs-WT contrast with dba.contrast using DBA_CONDITION and minMembers = 2, ensuring each condition has at least the minimum replicate membership for DESeq2 differential analysis. The replicate structure is what lets downstream normalization (dba.normalize with DESeq2 native normalization, DBA_NORM_NATIVE, DBA_LIBSIZE_BACKGROUND, background = TRUE) and the DBA_DESeq2 analysis estimate dispersion across biological replicates rather than treating samples as singletons.

## Related tools

- **DiffBind** (Consumes the replicate samplesheet to build a DBA object, count reads over consensus peaks, and set up the WT-vs-cKO contrast)
- **MACS2** (Produces the narrowPeak files referenced by each sample row in the samplesheet)
- **DESeq2** (Underlying normalization and differential analysis engine that requires the replicate structure defined in the samplesheet)
- **R** (Language in which the samplesheet-driven DiffBind workflow is executed)

## Examples

```
RUNX1_KO <- dba(sampleSheet='samplesheet.csv', config=data.frame(Analysis='DBA_DESEQ2'))
```

## Evaluation signals

- The DiffBind object must contain 4 samples (2 WT and 2 cKO replicates) with DBA_CONDITION metadata populated for the contrast
- After dba.contrast with minMembers = 2, the contrast must be defined as cKO-vs-WT with both groups meeting the minimum membership
- Each samplesheet row must resolve to an existing BAM file and a narrowPeak file before dba.count is run
- Downstream dba.report output must include Fold, p.value, and FDR columns, confirming the replicate-backed contrast was analyzed successfully

## Limitations

- BIT's downstream performance depends on NGS data quality; sequencing biases such as GC content bias and PCR amplification bias can reduce the reliability of the epigenomic region sets ultimately derived from these samples
- Biological complexities of TR regulation, including dynamic TR binding across cell types and time, can reduce the performance of analyses built on this perturbation design

## Evidence

- [methods] Build a DiffBind object from the four-sample samplesheet (2 WT replicates and 2 cKO replicates) pointing to the processed BAM files and MACS2 narrowPeak files.: [evidence span withheld — anchor status: not_found]
- [methods] macs2 callpeak command generating narrowPeak inputs: "macs2 callpeak -f BAM -g mm -n "${PEAK_DIR}${MIDDLE_DIR}${ATAC_NAME_1}" -B -q 0.01"
- [intro] The R package of BIT is available on GitHub: "The R package of BIT is available on GitHub (https://github.com/ZeyuL01/BIT) along with a detailed manual."
- [methods] Replicate-based normalization parameters: [evidence span withheld — anchor status: not_found]
