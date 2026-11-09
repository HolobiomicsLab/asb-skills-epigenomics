---
name: fastq-header-sequence-extraction
description: Use when you need to run or reproduce a minimal Watchdog workflow that
  extracts read headers and sequence subsets from compressed FASTQ files, or when
  benchmarking Watchdog's resource footprint on a small test dataset.
when_to_use_negative:
- Input files are not FASTQ or are already uncompressed plain text, since the workflow
  assumes gunzip-decompressible FASTQ archives
- You need a full RNA-seq analysis (quality checks, mapping with bwa/ContextMap2,
  differential gene expression) — use workflow2_differential_gene_expression.xml or
  the RNA-seq workflow instead
- Tasks must survive interactive interruptions if launched via java -jar directly;
  avoid bypassing ./watchdog.sh when locally running child processes matter
edam_operation: http://edamontology.org/operation_0233
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: Watchdog
  role: Workflow management system that defines and executes the module tasks (gunzip,
    header/sequence extraction, compression, merging) of the basic information extraction
    workflow
  repo: https://github.com/klugem/watchdog
- name: Java
  role: Runtime environment (JDK 11 or higher) required to run the Watchdog jar files
    via ./watchdog.sh
- name: XML
  role: Syntax in which the workflow (workflow1_basic_information_extraction.xml)
    is defined and validated
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-018-2107-4
    title: Watchdog – a workflow management system for the distributed analysis of
      large-scale experimental data
  source_content_hash: sha256:48a3fd9360beb1b2f132aed38f7d3afa23c501ea3ddd03227d47bfb3980643f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/klugem/watchdog
  tools:
  - Java
  - Watchdog
  - XML
  edam_operation: http://edamontology.org/operation_0233
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# fastq-header-sequence-extraction

## Summary

A Watchdog workflow pattern (workflow1_basic_information_extraction.xml) that extracts basic information — read headers and a limited number of sequences — from gzipped FASTQ files using basic UNIX tools (gunzip, grep-like extraction, gzip), executed as parallel module tasks followed by merging. It demonstrates how Watchdog orchestrates simple shell-based tasks on bundled HSV-1-derived test data.

## When to use

Use when you need to run or reproduce a minimal Watchdog workflow that extracts read headers and sequence subsets from compressed FASTQ files, or when benchmarking Watchdog's resource footprint on a small test dataset.

## When NOT to use

- Input files are not FASTQ or are already uncompressed plain text, since the workflow assumes gunzip-decompressible FASTQ archives
- You need a full RNA-seq analysis (quality checks, mapping with bwa/ContextMap2, differential gene expression) — use workflow2_differential_gene_expression.xml or the RNA-seq workflow instead
- Tasks must survive interactive interruptions if launched via java -jar directly; avoid bypassing ./watchdog.sh when locally running child processes matter

## Inputs

- Gzipped FASTQ files (bundled HSV-1-derived test dataset)
- Watchdog workflow XML definition (workflow1_basic_information_extraction.xml)
- Watchdog installation with Java 11 (and JavaFX 11+ for GUI components)

## Outputs

- Extracted read-header files per FASTQ input
- Limited sequence extracts per FASTQ input
- Merged, gzip-compressed information files under the 'Workflow 1 - Basic Information Extraction' results directory

## How to apply

Obtain Watchdog either by building from source with Maven (which downloads dependencies into jars/libs and rebuilds the jar files) or by downloading the Java 11 jar release. Configure the shipped example workflows with ./helper_scripts/configureExamples.sh -i /path/to/install/folder/of/watchdog, which places them under the examples directory. Inspect workflow1_basic_information_extraction.xml: it decompresses FASTQ inputs with gunzip, extracts read headers and a limited number of sequences in parallel module tasks, then recompresses and merges the results. Launch the workflow from the installation directory with ./watchdog.sh -x path2/xml-file.xml rather than calling java -jar directly, because the bash script prevents CTRL+C signals from being forwarded to child processes running local tasks. Verify completion via the workflow's success checkers and confirm the merged compressed output files exist in the results folder 'Workflow 1 - Basic Information Extraction'; on the bundled HSV-1-derived test dataset this requires about 50 MB storage and under one minute on a modern desktop.

## Related tools

- **Watchdog** (Workflow management system that defines and executes the module tasks (gunzip, header/sequence extraction, compression, merging) of the basic information extraction workflow) — https://github.com/klugem/watchdog
- **Java** (Runtime environment (JDK 11 or higher) required to run the Watchdog jar files via ./watchdog.sh)
- **XML** (Syntax in which the workflow (workflow1_basic_information_extraction.xml) is defined and validated)

## Examples

```
./helper_scripts/configureExamples.sh -i /path/to/install/folder/of/watchdog && ./watchdog.sh -x /path/to/install/folder/of/watchdog/examples/workflow1_basic_information_extraction.xml
```

## Evaluation signals

- All workflow tasks finish without triggering Watchdog's customizable error detection
- The workflow's success checkers report success for every module task
- Merged compressed output files exist in the 'Workflow 1 - Basic Information Extraction' results directory
- Extracted header/sequence files contain only the expected read headers and the limited number of sequences per file
- Resource usage stays near the documented envelope: ~50 MB storage and under one minute on a modern desktop for the bundled test dataset

## Limitations

- Calling java -jar jars/watchdog.jar directly is not recommended: CTRL+C signals are forwarded to all child processes, which may cause errors on locally running tasks
- The bundled workflow2 example (unlike workflow1) needs bwa and ContextMap2 to be installed and configured; workflow1 relies only on basic UNIX tools
- The GUI workflow designer and moduleMaker additionally require the JavaFX SDK 11 or higher
- No specific Watchdog software version number is stated for the release in the article text

## Evidence

- [results] Running the basic-information-extraction example workflow requires about 50MB of storage and less than one minute on a modern desktop computer.: [evidence span withheld — anchor status: not_found]
- [readme] workflow1_basic_information_extraction.xml - simple workflow that extracts information from files using basic UNIX tools: "workflow1_basic_information_extraction.xml - simple workflow that extracts information from files using basic UNIX tools"
- [readme] To call the jar file directly is not recommended as CTRL+C signal will be forwarded to all child processes, which might cause errors on locally running tasks.: "To call the jar file directly is not recommended as CTRL+C signal will be forwarded to all child processes, which might cause errors on locally running tasks."
- [abstract] Execution of workflows is possible using either the GUI or a command-line interface: "Execution of workflows is possible using either the GUI or a command-line interface"
