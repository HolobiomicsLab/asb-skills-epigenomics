---
name: gzip-compressed-fastq-handling
description: Use when a workflow must extract information from gzip-compressed FASTQ
  files using basic UNIX tools — e.g., decompress FASTQ, pull read headers and a few
  sequences per file, then recompress and merge results.
when_to_use_negative:
- Running Workflow 2 (differential gene expression analysis), which needs bwa and
  ContextMap2 to be installed and configured rather than basic UNIX tools
- Files are already uncompressed FASTQ and no decompression/recompression step is
  needed
- Tasks require mapping, quality scoring, or gene quantification — these belong to
  Watchdog's RNA-seq modules, not the basic information extraction workflow
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0769
- http://edamontology.org/topic_3307
tools:
- name: Watchdog
  role: Workflow management system that executes the XML workflow performing gunzip
    decompression, parallel sequence/header extraction, and recompression/merging
    with error detection
  repo: https://github.com/klugem/watchdog
- name: Java
  role: Runtime required to run Watchdog; the distributed jar files are built for
    Java 11 and called internally by bash scripts
- name: XML
  role: Format of the workflow definition file (workflow1_basic_information_extraction.xml)
    that specifies the decompression/extraction/compression tasks
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-018-2107-4
    title: Watchdog – a workflow management system for the distributed analysis of
      large-scale experimental data
  source_content_hash: sha256:8ce9ee68a672b8c906db71dbfcb3f1cffd2052d12c51d0a9d51fe57751c45989
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/klugem/watchdog
  tools:
  - Java
  - Watchdog
  - XML
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0769
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# gzip-compressed-fastq-handling

## Summary

Decompresses gzip-compressed FASTQ files with basic UNIX tools inside a Watchdog workflow, extracts information such as read headers and a limited number of sequences, then recompresses and merges the extracted files. This is demonstrated by Watchdog's shipped Workflow 1 (basic information extraction) and requires only ~50 MB storage and under a minute of runtime on a modern desktop.

## When to use

Use when a workflow must extract information from gzip-compressed FASTQ files using basic UNIX tools — e.g., decompress FASTQ, pull read headers and a few sequences per file, then recompress and merge results.

## When NOT to use

- Running Workflow 2 (differential gene expression analysis), which needs bwa and ContextMap2 to be installed and configured rather than basic UNIX tools
- Files are already uncompressed FASTQ and no decompression/recompression step is needed
- Tasks require mapping, quality scoring, or gene quantification — these belong to Watchdog's RNA-seq modules, not the basic information extraction workflow

## Inputs

- gzip-compressed FASTQ files (HSV-1-derived bundled test dataset)
- Workflow XML definition (workflow1_basic_information_extraction.xml)
- Watchdog installation folder with jars/ and modules

## Outputs

- Decompressed FASTQ extracts (read headers plus a limited number of sequences)
- Merged, recompressed (gzip) information files under the workflow results directory

## How to apply

Install Watchdog (build from source with Maven so dependencies land in jars/libs, or use the Java 11 jar files) and configure the shipped examples with ./helper_scripts/configureExamples.sh -i /path/to/install/folder/of/watchdog. Inspect workflow1_basic_information_extraction.xml, which uses basic UNIX tools: gunzip decompression of the FASTQ inputs, parallel extraction of read headers and a limited number of sequences from each decompressed file, followed by recompression and merging of the per-file outputs. Launch the workflow from the installation directory via ./watchdog.sh -x path2/xml-file.xml rather than calling java -jar directly, so CTRL+C signals are not forwarded to child processes running local tasks. Rely on Watchdog's customizable success/error checkers to verify each task completed, then collect the merged compressed information files from the 'Workflow 1 - Basic Information Extraction' results directory.

## Related tools

- **Watchdog** (Workflow management system that executes the XML workflow performing gunzip decompression, parallel sequence/header extraction, and recompression/merging with error detection) — https://github.com/klugem/watchdog
- **Java** (Runtime required to run Watchdog; the distributed jar files are built for Java 11 and called internally by bash scripts)
- **XML** (Format of the workflow definition file (workflow1_basic_information_extraction.xml) that specifies the decompression/extraction/compression tasks)

## Examples

```
./helper_scripts/configureExamples.sh -i /path/to/install/folder/of/watchdog; ./watchdog.sh -x /path/to/install/folder/of/watchdog/examples/workflow1_basic_information_extraction.xml
```

## Evaluation signals

- All workflow tasks finish without triggering Watchdog's customizable error detection
- Merged compressed information files exist in the workflow's results directory ('Workflow 1 - Basic Information Extraction')
- Decompressed FASTQ extracts contain read headers and the expected limited number of sequences per input file
- Total resource footprint matches the documented envelope (~50 MB storage, less than one minute on a modern desktop computer)
- Workflow completes when launched via ./watchdog.sh without errors in locally running child tasks (CTRL+C signals not forwarded)

## Limitations

- The example only performs basic UNIX-tool extraction; it is not a full analysis (mapping/quantification workflows need external tools like bwa and ContextMap2)
- Calling java -jar directly instead of ./watchdog.sh forwards CTRL+C signals to all child processes, which can cause errors on locally running tasks
- Watchdog requires JDK 11 or higher; the workflow designer and moduleMaker additionally depend on JavaFX SDK 11 or higher

## Evidence

- [readme] workflow1_basic_information_extraction.xml - simple workflow that extracts information from files using basic UNIX tools: "workflow1_basic_information_extraction.xml - simple workflow that extracts information from files using basic UNIX tools"
- [readme] In order to run a xml-file using the command line tool, call ./watchdog.sh -x xml-file: "In order to run a xml-file using the command line tool, call

`./watchdog.sh -x xml-file` # (or java -jar _jars/watchdog.jar_ -x xml-file)"
- [readme] To call the jar file directly is not recommended as CTRL+C signal will be forwarded to all child processes, which might cause errors on locally running tasks.: "To call the jar file directly is not recommended as CTRL+C signal will be forwarded to all child processes, which might cause errors on locally running tasks."
- [abstract] Watchdog is implemented in Java and thus platform-independent: "Watchdog is implemented in Java and thus platform-independent"
- [abstract] Execution of workflows is possible using either the GUI or a command-line interface: "Execution of workflows is possible using either the GUI or a command-line interface"
