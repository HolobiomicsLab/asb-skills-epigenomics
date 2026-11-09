---
name: task-dependency-topological-scheduling
description: 'Reference for driving or reimplementing the Watchdog execution engine:
  you must parse an XML workflow (e.g. example_dependencies.xml), schedule tasks in
  dependency order, verify success via custom checkers, and handle failures.'
when_to_use_negative:
- The task graph has no dependencies and a simple single-task workflow (example_basic_sleep.xml)
  suffices without scheduling logic
- You only need to validate workflow or module integrity without executing — use workflowValidator.jar
  or moduleValidator.jar instead of the execution loop
- Workflows must be defined by rule-based filename inference (as in Snakemake) rather
  than explicitly declared XML dependencies
edam_operation: http://edamontology.org/operation_3760
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3365
tools:
- name: Watchdog
  role: Workflow management system whose orchestrator parses the XML workflow, generates
    dependency-free tasks, schedules in dependency order, verifies success, and blocks
    on failure
  repo: github.com/klugem/watchdog
- name: workflowValidator.jar
  role: Command-line tool used to verify integrity of the XML workflow before execution
- name: Java
  role: Runtime/implementation language; watchdog.jar targets Java 11 and is invoked
    via bash scripts
- name: Maven
  role: Build tool; `mvn` downloads all dependencies into jars/libs and rebuilds the
    jar files
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-018-2107-4
    title: Watchdog – a workflow management system for the distributed analysis of
      large-scale experimental data
  source_content_hash: sha256:2a35b857194c2d864d9ef3088df5f0b4f5a7d168b5777985e4a03c4e34e7ef07
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - Maven
  - Watchdog
  - XML
  - workflowValidator.jar
  edam_operation: http://edamontology.org/operation_3760
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3365
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# task-dependency-topological-scheduling

## Summary

Reconstructs Watchdog's core control loop: parsing an XML workflow into an in-memory task/dependency representation, generating the initial set of dependency-free tasks, scheduling tasks as dependencies resolve, invoking per-task success checkers, and blocking downstream tasks on failure with manual intervention options.

## When to use

Reference for driving or reimplementing the Watchdog execution engine: you must parse an XML workflow (e.g. example_dependencies.xml), schedule tasks in dependency order, verify success via custom checkers, and handle failures.

## When NOT to use

- The task graph has no dependencies and a simple single-task workflow (example_basic_sleep.xml) suffices without scheduling logic
- You only need to validate workflow or module integrity without executing — use workflowValidator.jar or moduleValidator.jar instead of the execution loop
- Workflows must be defined by rule-based filename inference (as in Snakemake) rather than explicitly declared XML dependencies

## Inputs

- XML workflow file (e.g. example_dependencies.xml) configured via helper_scripts/configureExamples.sh
- Watchdog modules defining the executable tasks
- Custom success checker definitions (e.g. example_checkers.xml)

## Outputs

- Execution run log listing every task scheduled in topological order
- Per-task success-checker verdicts (pass/fail)
- Failure notifications and recorded manual intervention actions (release, restart, skip, resolve)

## How to apply

Build Watchdog from source with Maven so `mvn` resolves dependencies into jars/libs and rebuilds the Java 11 jar files, then parse the XML workflow (optionally validating its integrity first with workflowValidator.jar) into an in-memory task/dependency representation. Generate the initial set of tasks with no unresolved dependencies, as watchdog.jar does when launched via `./watchdog.sh -x xml-file`, and schedule them in a control loop. As each task completes, invoke its custom success checker (cf. example_checkers.xml) and release downstream tasks once their dependencies resolve, scheduling in a topological order consistent with declared dependencies. On detected failure, block the downstream task, record the notification, and expose the manual intervention actions release, restart (with modified parameters), skip, and resolve. Judge correctness by confirming the run log schedules every task in a topological order consistent with the XML dependencies and records exactly one intervention action per simulated failure event.

## Related tools

- **Watchdog** (Workflow management system whose orchestrator parses the XML workflow, generates dependency-free tasks, schedules in dependency order, verifies success, and blocks on failure) — github.com/klugem/watchdog
- **workflowValidator.jar** (Command-line tool used to verify integrity of the XML workflow before execution)
- **Java** (Runtime/implementation language; watchdog.jar targets Java 11 and is invoked via bash scripts)
- **Maven** (Build tool; `mvn` downloads all dependencies into jars/libs and rebuilds the jar files)

## Examples

```
./watchdog.sh -x path2/xml-file.xml
```

## Evaluation signals

- Run log lists every task of the example workflow as scheduled in a topological order consistent with the declared XML dependencies
- Run log records exactly one intervention action (release, restart, skip, or resolve) per simulated failure event
- No downstream task is scheduled before all of its declared dependencies have passed their success checkers
- workflowValidator.jar accepts the workflow file as structurally valid before the run

## Limitations

- Calling jars directly (java -jar jars/watchdog.jar) is not recommended because CTRL+C forwards to all child processes and may error locally running tasks; use ./watchdog.sh
- Requires JDK 11 or higher; the workflow designer and moduleMaker additionally depend on JavaFX SDK 11+
- Some example workflows (e.g. example_execution_environments.xml, workflow2_differential_gene_expression.xml) require modifications or external tools (bwa, ContextMap2) installed and configured

## Evidence

- [readme] readme_mvn_build: "Watchdog can be build from the source files using Maven. The command `mvn` downloads all dependencies into _jars/libs_ and rebuilds the jar files."
- [readme] readme_validator: "workflowValidator.jar - command-line tool that can be used to verify integrity of workflows"
- [readme] readme_checkers_example: "example_checkers.xml - shows how to use a custom success checker"
- [abstract] abstract_webinterface: "a web-interface is provided for monitoring the execution status and intervening in case of errors"
- [intro] intro_error_detection: "Additionally, Watchdog provides a sophisticated error detection system that can be customized by the user and allows manual intervention."
- [readme] readme_run: "In order to run a xml-file using the command line tool, call"
