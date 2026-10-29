---
name: file-pattern-path-enumeration
description: Use when a Watchdog XML task must be replicated over a folder or file
  pattern, so one subtask instance is created per matching path with placeholders
  substituted.
when_to_use_negative:
- The set of subtask inputs is a fixed, known table of conditions/replicates better
  expressed as a process table block expanding one instance per table row.
- Instances must follow the return values of a preceding task (process input block)
  rather than filesystem paths.
- The workflow is not a Watchdog XML definition — this pattern-instantiation mechanism
  is specific to Watchdog process blocks.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Watchdog
  role: Workflow management system whose folder/file-pattern process blocks enumerate
    matching paths and create one subtask per instance; run via ./watchdog.sh -x
  repo: https://github.com/klugem/watchdog
- name: Java
  role: Implementation language of Watchdog; the enumeration/instantiation runs platform-independently
    on JDK 11 or higher
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-018-2107-4
    title: Watchdog – a workflow management system for the distributed analysis of
      large-scale experimental data
  source_content_hash: sha256:fdb3166b489d5ca3075567ca20286e003b1e3170fa110464ef449ebe87670523
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
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# file-pattern-path-enumeration

## Summary

Reconstructs how Watchdog's folder/file-pattern process blocks enumerate matching filesystem paths from an XML workflow task definition and instantiate one subtask per matching path, substituting each path into the instance's placeholders.

## When to use

Use when a Watchdog XML task must be replicated over a folder or file pattern, so one subtask instance is created per matching path with placeholders substituted.

## When NOT to use

- The set of subtask inputs is a fixed, known table of conditions/replicates better expressed as a process table block expanding one instance per table row.
- Instances must follow the return values of a preceding task (process input block) rather than filesystem paths.
- The workflow is not a Watchdog XML definition — this pattern-instantiation mechanism is specific to Watchdog process blocks.

## Inputs

- XML workflow task definition containing a folder/file-pattern process block (e.g. example_process_blocks.xml from the Watchdog examples folder)
- Directory tree or file pattern over which matching paths are enumerated

## Outputs

- One Watchdog subtask instance per matching file path, with subtask placeholders replaced by the enumerated path
- process_block_instances.tsv with one row per expanded subtask

## How to apply

After running ./helper_scripts/configureExamples.sh -i to place the shipped examples in the installation folder's examples subdirectory, load example_process_blocks.xml and locate the process block of folder/file-pattern type in the XML task definition. Enumerate every path that matches the block's pattern, treating each match as one instance of the block. Watchdog process blocks define a set of instances, each holding one or more variables, and one subtask is created per instance, with subtask placeholders in the task definition replaced by the instance's variable values. If instances should instead be driven by return values of a preceding task, link each instance to the upstream task's returned value rather than to a filesystem path. Write the complete instance/subtask set to process_block_instances.tsv with one row per expanded subtask so the enumeration can be audited.

## Related tools

- **Watchdog** (Workflow management system whose folder/file-pattern process blocks enumerate matching paths and create one subtask per instance; run via ./watchdog.sh -x) — https://github.com/klugem/watchdog
- **Java** (Implementation language of Watchdog; the enumeration/instantiation runs platform-independently on JDK 11 or higher)

## Examples

```
./helper_scripts/configureExamples.sh -i /path/to/install/folder/of/watchdog; ./watchdog.sh -x /path/to/install/folder/of/watchdog/examples/example_process_blocks.xml
```

## Evaluation signals

- Running ./watchdog.sh -x on the configured example_process_blocks.xml completes without errors.
- process_block_instances.tsv exists and contains expanded instance rows for every block definition present in the example (sequences, folders, tables).
- The number of generated subtasks equals the number of filesystem paths matching the folder/file-pattern block.
- No unresolved subtask placeholders remain — every generated subtask's placeholders are fully replaced by an instance variable value.

## Limitations

- No quantitative threshold is reported in the paper for validating the expansion; correctness is judged by the presence of expanded instance rows, not by counts.
- Enumeration covers paths matching at instantiation time; the provided context does not describe re-enumeration for files appearing later.
- Requires a Watchdog installation (Java/JDK 11 or higher) with examples configured via ./helper_scripts/configureExamples.sh.

## Evidence

- [full_text] Process blocks instantiate one subtask per instance with placeholders replaced by instance variables: [evidence span withheld — anchor status: not_found]
- [full_text] Folder/file-pattern block expansion procedure: [evidence span withheld — anchor status: not_found]
- [readme] Shipped example demonstrating folder blocks alongside sequences and tables: "example_process_blocks.xml - shows how to work with process sequences, folders and tables"
- [readme] Example workflows are configured into the examples folder and executed with watchdog.sh: "the example workflows are located in _/path/to/install/folder/of/watchdog/examples_ and can be started using `./watchdog.sh -x path2/xml-file.xml`"
