---
name: xml-workflow-configuration
description: Use when a task requires defining or running a multi-step analysis pipeline
  in Watchdog, editing XML workflow definitions (task dependencies, executors, success
  checkers, constants), or reusing shipped example workflows such as workflow1_basic_information_extraction.xml.
when_to_use_negative:
- The analysis consists of a single ad-hoc command with no dependencies, replication,
  or error handling needs — a plain shell script suffices
- The target system cannot run Java 11+ / JDK 11 with JavaFX SDK 11+ (required for
  watchdog, workflow designer and moduleMaker GUIs)
- You need to author new analysis tools rather than orchestrate existing ones — use
  moduleMaker.jar (obtained via ./helper_scripts/downloadModuleMaker.sh) instead of
  editing workflow XML alone
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3071
tools:
- name: Watchdog
  role: Workflow management system whose XML files define tasks, dependencies, executors
    and success checkers; executed via ./watchdog.sh -x
  repo: github.com/klugem/watchdog
- name: Java
  role: Runtime (JDK 11+) required to build Watchdog from source with Maven and to
    execute the distributed Java 11 jar files
  repo: github.com/klugem/watchdog
- name: XML
  role: Format in which Watchdog workflows and module configurations are authored;
    widely used syntax chosen so most users are already familiar
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1186/s12859-018-2107-4
    title: Watchdog – a workflow management system for the distributed analysis of
      large-scale experimental data
  source_content_hash: sha256:9e6a39aca14b5853d9e7da81405d0f3c66e0bda9d6f139a68e7df3b9a3c58c17
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - Watchdog
  - XML
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3071
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# xml-workflow-configuration

## Summary

Configure, validate, and launch Watchdog workflows expressed as XML files, using the shipped example workflows as templates and the watchdog.sh command-line wrapper rather than direct java -jar calls.

## When to use

Use when a task requires defining or running a multi-step analysis pipeline in Watchdog, editing XML workflow definitions (task dependencies, executors, success checkers, constants), or reusing shipped example workflows such as workflow1_basic_information_extraction.xml.

## When NOT to use

- The analysis consists of a single ad-hoc command with no dependencies, replication, or error handling needs — a plain shell script suffices
- The target system cannot run Java 11+ / JDK 11 with JavaFX SDK 11+ (required for watchdog, workflow designer and moduleMaker GUIs)
- You need to author new analysis tools rather than orchestrate existing ones — use moduleMaker.jar (obtained via ./helper_scripts/downloadModuleMaker.sh) instead of editing workflow XML alone

## Inputs

- Watchdog workflow XML file (e.g. workflow1_basic_information_extraction.xml)
- Watchdog installation directory with jars/watchdog.jar and helper scripts
- Input data files referenced by the workflow (e.g. gzipped FASTQ test data)
- Module definitions from github.com/watchdog-wms/watchdog-wms-modules

## Outputs

- Executed workflow task outputs (e.g. decompressed FASTQ extracts, merged compressed information files) under the workflow's results directory
- Execution log with per-task status and success-checker verdicts

## How to apply

Write or adapt a workflow XML file defining tasks, their dependencies, execution environments (local, remote SSH, DRMAA/Slurm, or plugin executors), and success checkers; the shipped example_basic_sleep.xml shows the basic one-task XML structure and workflow1_basic_information_extraction.xml shows a real pipeline of basic UNIX tools (gunzip FASTQ decompression, parallel header/sequence extraction, re-compression and merging). Configure example workflows in-place with ./helper_scripts/configureExamples.sh -i /path/to/install/folder/of/watchdog, which places them under the examples directory. Launch from the installation directory with ./watchdog.sh -x path2/xml-file.xml — never call java -jar jars/watchdog.jar directly, because CTRL+C signals would be forwarded to all child processes and can kill locally running tasks. Validate structure beforehand with workflowValidator.jar and module integrity with moduleValidator.jar, and judge success by all tasks finishing without triggering Watchdog's error detection and by the presence of expected outputs (e.g., merged compressed information files in the workflow's results directory). Workflows and modules can be shared via the watchdog-wms GitHub organizations, where contributed workflows must be GPLv3 licensed.

## Related tools

- **Watchdog** (Workflow management system whose XML files define tasks, dependencies, executors and success checkers; executed via ./watchdog.sh -x) — github.com/klugem/watchdog
- **Java** (Runtime (JDK 11+) required to build Watchdog from source with Maven and to execute the distributed Java 11 jar files) — github.com/klugem/watchdog
- **XML** (Format in which Watchdog workflows and module configurations are authored; widely used syntax chosen so most users are already familiar)

## Examples

```
./watchdog.sh -x examples/workflow1_basic_information_extraction.xml
```

## Evaluation signals

- All workflow tasks finish without triggering Watchdog's customizable error detection
- Success checkers attached to tasks report success for every task
- Expected output artifacts exist in the workflow's results directory (e.g. merged compressed information files for Workflow 1)
- workflowValidator.jar reports the XML workflow as structurally valid; moduleValidator.jar verifies module integrity
- The process was started via ./watchdog.sh (not java -jar) and CTRL+C interruption does not leave partially killed child tasks

## Limitations

- CTRL+C signals are forwarded to all child processes if the jar is invoked directly via java -jar, which can cause errors on locally running tasks
- workflow2_differential_gene_expression.xml requires external tools bwa and ContextMap2 to be installed and configured
- Building the workflow designer and moduleMaker requires the JavaFX SDK 11 or higher; the bash scripts try to locate it automatically
- No specific version number is stated for the software release in the article's discussion section

## Evidence

- [readme] Launch via watchdog.sh: "To call the jar file directly is not recommended as CTRL+C signal will be forwarded to all child processes, which might cause errors on locally running tasks."
- [readme] Configure examples: "To configure them run `./helper_scripts/configureExamples.sh -i /path/to/install/folder/of/watchdog`"
- [discussion] XML familiarity rationale: "Due to the wide use of XML, most potential users of Watchdog will already be familiar with the syntax used in Watchdog"
- [abstract] Java implementation: "Watchdog is implemented in Java and thus platform-independent"
- [abstract] Error detection feature: "Main features include straightforward processing of replicate data, support for distributed computer systems, customizable error detection and manual intervention into workflow execution."
