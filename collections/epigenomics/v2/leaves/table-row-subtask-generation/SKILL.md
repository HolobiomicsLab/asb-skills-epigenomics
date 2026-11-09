---
name: table-row-subtask-generation
description: Use when an XML Watchdog workflow task definition contains <processblocks>
  entries and the concrete set of subtasks (one per sequence element, matching path,
  or table row) must be enumerated before execution.
when_to_use_negative:
- The workflow task definition contains no process blocks (e.g. example_basic_sleep.xml
  with a single fixed task) — no instantiation is needed
- Subtask inputs are already concrete file paths written directly in the XML task
  rather than placeholders to be expanded
- You are authoring module definitions rather than instantiating workflow subtasks
  — use the moduleMaker workflow instead
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Watchdog
  role: Workflow management system whose process block mechanism expands XML task
    definitions into one subtask per instance; run via watchdog.sh -x
  repo: https://github.com/klugem/watchdog
- name: Java
  role: Platform-independent implementation language of Watchdog; requires JDK 11
    or higher to build and run the jar files that execute workflows
  repo: https://github.com/klugem/watchdog
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-018-2107-4
    title: Watchdog – a workflow management system for the distributed analysis of
      large-scale experimental data
  source_content_hash: sha256:9933c1db6e478756b507290fb4eab5e83407e74c3136b63be310f8b9dd05e144
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# table-row-subtask-generation

## Summary

Generates the set of Watchdog subtasks from process blocks defined in an XML workflow: each process block (process sequences, folders/paths, or tables) enumerates instances holding one or more variables, and one subtask is created per instance with placeholders replaced by the instance's variable values.

## When to use

Use when an XML Watchdog workflow task definition contains <processblocks> entries and the concrete set of subtasks (one per sequence element, matching path, or table row) must be enumerated before execution.

## When NOT to use

- The workflow task definition contains no process blocks (e.g. example_basic_sleep.xml with a single fixed task) — no instantiation is needed
- Subtask inputs are already concrete file paths written directly in the XML task rather than placeholders to be expanded
- You are authoring module definitions rather than instantiating workflow subtasks — use the moduleMaker workflow instead

## Inputs

- Watchdog XML workflow definition file containing process block definitions (e.g. example_process_blocks.xml)
- For folder/file-pattern blocks: filesystem paths matching the pattern
- For table blocks: the table (rows with fields) defined in the process block
- For dependent blocks: return values of preceding tasks in the workflow

## Outputs

- Expanded set of subtasks, one per process block instance (sequence element, matching path, or table row)
- Task definitions with all subtask placeholders substituted by instance variable values

## How to apply

Load the XML workflow definition (e.g. the shipped example_process_blocks.xml, generated via ./helper_scripts/configureExamples.sh) and parse each process block definition, identifying all placeholders in the block. For process-sequence blocks, enumerate all listed sequence elements; for folder/file-pattern blocks, enumerate matching paths on the filesystem; for table blocks, expand one instance per table row, substituting each row's fields into the placeholders. For blocks driven by return values of preceding tasks, link each instance to the upstream task's returned value. Each instance becomes one subtask, with subtask placeholders in the task definition replaced by that instance's variable values, so Watchdog can then execute the expanded tasks (e.g. via ./watchdog.sh -x xml-file).

## Related tools

- **Watchdog** (Workflow management system whose process block mechanism expands XML task definitions into one subtask per instance; run via watchdog.sh -x) — https://github.com/klugem/watchdog
- **Java** (Platform-independent implementation language of Watchdog; requires JDK 11 or higher to build and run the jar files that execute workflows) — https://github.com/klugem/watchdog

## Examples

```
./helper_scripts/configureExamples.sh -i /path/to/install/folder/of/watchdog && ./watchdog.sh -x /path/to/install/folder/of/watchdog/examples/example_process_blocks.xml
```

## Evaluation signals

- Running ./watchdog.sh -x on the expanded XML workflow completes without placeholder-substitution errors
- The number of generated subtasks equals the number of instances derivable from each block (one per sequence element, one per matching path, one per table row)
- No unsubstituted placeholder tokens remain in the instantiated task definitions
- Subtasks derived from preceding-task return values are linked to the correct upstream task output
- Blocks of all types present in the example (sequences, folders, tables) each yield at least one expanded subtask

## Limitations

- No quantitative threshold for instance expansion is reported in the paper; correctness is judged structurally (placeholders fully substituted) rather than numerically
- Folder/file-pattern block expansion depends on filesystem state at instantiation time, so missing directories or non-matching paths yield no instances
- Blocks driven by preceding tasks' return values only expand after the upstream task has completed, so subtask counts are not known upfront

## Evidence

- [readme] example_process_blocks.xml: "example_process_blocks.xml - shows how to work with process sequences, folders and tables"
- [readme] configure examples: "Once Watchdog is correctly installed, you can run example workflows shipped with Watchdog. To configure them run `./helper_scripts/configureExamples.sh -i /path/to/install/folder/of/watchdog`"
- [discussion] XML syntax: "Due to the wide use of XML, most potential users of Watchdog will already be familiar with the syntax used in Watchdog"
- [readme] run workflow: "In order to run a xml-file using the command line tool, call

`./watchdog.sh -x xml-file`"
