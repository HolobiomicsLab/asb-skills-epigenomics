---
name: reference-genome-read-mapping
description: Use when raw sequencing reads (e.g. RNA-seq FASTQ input) must be aligned
  to a reference genome as an early step of a Watchdog workflow before feature counting
  or differential expression testing.
when_to_use_negative:
- Input is already a gene count table — proceed directly to the DETest differential
  expression module instead of mapping
- Reads do not need genome-level alignment (e.g. only quality control is required
  — use the Watchdog QC module alone)
- bwa and ContextMap2 are not installed and cannot be configured as workflow prerequisites
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3170
tools:
- name: Watchdog
  role: Workflow management system that orchestrates the read mapping modules and
    monitors task success/error during execution
  repo: https://github.com/klugem/watchdog
- name: ContextMap 2
  role: Read mapping module used to align reads against the reference genome in the
    differential gene expression workflow
- name: bwa
  role: Required mapping prerequisite that must be installed and configured for the
    example workflow
- name: samtools
  role: Performs SAM-to-BAM conversion and indexing of mapped reads after mapping
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-018-2107-4
    title: Watchdog – a workflow management system for the distributed analysis of
      large-scale experimental data
  source_content_hash: sha256:d811b7e03376117ad6a15b0b07079a69760631dc90edd286cfa8bd1d94bb7cb4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/klugem/watchdog
  tools:
  - ContextMap 2
  - Watchdog
  - bwa
  - samtools
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# reference-genome-read-mapping

## Summary

Maps sequencing reads against a reference genome within a Watchdog workflow, using the bwa and ContextMap 2 modules followed by SAM-to-BAM conversion and indexing with samtools as a precursor to gene-level quantification and QC.

## When to use

Use when raw sequencing reads (e.g. RNA-seq FASTQ input) must be aligned to a reference genome as an early step of a Watchdog workflow before feature counting or differential expression testing.

## When NOT to use

- Input is already a gene count table — proceed directly to the DETest differential expression module instead of mapping
- Reads do not need genome-level alignment (e.g. only quality control is required — use the Watchdog QC module alone)
- bwa and ContextMap2 are not installed and cannot be configured as workflow prerequisites

## Inputs

- Raw sequencing reads (e.g. RNA-seq input reads referenced in the workflow XML)
- Reference genome (required by the bwa and ContextMap2 modules)
- Watchdog workflow XML file specifying mapping modules

## Outputs

- Mapped reads (SAM converted to indexed BAM via samtools)
- Mapping QC reports (via RSeQC module)
- Per-task execution reports (optionally via reportGenerator.jar)

## How to apply

Configure the shipped example workflows with Watchdog via ./helper_scripts/configureExamples.sh, ensuring that bwa and ContextMap2 are installed and configured as workflow prerequisites. After initial read quality checks with the Watchdog QC module, map the reads against the reference genome using the ContextMap 2 module, then convert the SAM output to BAM and index it with samtools so downstream modules can consume it. The mapping step feeds directly into gene expression quantification with featureCounts and mapping QC with RSeQC, so verify mapper settings are consistent with the count table gene annotations. Run the workflow as XML with ./watchdog.sh -x path/to/workflow2_differential_gene_expression.xml and rely on Watchdog's customizable error detection to flag failing mapping tasks.

## Related tools

- **Watchdog** (Workflow management system that orchestrates the read mapping modules and monitors task success/error during execution) — https://github.com/klugem/watchdog
- **ContextMap 2** (Read mapping module used to align reads against the reference genome in the differential gene expression workflow)
- **bwa** (Required mapping prerequisite that must be installed and configured for the example workflow)
- **samtools** (Performs SAM-to-BAM conversion and indexing of mapped reads after mapping)

## Examples

```
./watchdog.sh -x path/to/workflow2_differential_gene_expression.xml
```

## Evaluation signals

- All mapping and SAM-to-BAM tasks complete without Watchdog error-detection triggers firing
- Indexed BAM files are produced in the workflow output folder as expected by downstream modules
- RSeQC mapping QC reports are generated and available alongside the mapping outputs
- Downstream featureCounts module successfully produces a gene count table, indicating the BAM input was valid
- Workflow prerequisites bwa and ContextMap2 were detected as installed and configured before launch

## Limitations

- The differential gene expression example workflow requires bwa and ContextMap2 to be installed and configured beforehand; the workflow fails or cannot run otherwise
- Watchdog itself imposes no restriction on which programs can be included as modules, but each external mapper must be configured as a workflow prerequisite
- No specific version number is stated for the Watchdog software release in the article

## Evidence

- [methods] ContextMap2 mapping step: [evidence span withheld — anchor status: not_found]
- [readme] workflow prerequisites: "workflow2_differential_gene_expression.xml - workflow performing a differential gene expression analysis on an example data set (needs bwa and ContextMap2 to be installed and configured)"
- [intro] module scope: "As a case study on how Watchdog can be applied, modules for read quality checks, read mapping, gene expression quantification and differential gene expression analysis were implemented"
- [abstract] error detection: "Main features include straightforward processing of replicate data, support for distributed computer systems, customizable error detection and manual intervention into workflow execution."
