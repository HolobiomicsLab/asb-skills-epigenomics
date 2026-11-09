---
name: cloud-executor-integration-without-ssh
description: Use when the target compute environment for Watchdog tasks is a cloud
  service or other execution backend that is not reachable via SSH and is not covered
  by the built-in local, remote-SSH, or DRMAA/Slurm executors.
when_to_use_negative:
- The target execution environment is already served by Watchdog's built-in local,
  remote SSH, or DRMAA/Slurm cluster executors
- The compute resource allows SSH — use the built-in remote executor instead of writing
  a plugin
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Watchdog
  role: Workflow management system whose plugin system is extended with the new executor
    type
  repo: github.com/klugem/watchdog
- name: Java
  role: Implementation language for the new executor plugin; Watchdog requires JDK
    11 or higher
- name: XML
  role: Format of the example workflow (example_basic_sleep.xml) configured to exercise
    the new executor
- name: Maven
  role: Build tool used to download dependencies into jars/libs and rebuild the jar
    files after adding the plugin
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-018-2107-4
    title: Watchdog – a workflow management system for the distributed analysis of
      large-scale experimental data
  source_content_hash: sha256:bc72b5ec1a06a4da3bdd23365acc8adb175365093250da2c5f31937bb0b90afb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - Maven
  - Watchdog
  - XML
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# cloud-executor-integration-without-ssh

## Summary

Extends Watchdog's task execution layer beyond its built-in executors (local, remote SSH, DRMAA/Slurm) by implementing a new executor plugin in Java, following the pattern of the authors' own Slurm executor. This is the sanctioned route for dispatching Watchdog tasks to cloud computing services that do not allow SSH.

## When to use

Use when the target compute environment for Watchdog tasks is a cloud service or other execution backend that is not reachable via SSH and is not covered by the built-in local, remote-SSH, or DRMAA/Slurm executors.

## When NOT to use

- The target execution environment is already served by Watchdog's built-in local, remote SSH, or DRMAA/Slurm cluster executors
- The compute resource allows SSH — use the built-in remote executor instead of writing a plugin

## Inputs

- Watchdog plugin API specification (Additional file 3)
- Slurm executor source code as reference pattern
- Example workflow XML (example_basic_sleep.xml) with XML workflow structure

## Outputs

- New Java executor plugin class dispatching Watchdog tasks to a non-SSH cloud service
- Rebuilt Watchdog jar files with dependencies in jars/libs
- Configured example workflow XML bound to the new executor

## How to apply

Study the plugin API documented in Additional file 3 and use the authors' Slurm executor as the reference pattern for adding new execution backends without modifying the core code. Implement the new executor as a Java plugin that mirrors the Slurm executor's interface and dispatches tasks to the non-SSH cloud service. Rebuild the software with Maven, since the `mvn` command downloads all dependencies into jars/libs and rebuilds the jar files. Configure the shipped one-task example workflow example_basic_sleep.xml to use the new executor, then launch it via `./watchdog.sh -x` rather than invoking java -jar directly, because CTRL+C signals forwarded to child processes can break locally running tasks.

## Related tools

- **Watchdog** (Workflow management system whose plugin system is extended with the new executor type) — github.com/klugem/watchdog
- **Java** (Implementation language for the new executor plugin; Watchdog requires JDK 11 or higher)
- **XML** (Format of the example workflow (example_basic_sleep.xml) configured to exercise the new executor)
- **Maven** (Build tool used to download dependencies into jars/libs and rebuild the jar files after adding the plugin)

## Examples

```
./watchdog.sh -x example_basic_sleep.xml
```

## Evaluation signals

- The example workflow example_basic_sleep.xml completes end-to-end under the new executor with all tasks finishing in a success state
- No error-detection alerts appear in the execution log during the run
- The plugin required no modifications to the Watchdog core code, mirroring how the Slurm executor was added
- Rebuilt jar files are consistent after `mvn` (dependencies present in jars/libs)
- Workflow was launched via ./watchdog.sh and local tasks were not disrupted by CTRL+C signal forwarding

## Limitations

- Requires JDK 11 or higher; the workflow designer additionally depends on JavaFX SDK 11 or higher
- The plugin must follow the interface of existing executors (Slurm is the reference); no article-side validation of a cloud executor implementation is provided beyond the statement that it is sanctioned
- Direct java -jar invocation is not recommended as CTRL+C signals will be forwarded to all child processes, which might cause errors on locally running tasks

## Evidence

- [intro] Although Watchdog is implemented in Java, there is no restriction on which programs can be included as modules.: "Although Watchdog is implemented in Java, there is no restriction on which programs can be included as modules."
- [discussion] Due to the wide use of XML, most potential users of Watchdog will already be familiar with the syntax used in Watchdog: "Due to the wide use of XML, most potential users of Watchdog will already be familiar with the syntax used in Watchdog"
- [readme] Watchdog can be build from the source files using Maven. The command `mvn` downloads all dependencies into _jars/libs_ and rebuilds the jar files.: "The command `mvn` downloads all dependencies into _jars/libs_ and rebuilds the jar files."
- [readme] To call the jar file directly is not recommended as CTRL+C signal will be forwarded to all child processes: "To call the jar file directly is not recommended as CTRL+C signal will be forwarded to all child processes, which might cause errors on locally running tasks."
- [readme] example_basic_sleep.xml - basic example with one task to show XML workflow structure: "example_basic_sleep.xml - basic example with one task to show XML workflow structure"
