---
name: unix-text-processing-pipelining
description: Use when a workflow task over plain-text data (e.g. gzipped FASTQ read
  headers and sequence subsets) can be expressed as chained basic UNIX tools inside
  a Watchdog XML workflow, and resource-light local execution is sufficient.
when_to_use_negative:
- The task requires real alignment or statistical analysis (e.g. Workflow 2's differential
  gene expression needs bwa and ContextMap2 modules, not UNIX text tools)
- Input data are not plain-text/gzip files amenable to line-oriented UNIX utilities
  (e.g. binary image or proprietary formats)
- You need interactive GUI-based module chaining rather than a pre-defined XML workflow
tools:
- name: Watchdog
  role: Workflow management system that schedules, parallelizes, error-checks and
    success-checks the chained UNIX text-processing tasks defined in the XML workflow
  repo: https://github.com/klugem/watchdog
- name: Java
  role: 'Runtime platform: Watchdog jar files are built for Java 11 and invoked internally
    via the watchdog.sh bash script'
  repo: https://github.com/klugem/watchdog
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-018-2107-4
    title: Watchdog – a workflow management system for the distributed analysis of
      large-scale experimental data
  source_content_hash: sha256:1b694772abcbd0a1946a2eb625ff1eb3f8ddc6dab0beba15608209e24e4ea8cd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/klugem/watchdog
  tools:
  - Java
  - Watchdog
  - XML
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# unix-text-processing-pipelining

## Summary

Encapsulates chained basic UNIX text utilities (gunzip/gzip decompression and recompression, header and sequence extraction, merging) as Watchdog workflow tasks, exemplified by the shipped Workflow 1 for basic information extraction from FASTQ files.

## When to use

Use when a workflow task over plain-text data (e.g. gzipped FASTQ read headers and sequence subsets) can be expressed as chained basic UNIX tools inside a Watchdog XML workflow, and resource-light local execution is sufficient.

## When NOT to use

- The task requires real alignment or statistical analysis (e.g. Workflow 2's differential gene expression needs bwa and ContextMap2 modules, not UNIX text tools)
- Input data are not plain-text/gzip files amenable to line-oriented UNIX utilities (e.g. binary image or proprietary formats)
- You need interactive GUI-based module chaining rather than a pre-defined XML workflow

## Inputs

- gzipped FASTQ files from the bundled HSV-1-derived test dataset
- Watchdog workflow definition XML (workflow1_basic_information_extraction.xml)
- Watchdog installation with Java 11 jar files (or Maven-built jars)

## Outputs

- decompressed FASTQ extracts containing read headers and a limited number of sequences
- merged compressed information files in the workflow results directory

## How to apply

Configure the shipped examples with ./helper_scripts/configureExamples.sh -i /path/to/install/folder/of/watchdog so workflow1_basic_information_extraction.xml lands in the examples directory. The workflow decompresses FASTQ inputs with gunzip, runs parallel tasks that extract read headers and a limited number of sequences using basic UNIX tools, then compresses and merges the per-task outputs. Write the pipeline as module tasks in the XML rather than a monolithic shell script so Watchdog handles dependencies, parallelism, error detection and success checkers. Launch from the installation directory with ./watchdog.sh -x path2/xml-file.xml instead of java -jar, because calling the jar directly forwards CTRL+C signals to child processes and can break locally running tasks. Judge success by all tasks finishing without triggering Watchdog's error detection and by the existence of the merged compressed information files under the 'Workflow 1 - Basic Information Extraction' results; the bundled HSV-1-derived test data runs in under one minute using about 50MB of storage.

## Related tools

- **Watchdog** (Workflow management system that schedules, parallelizes, error-checks and success-checks the chained UNIX text-processing tasks defined in the XML workflow) — https://github.com/klugem/watchdog
- **Java** (Runtime platform: Watchdog jar files are built for Java 11 and invoked internally via the watchdog.sh bash script) — https://github.com/klugem/watchdog

## Examples

```
./watchdog.sh -x examples/workflow1_basic_information_extraction.xml
```

## Evaluation signals

- All workflow tasks finish without triggering Watchdog's customizable error detection
- The workflow's success checkers report success for every task
- Merged compressed information files exist in the 'Workflow 1 - Basic Information Extraction' results directory
- Runtime stays under one minute and storage footprint around 50MB on a modern desktop for the bundled HSV-1-derived test dataset
- The workflow was launched via ./watchdog.sh -x so CTRL+C signals are not forwarded to child processes

## Limitations

- Calling java -jar jars/watchdog.jar directly forwards CTRL+C to all child processes, which might cause errors on locally running tasks
- The workflow only performs basic information extraction with basic UNIX tools; heavier analyses require additional configured modules (e.g. bwa, ContextMap2)
- Requires JDK 11 or higher (JavaFX SDK 11+ for GUI-related tools), which must be installed or bundled

## Evidence

- [readme] workflow1_basic_information_extraction.xml: "workflow1_basic_information_extraction.xml - simple workflow that extracts information from files using basic UNIX tools"
- [readme] CTRL+C forwarding caveat: "To call the jar file directly is not recommended as CTRL+C signal will be forwarded to all child processes, which might cause errors on locally running tasks."
- [abstract] Java implementation: "Watchdog is implemented in Java and thus platform-independent"
