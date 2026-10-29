---
name: workflow-management-system-execution
description: Use when a multi-step bioinformatics analysis (e.g. FASTQ information
  extraction or RNA-seq replicate analysis) must be orchestrated as reproducible module
  tasks with error detection, success checkers, or distributed execution across conditions
  and replicates.
when_to_use_negative:
- The analysis is a single one-off command better run directly in a shell, where WMS
  overhead (XML authoring, module definitions) adds no value
- The target environment lacks Java 11+ (and JavaFX for the GUI/moduleMaker), so neither
  watchdog.sh nor the workflow designer can run
- The workflow requires tools not present as Watchdog modules (e.g. workflow2 needs
  bwa and ContextMap2 to be installed and configured first)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: Watchdog
  role: Workflow management system that executes module-based XML workflows via command
    line (watchdog.jar) or GUI (watchdogDesigner.jar) with error detection and web-based
    monitoring
  repo: https://github.com/klugem/watchdog
- name: Java
  role: Runtime platform (JDK 11+) in which Watchdog is implemented and on which its
    distributed jar files are built
- name: XML
  role: Syntax used to define Watchdog workflows and task dependencies
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-018-2107-4
    title: Watchdog – a workflow management system for the distributed analysis of
      large-scale experimental data
  source_content_hash: sha256:bfdf6849130a7f79e32b3f462fb055bf07bced1df5bebf85f26595286b2b11ea
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
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# workflow-management-system-execution

## Summary

Execute scientific analysis workflows with the Watchdog workflow management system (WMS), a Java-11-based tool that encapsulates analysis tasks as modules and orchestrates them via XML workflow definitions from the GUI or command line. This skill covers installing Watchdog, configuring shipped example workflows, and running them via ./watchdog.sh so error detection, success checkers, and CTRL+C signal handling behave correctly.

## When to use

Use when a multi-step bioinformatics analysis (e.g. FASTQ information extraction or RNA-seq replicate analysis) must be orchestrated as reproducible module tasks with error detection, success checkers, or distributed execution across conditions and replicates.

## When NOT to use

- The analysis is a single one-off command better run directly in a shell, where WMS overhead (XML authoring, module definitions) adds no value
- The target environment lacks Java 11+ (and JavaFX for the GUI/moduleMaker), so neither watchdog.sh nor the workflow designer can run
- The workflow requires tools not present as Watchdog modules (e.g. workflow2 needs bwa and ContextMap2 to be installed and configured first)

## Inputs

- XML workflow definition file (e.g. workflow1_basic_information_extraction.xml)
- Gzipped FASTQ test data files (bundled HSV-1-derived example dataset)
- Watchdog installation folder with jars/ directory and modules
- Community modules from the watchdog-wms-modules repository (optional)

## Outputs

- Decompressed FASTQ extracts produced by Workflow 1 module tasks
- Merged, compressed information files in the workflow's results directory ('Workflow 1 - Basic Information Extraction')
- Workflow execution logs and error-detection reports (optionally mail notifications on completed subtasks)

## How to apply

Install Watchdog either from a GitHub release, via `conda install -c bioconda -c conda-forge watchdog-wms`, `docker pull klugem/watchdog-wms`, or by building from source with Maven (which downloads dependencies into jars/libs and rebuilds the jar files); JDK 11 or higher is required, and the GUI/workflow designer needs JavaFX SDK 11+. Build or obtain workflow XML definitions, then configure the shipped examples with `./helper_scripts/configureExamples.sh -i /path/to/install/folder/of/watchdog` so they appear under the examples directory. Launch workflows from the installation directory with `./watchdog.sh -x path2/xml-file.xml` rather than calling `java -jar jars/watchdog.jar` directly, because direct jar invocation forwards CTRL+C signals to all child processes and can break locally running tasks. Let Watchdog execute the module tasks (e.g. Workflow 1's gunzip decompression of FASTQ, parallel extraction of read headers and sequence subsets, recompression and merging) and verify completion through the workflow's success checkers; monitor status and intervene on errors via the web-interface.

## Related tools

- **Watchdog** (Workflow management system that executes module-based XML workflows via command line (watchdog.jar) or GUI (watchdogDesigner.jar) with error detection and web-based monitoring) — https://github.com/klugem/watchdog
- **Java** (Runtime platform (JDK 11+) in which Watchdog is implemented and on which its distributed jar files are built)
- **XML** (Syntax used to define Watchdog workflows and task dependencies)

## Examples

```
./watchdog.sh -x examples/workflow1_basic_information_extraction.xml
```

## Evaluation signals

- All workflow tasks finish without triggering Watchdog's customizable error detection
- The merged compressed output files exist in the workflow's results directory (e.g. under 'Workflow 1 - Basic Information Extraction')
- The workflow's success checkers report success for every task
- Workflow 1 completes in under one minute using about 50MB of storage on a modern desktop computer
- No child processes were killed mid-run (confirming the workflow was launched via ./watchdog.sh, not java -jar directly)

## Limitations

- Calling `java -jar jars/watchdog.jar` directly is not recommended because CTRL+C signals are forwarded to all child processes, which might cause errors on locally running tasks
- The GUI (workflow designer) and moduleMaker depend on JavaFX SDK 11 or higher
- Workflow 2 (differential gene expression example) needs bwa and ContextMap2 to be installed and configured
- The distributed jar files are built for Java 11 specifically

## Evidence

- [readme] Launch via wrapper script rather than direct jar call: "To call the jar file directly is not recommended as CTRL+C signal will be forwarded to all child processes, which might cause errors on locally running tasks."
- [readme] Example workflow configuration command: "To configure them run `./helper_scripts/configureExamples.sh -i /path/to/install/folder/of/watchdog`"
- [readme] Java 11 requirement: "Watchdog is written in Java and requires JDK 11 or higher."
- [abstract] WMS features: "Main features include straightforward processing of replicate data, support for distributed computer systems, customizable error detection and manual intervention into workflow execution."
- [abstract] Command-line execution mode: "Execution of workflows is possible using either the GUI or a command-line interface"
