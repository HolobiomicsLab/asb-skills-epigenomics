---
name: java-plugin-api-extension
description: Use when a Watchdog workflow must dispatch tasks to an execution environment
  not covered by the built-in local, remote SSH, or DRMAA/Slurm executors — e.g. a
  cloud computing service that does not allow SSH access.
when_to_use_negative:
- The target execution environment is already supported by the built-in local, remote
  SSH, or DRMAA/Slurm executors
- Only a new analysis module is needed — use moduleMaker / module definitions instead
  of extending the execution layer
- The workflow design itself (not the execution backend) needs to change — use the
  workflow designer GUI or XML syntax rather than the plugin API
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Watchdog
  role: Workflow management system whose plugin system is extended with a new executor
    type
  repo: github.com/klugem/watchdog
- name: Java
  role: Implementation language for the new executor plugin; Watchdog requires JDK
    11 or higher and builds via Maven
- name: XML
  role: Workflow definition format configured to reference the new executor (e.g.
    example_basic_sleep.xml)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-018-2107-4
    title: Watchdog – a workflow management system for the distributed analysis of
      large-scale experimental data
  source_content_hash: sha256:0be5176dd0170ab82899d89de10ec4ad4f2ad0eedd70332fcaf7c4af663e47b1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - Watchdog
  - XML
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# java-plugin-api-extension

## Summary

Extends Watchdog's task execution layer with new executor types (e.g. a cloud executor without SSH) by implementing a Java plugin against its executor API, without modifying the Watchdog core. The same plugin mechanism was already exercised by the authors to add a Slurm executor.

## When to use

Use when a Watchdog workflow must dispatch tasks to an execution environment not covered by the built-in local, remote SSH, or DRMAA/Slurm executors — e.g. a cloud computing service that does not allow SSH access.

## When NOT to use

- The target execution environment is already supported by the built-in local, remote SSH, or DRMAA/Slurm executors
- Only a new analysis module is needed — use moduleMaker / module definitions instead of extending the execution layer
- The workflow design itself (not the execution backend) needs to change — use the workflow designer GUI or XML syntax rather than the plugin API

## Inputs

- Watchdog plugin API specification (Additional file 3)
- Slurm executor source as reference implementation pattern
- Watchdog XML workflow file (e.g. example_basic_sleep.xml)
- Watchdog source tree buildable with Maven

## Outputs

- New Java executor plugin (e.g. cloud executor without SSH) integrated into rebuilt jar files
- Configured XML workflow using the new executor
- Successfully completed workflow run with all tasks in success state

## How to apply

Study the plugin API documented in Additional file 3 and use the authors' Slurm executor as the reference pattern for adding new execution backends. Implement a new executor plugin in Java — Watchdog itself is implemented in Java, so the plugin mirrors the existing executor interface and is platform-independent. Build from source with Maven: running `mvn` downloads all dependencies into jars/libs and rebuilds the jar files, so the plugin is linked into the shipped watchdog.jar. Configure an XML workflow (e.g. the shipped example_basic_sleep.xml, which shows the XML workflow structure with one task) to reference the new executor, then launch it via ./watchdog.sh -x rather than calling java -jar directly, since CTRL+C signals forwarded to child processes can cause errors on locally running tasks. Treat the plugin as sanctioned by the authors explicitly as the route to supporting cloud computing services without SSH.

## Related tools

- **Watchdog** (Workflow management system whose plugin system is extended with a new executor type) — github.com/klugem/watchdog
- **Java** (Implementation language for the new executor plugin; Watchdog requires JDK 11 or higher and builds via Maven)
- **XML** (Workflow definition format configured to reference the new executor (e.g. example_basic_sleep.xml))

## Examples

```
./watchdog.sh -x example_basic_sleep.xml
```

## Evaluation signals

- The example workflow example_basic_sleep.xml completes end-to-end under the new executor
- All tasks in the workflow finish in a success state with no error-detection alerts in the execution log
- Watchdog core code was not modified — the new executor was added purely as a plugin
- Maven build (mvn) completes and rebuilt jar files in jars/ include the new plugin
- Launch via ./watchdog.sh -x does not abort with signal-forwarding errors on locally running tasks

## Limitations

- Calling java -jar jars/watchdog.jar directly is not recommended because CTRL+C signals will be forwarded to all child processes, possibly erroring on locally running tasks
- The plugin requires the Java toolchain (JDK 11 or higher per README) and a Maven build of the Watchdog source
- Article text does not state a specific version number for the Watchdog software release, so plugin-API compatibility across versions is unverified from context

## Evidence

- [abstract] Watchdog is implemented in Java and thus platform-independent: "Watchdog is implemented in Java and thus platform-independent"
- [discussion] Due to the wide use of XML, most potential users of Watchdog will already be familiar with the syntax used in Watchdog: "Due to the wide use of XML, most potential users of Watchdog will already be familiar with the syntax used in Watchdog"
- [readme] The command `mvn` downloads all dependencies into _jars/libs_ and rebuilds the jar files.: "Watchdog can be build from the source files using Maven. The command `mvn` downloads all dependencies into _jars/libs_ and rebuilds the jar files."
- [readme] To call the jar file directly is not recommended as CTRL+C signal will be forwarded to all child processes, which might cause errors on locally running tasks.: "To call the jar file directly is not recommended as CTRL+C signal will be forwarded to all child processes, which might cause errors on locally running tasks."
