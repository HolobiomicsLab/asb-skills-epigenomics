---
name: java-command-line-workflow-execution
description: Use when a multi-step analysis must be executed from a Java command line
  as an XML-defined workflow with task dependencies, custom success verification,
  and manual error intervention (release/restart/skip/resolve).
when_to_use_negative:
- The job is a single standalone command with no inter-task dependencies — run it
  directly instead of wrapping it in a Watchdog workflow
- Dependencies should be inferred automatically by filename matching (the Snakemake-style
  rules approach) rather than declared explicitly in XML
- Interactive, browser-based workflow construction by users without programming experience
  is the primary goal (use the workflow designer GUI workflow instead)
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Watchdog
  role: Workflow management system whose command-line control loop parses the XML
    workflow, schedules tasks, verifies success, and manages error intervention
  repo: https://github.com/klugem/watchdog
- name: watchdog.jar
  role: Java 11 jar launched via ./watchdog.sh -x that generates the initial dependency-free
    tasks and runs the scheduling loop
  repo: https://github.com/klugem/watchdog
- name: workflowValidator.jar
  role: Pre-execution check that verifies the integrity of the XML workflow before
    the control loop runs
  repo: https://github.com/klugem/watchdog
- name: Java
  role: Implementation platform; Watchdog requires JDK 11+ and its distributed jars
    are built for Java 11
  repo: https://github.com/klugem/watchdog
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-018-2107-4
    title: Watchdog – a workflow management system for the distributed analysis of
      large-scale experimental data
  source_content_hash: sha256:0187a4a941a93d3938c8172e538075da6401e72ad78d0a587ef3dbb6a0672246
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/klugem/watchdog
  tools:
  - Java
  - Watchdog
  - XML
  - watchdog.jar
  - workflowValidator.jar
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 18
---

# java-command-line-workflow-execution

## Summary

Reconstructs and drives the Watchdog control loop: parsing an XML workflow, scheduling dependency-free tasks first, invoking per-task custom success checkers, and handling failures via task blocking plus manual intervention actions.

## When to use

Use when a multi-step analysis must be executed from a Java command line as an XML-defined workflow with task dependencies, custom success verification, and manual error intervention (release/restart/skip/resolve).

## When NOT to use

- The job is a single standalone command with no inter-task dependencies — run it directly instead of wrapping it in a Watchdog workflow
- Dependencies should be inferred automatically by filename matching (the Snakemake-style rules approach) rather than declared explicitly in XML
- Interactive, browser-based workflow construction by users without programming experience is the primary goal (use the workflow designer GUI workflow instead)

## Inputs

- XML workflow definition file (e.g. example_dependencies.xml)
- Module definitions folder (e.g. watchdog-wms-modules)
- Optional custom success checker definitions (example_checkers.xml)
- Watchdog installation folder configured via helper_scripts/configureExamples.sh

## Outputs

- Executed task outputs produced in dependency order
- Run log listing every task scheduled in topological order consistent with declared dependencies
- Failure notifications and exactly one recorded intervention action (release/restart/skip/resolve) per failure event

## How to apply

Build Watchdog from source with Maven so `mvn` resolves dependencies into jars/libs and rebuilds the Java 11 jar files. Optionally verify workflow integrity with workflowValidator.jar, then launch via `./watchdog.sh -x xml-file`, which parses the XML into an in-memory task/dependency representation. In the control loop, first generate the tasks with no unresolved dependencies, then schedule remaining tasks in dependency (topological) order as upstream dependencies resolve. For each scheduled task, invoke its custom success checker (cf. example_checkers.xml) to verify completion. On detected failure, block the downstream task, record the notification, and expose the manual intervention actions release, restart (with modified parameters), skip, and resolve in the log.

## Related tools

- **Watchdog** (Workflow management system whose command-line control loop parses the XML workflow, schedules tasks, verifies success, and manages error intervention) — https://github.com/klugem/watchdog
- **watchdog.jar** (Java 11 jar launched via ./watchdog.sh -x that generates the initial dependency-free tasks and runs the scheduling loop) — https://github.com/klugem/watchdog
- **workflowValidator.jar** (Pre-execution check that verifies the integrity of the XML workflow before the control loop runs) — https://github.com/klugem/watchdog
- **Java** (Implementation platform; Watchdog requires JDK 11+ and its distributed jars are built for Java 11) — https://github.com/klugem/watchdog

## Examples

```
./watchdog.sh -x path2/xml-file.xml
```

## Evaluation signals

- workflowValidator.jar reports the XML workflow as structurally valid before execution
- The run log lists every task of the workflow as scheduled in a topological order consistent with its declared dependencies
- No task is scheduled before all of its declared upstream dependencies have resolved successfully
- Each task's custom success checker is invoked and its verdict recorded for every scheduled task
- Exactly one intervention action (release, restart, skip, or resolve) is recorded per simulated failure event

## Limitations

- Watchdog requires JDK 11 or higher, and the workflow designer and moduleMaker additionally depend on JavaFX SDK 11+
- Invoking watchdog.jar directly (java -jar jars/watchdog.jar) is not recommended because a CTRL+C signal is forwarded to all child processes, which may error locally running tasks — use ./watchdog.sh
- Tasks that fail block their downstream dependents until a manual intervention action (release, restart, skip, or resolve) is taken

## Evidence

- [abstract] Main features include straightforward processing of replicate data, support for distributed computer systems, customizable error detection and manual intervention into workflow execution.: "Main features include straightforward processing of replicate data, support for distributed computer systems, customizable error detection and manual intervention into workflow execution."
- [readme] In order to run a xml-file using the command line tool, call: "In order to run a xml-file using the command line tool, call"
- [readme] workflowValidator.jar - command-line tool that can be used to verify integrity of workflows: "workflowValidator.jar - command-line tool that can be used to verify integrity of workflows"
- [readme] Watchdog is written in Java and requires JDK 11 or higher.: "Watchdog is written in Java and requires JDK 11 or higher."
