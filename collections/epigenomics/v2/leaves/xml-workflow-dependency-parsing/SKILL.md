---
name: xml-workflow-dependency-parsing
description: Use when an agent must execute or reconstruct a Watchdog workflow from
  an XML file (e.g. example_dependencies.xml), resolving task dependencies, verifying
  success with custom checkers, and handling blocked tasks.
when_to_use_negative:
- The workflow is defined as Snakemake rules or a Galaxy/KNIME pipeline rather than
  a Watchdog XML file — dependency inference mechanisms differ (Snakemake infers dependencies
  by matching filenames)
- Only module integrity needs checking — use moduleValidator.jar on modules instead
  of parsing a workflow XML
- A GUI-driven construction/execution session is required — use the workflow designer
  (watchdogDesigner.sh) rather than the command-line XML parsing loop
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0769
tools:
- name: Watchdog
  role: Workflow management system whose control loop parses the XML workflow, schedules
    dependency-free tasks first, and runs success checkers
  repo: https://github.com/klugem/watchdog
- name: workflowValidator.jar
  role: Pre-execution validation of the XML workflow's integrity before parsing into
    the task/dependency representation
  repo: https://github.com/klugem/watchdog
- name: Java
  role: Runtime (JDK 11+) and implementation language for the watchdog.jar command-line
    executor
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-018-2107-4
    title: Watchdog – a workflow management system for the distributed analysis of
      large-scale experimental data
  source_content_hash: sha256:9077e61eadbb8cbb3c4cf8b19f527ad6e468f3ff6ba3ffa537891981c42e7976
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/klugem/watchdog
  tools:
  - Java
  - Watchdog
  - XML
  - workflowValidator.jar
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# xml-workflow-dependency-parsing

## Summary

Parses a Watchdog XML workflow definition into an in-memory task/dependency graph, generates the initial set of dependency-free tasks, and schedules downstream tasks as their dependencies resolve, with per-task success checking and manual intervention on failure.

## When to use

Use when an agent must execute or reconstruct a Watchdog workflow from an XML file (e.g. example_dependencies.xml), resolving task dependencies, verifying success with custom checkers, and handling blocked tasks.

## When NOT to use

- The workflow is defined as Snakemake rules or a Galaxy/KNIME pipeline rather than a Watchdog XML file — dependency inference mechanisms differ (Snakemake infers dependencies by matching filenames)
- Only module integrity needs checking — use moduleValidator.jar on modules instead of parsing a workflow XML
- A GUI-driven construction/execution session is required — use the workflow designer (watchdogDesigner.sh) rather than the command-line XML parsing loop

## Inputs

- Watchdog XML workflow file (e.g. example_dependencies.xml)
- Custom success checker definitions (e.g. example_checkers.xml)
- Watchdog installation built with Maven (jars/watchdog.jar, jars/workflowValidator.jar, Java 11+)

## Outputs

- In-memory task/dependency graph derived from the parsed XML workflow
- Execution run log listing every task scheduled in topological order
- Per-task success-checker verdicts and failure/intervention records (release, restart, skip, resolve)

## How to apply

Build Watchdog from source with Maven so dependencies land in jars/libs and the Java 11 jar files are rebuilt. Parse the XML workflow (e.g. example_dependencies.xml, configured via helper_scripts/configureExamples.sh) into an in-memory task/dependency representation, optionally validating integrity first with workflowValidator.jar. Launch execution with ./watchdog.sh -x xml-file, which generates the initial set of tasks that have no unresolved dependencies and schedules remaining tasks in dependency order as upstream tasks complete. In the control loop, invoke each task's custom success checker (cf. example_checkers.xml) to verify task success; on detected failure, block downstream tasks, record a notification, and expose the manual actions release, restart with modified parameters, skip, and resolve. Validation is considered successful if the run log lists every task scheduled in a topological order consistent with declared dependencies and records exactly one intervention action per simulated failure.

## Related tools

- **Watchdog** (Workflow management system whose control loop parses the XML workflow, schedules dependency-free tasks first, and runs success checkers) — https://github.com/klugem/watchdog
- **workflowValidator.jar** (Pre-execution validation of the XML workflow's integrity before parsing into the task/dependency representation) — https://github.com/klugem/watchdog
- **Java** (Runtime (JDK 11+) and implementation language for the watchdog.jar command-line executor)

## Examples

```
./watchdog.sh -x examples/example_dependencies.xml
```

## Evaluation signals

- The run log lists every task of the example workflow as scheduled in a topological order consistent with its declared dependencies
- Exactly one intervention action (release, restart, skip, or resolve) is recorded per simulated failure event
- workflowValidator.jar reports the workflow XML as valid before execution
- Only tasks with no unresolved dependencies appear in the initial task set; downstream tasks are scheduled only after their upstream dependencies complete successfully

## Limitations

- Distributed jar files are built for Java 11 or higher; running the jar directly (java -jar jars/watchdog.jar) forwards CTRL+C to all child processes, which may cause errors on locally running tasks
- Some example workflows require modifications or external tools (e.g. example_execution_environments.xml requires changes; workflow2_differential_gene_expression.xml needs bwa and ContextMap2 installed and configured)
- No specific version number is stated for the Watchdog software release in the article

## Evidence

- [abstract] Main features include straightforward processing of replicate data, support for distributed computer systems, customizable error detection and manual intervention into workflow execution.: "Main features include straightforward processing of replicate data, support for distributed computer systems, customizable error detection and manual intervention into workflow execution."
- [other] The orchestrator validates and parses the XML workflow, schedules dependency-free tasks first: [evidence span withheld — anchor status: not_found]
- [readme] In order to run a xml-file using the command line tool, call ./watchdog.sh -x xml-file: "In order to run a xml-file using the command line tool, call

`./watchdog.sh -x xml-file` # (or java -jar _jars/watchdog.jar_ -x xml-file)"
- [readme] workflowValidator.jar - command-line tool that can be used to verify integrity of workflows: "workflowValidator.jar - command-line tool that can be used to verify integrity of workflows"
