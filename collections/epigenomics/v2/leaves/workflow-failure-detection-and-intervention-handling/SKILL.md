---
name: workflow-failure-detection-and-intervention-handling
description: Use when a Watchdog XML workflow execution must verify task success with
  custom success checkers and the run log must record blocked tasks and one intervention
  action (release, restart, skip, resolve) per failure.
when_to_use_negative:
- The workflow is not a Watchdog XML workflow (e.g. a Snakemake or Galaxy workflow)
  — Watchdog's checker/intervention machinery does not apply
- The user needs fully automatic error recovery without manual intervention — this
  skill's core mechanism is explicit human release/restart/skip/resolve decisions
- The task has no defined success checker and exit-status checking is sufficient —
  the extra checker configuration adds overhead
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Watchdog
  role: Executes the workflow control loop, invokes custom success checkers, blocks
    failed tasks, and exposes manual intervention actions
  repo: github.com/klugem/watchdog
- name: workflowValidator.jar
  role: Pre-execution verification of the integrity of the XML workflow whose tasks
    will be checked and intervened upon
  repo: github.com/klugem/watchdog
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-018-2107-4
    title: Watchdog – a workflow management system for the distributed analysis of
      large-scale experimental data
  source_content_hash: sha256:045a4f09c82f2525ada44caacfaa8c697724eb33daff9ea058fb0f219fd765ff
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Watchdog
  - XML
  - workflowValidator.jar
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# workflow-failure-detection-and-intervention-handling

## Summary

Detects failed tasks in a Watchdog workflow run using per-task custom success checkers and manages the recovery path: the failed task is blocked, the user is notified, and manual intervention actions (release, restart, skip, resolve) are offered. This is Watchdog's customizable error-detection and manual-intervention mechanism for large-scale analysis runs.

## When to use

Use when a Watchdog XML workflow execution must verify task success with custom success checkers and the run log must record blocked tasks and one intervention action (release, restart, skip, resolve) per failure.

## When NOT to use

- The workflow is not a Watchdog XML workflow (e.g. a Snakemake or Galaxy workflow) — Watchdog's checker/intervention machinery does not apply
- The user needs fully automatic error recovery without manual intervention — this skill's core mechanism is explicit human release/restart/skip/resolve decisions
- The task has no defined success checker and exit-status checking is sufficient — the extra checker configuration adds overhead

## Inputs

- Watchdog XML workflow file (e.g. example_dependencies.xml, example_checkers.xml)
- Custom success checker definitions (XML)
- Workflow execution run log
- Java 11 runtime with watchdog.jar

## Outputs

- Run log listing every task scheduled in dependency-consistent (topological) order
- Task success/failure determinations from custom success checkers
- Blocked-task records and user notifications per failure
- Intervention records: exactly one of release / restart / skip / resolve per failure event

## How to apply

Define a custom success checker per task (as shown in example_checkers.xml) so the control loop can verify completion rather than relying on exit codes alone. During the scheduling loop, invoke the checker after each task; on detected failure, block downstream tasks that depend on the failed one, record the notification, and expose the manual intervention actions release, restart (with modified parameters), skip, and resolve in the log or web-interface. Choose 'restart' only after adjusting task parameters, 'skip' when the downstream analysis tolerates missing input, and 'resolve' after fixing the problem manually. Verify correctness by confirming the run log lists every task scheduled in a topological order consistent with the declared dependencies and records exactly one intervention action per simulated failure event. Integrity of the workflow definition can be pre-checked with workflowValidator.jar before execution.

## Related tools

- **Watchdog** (Executes the workflow control loop, invokes custom success checkers, blocks failed tasks, and exposes manual intervention actions) — github.com/klugem/watchdog
- **workflowValidator.jar** (Pre-execution verification of the integrity of the XML workflow whose tasks will be checked and intervened upon) — github.com/klugem/watchdog

## Examples

```
./watchdog.sh -x path2/xml-file.xml
```

## Evaluation signals

- Run log lists every task of the example workflow as scheduled in a topological order consistent with its declared dependencies
- Run log records exactly one intervention action (release, restart, skip, or resolve) per simulated failure event
- Each task's custom success checker (as defined in example_checkers.xml) is invoked before the task is marked successful
- All downstream tasks of a failed task appear as blocked in the log until an intervention action is applied
- workflowValidator.jar reports the workflow as valid before the run begins

## Limitations

- Watchdog requires JDK 11 or higher, and the workflow designer GUI additionally depends on the JavaFX SDK 11 or higher
- Running watchdog.jar directly (java -jar jars/watchdog.jar) forwards CTRL+C to all child processes, which might cause errors on locally running tasks — the ./watchdog.sh wrapper is recommended
- Intervention actions are manual: the system notifies the user but does not automatically choose between release, restart, skip, and resolve

## Evidence

- [abstract] Main features include straightforward processing of replicate data, support for distributed computer systems, customizable error detection and manual intervention into workflow execution.: "Main features include straightforward processing of replicate data, support for distributed computer systems, customizable error detection and manual intervention into workflow execution."
- [abstract] a web-interface is provided for monitoring the execution status and intervening in case of errors: "a web-interface is provided for monitoring the execution status and intervening in case of errors"
- [readme] example_checkers.xml - shows how to use a custom success checker: "example_checkers.xml - shows how to use a custom success checker"
- [readme] watchdog.jar - command-line tool that executes Watchdog workflows: "watchdog.jar - command-line tool that executes Watchdog workflows"
