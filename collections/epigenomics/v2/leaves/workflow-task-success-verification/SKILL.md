---
name: workflow-task-success-verification
description: Use when executing a Watchdog XML workflow and each scheduled task's
  completion must be checked (e.g. via a custom success checker as in example_checkers.xml)
  before dependents are released, or when a failed task must be blocked and resolved
  by manual intervention.
when_to_use_negative:
- You only need to validate the XML workflow structure before execution — use workflowValidator.jar
  instead of instrumenting success checks
- The workflow runs under a WMS without user-definable error detection (e.g. Snakemake-style
  filename-inferred dependencies), where per-task custom success checkers do not exist
- Tasks have no declared dependencies or checkers, e.g. the single-task example_basic_sleep.xml,
  where there is nothing downstream to block or verify
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Watchdog
  role: WMS whose control loop schedules tasks, runs custom success checkers, blocks
    downstream tasks on failure, and offers manual intervention (release/restart/skip/resolve)
  repo: github.com/klugem/watchdog
- name: watchdog.jar
  role: command-line jar that executes Watchdog workflows and drives the scheduling/verification
    loop; launched via ./watchdog.sh -x xml-file
  repo: github.com/klugem/watchdog
- name: workflowValidator.jar
  role: pre-execution integrity check of the workflow XML so checkers and dependencies
    are well-formed before verification runs
  repo: github.com/klugem/watchdog
- name: reportGenerator.jar
  role: basic reporting of steps performed during workflow execution, useful to audit
    success/failure outcomes
  repo: github.com/klugem/watchdog
- name: Java
  role: runtime (JDK 11+) required to build and run the Watchdog jar files that execute
    the verification loop
  repo: github.com/klugem/watchdog
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-018-2107-4
    title: Watchdog – a workflow management system for the distributed analysis of
      large-scale experimental data
  source_content_hash: sha256:9305e28c2052cb8d26f34b54c4ef3b6d92e05e3506ea3268b8adaaa42c3c8ce1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - Watchdog
  - XML
  - reportGenerator.jar
  - watchdog.jar
  - workflowValidator.jar
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# workflow-task-success-verification

## Summary

Verifies task success within Watchdog workflow executions by invoking per-task custom success checkers inside the control loop, and on failure blocks downstream tasks while exposing manual intervention actions (release, restart, skip, resolve) via the web-interface and logs.

## When to use

Use when executing a Watchdog XML workflow and each scheduled task's completion must be checked (e.g. via a custom success checker as in example_checkers.xml) before dependents are released, or when a failed task must be blocked and resolved by manual intervention.

## When NOT to use

- You only need to validate the XML workflow structure before execution — use workflowValidator.jar instead of instrumenting success checks
- The workflow runs under a WMS without user-definable error detection (e.g. Snakemake-style filename-inferred dependencies), where per-task custom success checkers do not exist
- Tasks have no declared dependencies or checkers, e.g. the single-task example_basic_sleep.xml, where there is nothing downstream to block or verify

## Inputs

- Watchdog XML workflow definition (e.g. example_dependencies.xml, example_checkers.xml)
- Per-task custom success checker definitions (task actions / checkers section of the XML)
- Task execution stdout/stderr streams and output files produced by each module
- Optionally, workflows pre-validated with workflowValidator.jar

## Outputs

- Per-task success/failure verdicts emitted by the custom success checkers
- Blocked downstream tasks and failure notifications in the run log and web-interface
- Log entries exposing one of the intervention actions: release, restart with modified parameters, skip, or resolve
- Optional report of executed workflow steps from reportGenerator.jar

## How to apply

Run the workflow through the watchdog control loop (`./watchdog.sh -x xml-file`), which schedules dependency-free tasks first and schedules downstream tasks only as dependencies resolve. After each task finishes, invoke its custom success checker (defined per task, cf. example_checkers.xml) to decide whether the task succeeded; optionally validate the workflow definition beforehand with workflowValidator.jar. On detected failure, block the downstream task, record the notification, and surface the four manual intervention actions in the log/web-interface: release, restart (with modified parameters), skip, and resolve. Validate the run afterwards by checking that the log lists every task scheduled in an order consistent with declared dependencies and records exactly one intervention action per simulated failure event; reportGenerator.jar can provide basic reporting of executed steps.

## Related tools

- **Watchdog** (WMS whose control loop schedules tasks, runs custom success checkers, blocks downstream tasks on failure, and offers manual intervention (release/restart/skip/resolve)) — github.com/klugem/watchdog
- **watchdog.jar** (command-line jar that executes Watchdog workflows and drives the scheduling/verification loop; launched via ./watchdog.sh -x xml-file) — github.com/klugem/watchdog
- **workflowValidator.jar** (pre-execution integrity check of the workflow XML so checkers and dependencies are well-formed before verification runs) — github.com/klugem/watchdog
- **reportGenerator.jar** (basic reporting of steps performed during workflow execution, useful to audit success/failure outcomes) — github.com/klugem/watchdog
- **Java** (runtime (JDK 11+) required to build and run the Watchdog jar files that execute the verification loop) — github.com/klugem/watchdog

## Examples

```
./watchdog.sh -x examples/example_checkers.xml
```

## Evaluation signals

- The run log lists every task of the example workflow as scheduled in a topological order consistent with its declared dependencies
- For each simulated failure event, the log records exactly one intervention action (release, restart, skip, or resolve)
- No task was scheduled before all of its declared dependencies had passed their custom success checkers
- workflowValidator.jar accepted the workflow XML before execution (integrity check passed)
- Blocked tasks remain blocked until an explicit manual intervention action is recorded in the web-interface or log

## Limitations

- Requires JDK 11 or higher; the distributed jar files are built for Java 11, and calling java -jar directly forwards CTRL+C to child processes, which might cause errors on locally running tasks
- Some example workflows need modifications or external tools (e.g. example_execution_environments.xml requires modifications; workflow2 needs bwa and ContextMap2 installed and configured)
- No specific version number is stated for the Watchdog software release, and no changelog is available for the repository

## Evidence

- [abstract] customizable error detection and manual intervention: "Main features include straightforward processing of replicate data, support for distributed computer systems, customizable error detection and manual intervention into workflow execution."
- [abstract] web-interface monitoring and intervention: "a web-interface is provided for monitoring the execution status and intervening in case of errors"
- [readme] watchdog.sh invocation: "`./watchdog.sh -x xml-file` # (or java -jar _jars/watchdog.jar_ -x xml-file)"
- [readme] workflowValidator purpose: "workflowValidator.jar - command-line tool that can be used to verify integrity of workflows"
- [readme] custom success checker example: "example_checkers.xml - shows how to use a custom success checker"
