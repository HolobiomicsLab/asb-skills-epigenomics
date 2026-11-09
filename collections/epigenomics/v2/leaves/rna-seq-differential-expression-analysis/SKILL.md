---
name: rna-seq-differential-expression-analysis
description: Use when FASTQ RNA-seq reads (with replicates across conditions) must
  be turned into per-method differential expression result tables and the Watchdog
  WMS is available, with bwa and ContextMap2 installed and configured.
when_to_use_negative:
- 'Input is already a gene count table produced by another quantifier and only the
  DE testing step (DETest: limma/edgeR/DESeq/DESeq2) is needed without the mapping/quantification
  modules'
- The analysis targets non-RNA-seq data types; the shipped workflow is specific to
  RNA-seq experiments
- bwa and ContextMap2 are not installed and configured, since the workflow prerequisites
  cannot be satisfied
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Watchdog
  role: Workflow management system that executes the shipped differential gene expression
    workflow via the command-line tool (watchdog.sh -x xml-file)
  repo: https://github.com/klugem/watchdog
- name: Java
  role: Runtime requirement for Watchdog; the distributed jar files are built for
    Java 11 and invoked by the bash scripts
  repo: https://github.com/klugem/watchdog
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-018-2107-4
    title: Watchdog – a workflow management system for the distributed analysis of
      large-scale experimental data
  source_content_hash: sha256:dc520c2bfd2acfb3057da96b0a9dab557e325bd62926acd4b1b9b8cea35972ce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/klugem/watchdog
  tools:
  - Java
  - Watchdog
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# rna-seq-differential-expression-analysis

## Summary

A Watchdog-shipped workflow that performs differential gene expression analysis on RNA-seq data by quantifying gene-level read counts with featureCounts and running the DETest module, which applies limma, edgeR, DESeq, and DESeq2 to the count table.

## When to use

Use when FASTQ RNA-seq reads (with replicates across conditions) must be turned into per-method differential expression result tables and the Watchdog WMS is available, with bwa and ContextMap2 installed and configured.

## When NOT to use

- Input is already a gene count table produced by another quantifier and only the DE testing step (DETest: limma/edgeR/DESeq/DESeq2) is needed without the mapping/quantification modules
- The analysis targets non-RNA-seq data types; the shipped workflow is specific to RNA-seq experiments
- bwa and ContextMap2 are not installed and configured, since the workflow prerequisites cannot be satisfied

## Inputs

- RNA-seq FASTQ read files (replicated across conditions)
- Reference genome for read mapping (bwa and ContextMap2 required and configured)
- Gene annotation for featureCounts quantification

## Outputs

- Gene count table from the featureCounts module
- Per-method differential expression result tables (limma, edgeR, DESeq, DESeq2) from the DETest module
- QC reports (FastQC-style Watchdog QC and RSeQC mapping QC)
- Optional workflow execution report via reportGenerator.jar

## How to apply

Configure the shipped example workflows with ./helper_scripts/configureExamples.sh -i /path/to/install/folder/of/watchdog, then launch workflow2_differential_gene_expression.xml via ./watchdog.sh -x path/to/workflow2_differential_gene_expression.xml. The workflow runs Watchdog QC (FastQC-style read quality checks), maps reads against the reference genome with the ContextMap 2 module, converts and indexes alignments with samtools, quantifies gene expression with featureCounts, and runs mapping QC with RSeQC. The DETest module then evaluates limma, edgeR, DESeq, and DESeq2 on the resulting gene count table, providing per-method differential expression results. Optionally summarize execution with reportGenerator.jar. Success is judged by all steps completing without Watchdog error-detection triggers and by production of the count table plus per-method DE result tables in the workflow output folder.

## Related tools

- **Watchdog** (Workflow management system that executes the shipped differential gene expression workflow via the command-line tool (watchdog.sh -x xml-file)) — https://github.com/klugem/watchdog
- **Java** (Runtime requirement for Watchdog; the distributed jar files are built for Java 11 and invoked by the bash scripts) — https://github.com/klugem/watchdog

## Examples

```
./helper_scripts/configureExamples.sh -i /path/to/install/folder/of/watchdog && ./watchdog.sh -x path/to/workflow2_differential_gene_expression.xml
```

## Evaluation signals

- All workflow steps complete without triggering Watchdog's customizable error-detection system
- A gene count table file is produced in the workflow output folder after featureCounts
- Separate differential expression result tables for each of limma, edgeR, DESeq, and DESeq2 appear in the workflow output folder
- QC reports from the Watchdog QC module and RSeQC module are generated
- reportGenerator.jar can summarize the steps performed during workflow execution

## Limitations

- The differential gene expression example workflow needs bwa and ContextMap2 to be installed and configured as prerequisites
- Watchdog requires JDK 11 or higher (JavaFX SDK 11+ for the GUI/moduleMaker), and running watchdog.jar directly is discouraged because CTRL+C signals are forwarded to child processes, which might cause errors on locally running tasks
- Modules for this workflow ship with the software as a case study; additional shared modules/workflows live in the watchdog-wms community repositories and may carry individual licenses

## Evidence

- [intro] modules for read quality checks, read mapping, gene expression quantification and differential gene expression analysis were implemented: "As a case study on how Watchdog can be applied, modules for read quality checks, read mapping, gene expression quantification and differential gene expression analysis were implemented"
- [readme] workflow2_differential_gene_expression.xml description: "workflow2_differential_gene_expression.xml - workflow performing a differential gene expression analysis on an example data set (needs bwa and ContextMap2 to be installed and configured)"
- [other] DETest module methods: [evidence span withheld — anchor status: not_found]
- [readme] command-line execution: "In order to run a xml-file using the command line tool, call

`./watchdog.sh -x xml-file`"
