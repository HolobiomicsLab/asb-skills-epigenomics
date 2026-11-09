---
name: xml-workflow-placeholder-expansion
description: Use when a Watchdog XML workflow contains process blocks (process sequences,
  folders, tables, or predecessor return values) and you need to enumerate the resulting
  subtasks by expanding placeholders into concrete instance variable assignments.
when_to_use_negative:
- The workflow XML has no process blocks (e.g. example_basic_sleep.xml, a single-task
  workflow with no instance expansion)
- The goal is module creation or validation (use moduleMaker or moduleValidator) rather
  than task instance expansion
- You need to design or edit the workflow graph itself rather than enumerate the instances
  a process block generates
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: Watchdog
  role: Workflow management system whose process-block mechanism is being reconstructed;
    executed via ./watchdog.sh -x xml-file to run the expanded workflow
  repo: https://github.com/klugem/watchdog
- name: Java
  role: Runtime platform in which Watchdog is implemented; requires JDK 11 or higher
  repo: https://github.com/klugem/watchdog
- name: example_process_blocks.xml
  role: Shipped example workflow showing how to work with process sequences, folders
    and tables; the reference input for instance expansion
  repo: https://github.com/klugem/watchdog
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-018-2107-4
    title: Watchdog – a workflow management system for the distributed analysis of
      large-scale experimental data
  source_content_hash: sha256:6d0ef0b7b5a7dfdf878f1741135a3e37d7bbb1c95eef9b9b44c96a47319aa4bd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/klugem/watchdog
  tools:
  - Java
  - Watchdog
  - XML
  - example_process_blocks.xml
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# xml-workflow-placeholder-expansion

## Summary

Reconstructs how Watchdog's process-block mechanism turns an XML workflow task definition into a set of subtask instances, where each process block defines a set of instances holding one or more variables and one subtask is created per instance with placeholders replaced by instance variable values.

## When to use

Use when a Watchdog XML workflow contains process blocks (process sequences, folders, tables, or predecessor return values) and you need to enumerate the resulting subtasks by expanding placeholders into concrete instance variable assignments.

## When NOT to use

- The workflow XML has no process blocks (e.g. example_basic_sleep.xml, a single-task workflow with no instance expansion)
- The goal is module creation or validation (use moduleMaker or moduleValidator) rather than task instance expansion
- You need to design or edit the workflow graph itself rather than enumerate the instances a process block generates

## Inputs

- Watchdog XML workflow file with process block definitions (e.g. example_process_blocks.xml)
- Folder paths or file patterns enumerated by folder-type process blocks
- Table data with one row per instance for table-type process blocks
- Return values of preceding tasks for return-value-driven blocks

## Outputs

- Expanded set of subtask instances, one per process block instance, with placeholders replaced by variable values
- Tabular listing of expanded instances (e.g. process_block_instances.tsv, one row per subtask)

## How to apply

First generate and load the shipped example workflow example_process_blocks.xml via ./helper_scripts/configureExamples.sh -i /path/to/install/folder/of/watchdog, which places it in the examples subdirectory. Parse the XML process block definitions and identify all placeholders in each block according to the block types illustrated by the example (process sequences, folders, tables). For process sequence blocks, enumerate all sequence elements and substitute each into the task instance pattern; for folder/file-pattern blocks, enumerate matching paths and substitute them; for table blocks, expand one instance per table row substituting row fields into placeholders. For blocks driven by return values of preceding tasks, link each instance to the upstream task's returned value and substitute accordingly. Write the complete instance/subtask sets to a TSV with one row per expanded subtask and validate that every block definition in the example (sequences, folders, tables) produced expanded rows.

## Related tools

- **Watchdog** (Workflow management system whose process-block mechanism is being reconstructed; executed via ./watchdog.sh -x xml-file to run the expanded workflow) — https://github.com/klugem/watchdog
- **Java** (Runtime platform in which Watchdog is implemented; requires JDK 11 or higher) — https://github.com/klugem/watchdog
- **example_process_blocks.xml** (Shipped example workflow showing how to work with process sequences, folders and tables; the reference input for instance expansion) — https://github.com/klugem/watchdog

## Examples

```
./helper_scripts/configureExamples.sh -i /path/to/install/folder/of/watchdog && ./watchdog.sh -x examples/example_process_blocks.xml
```

## Evaluation signals

- The script runs without error on example_process_blocks.xml after configureExamples.sh configuration
- The output TSV (e.g. process_block_instances.tsv) exists and contains one expanded row per instance for every block definition (sequences, folders, tables) in the example
- No placeholders remain in any expanded subtask — each instance row carries concrete substituted variable values
- For return-value-driven blocks, each instance correctly references the upstream task's returned value
- The expanded workflow can be executed end-to-end with ./watchdog.sh -x path2/xml-file.xml

## Limitations

- No quantitative threshold for expansion correctness is reported in the paper — validation is structural (rows exist per block definition), not numeric
- Execution via direct java -jar invocation is discouraged because CTRL+C signals are forwarded to all child processes and may error locally running tasks
- Some example workflows (e.g. example_execution_environments.xml) require modifications before they can be run

## Evidence

- [readme] process blocks example: "example_process_blocks.xml - shows how to work with process sequences, folders and tables"
- [readme] configure examples: "To configure them run `./helper_scripts/configureExamples.sh -i /path/to/install/folder/of/watchdog [-m your@mail-adress.com]`"
- [readme] run workflow: "In order to run a xml-file using the command line tool, call

`./watchdog.sh -x xml-file`"
- [readme] Java requirement: "Watchdog is written in Java and requires JDK 11 or higher."
