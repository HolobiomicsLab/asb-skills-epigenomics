---
name: cross-wms-workflow-reimplementation
description: Use when benchmarking the scheduling overhead of a workflow management
  system (e.g. Watchdog vs. Snakemake) against the runtime of the tasks themselves,
  or when re-implementing a Watchdog workflow as Snakemake rules.
when_to_use_negative:
- The goal is to benchmark raw task performance (e.g. mapper speed) rather than workflow-management
  scheduling overhead
- The workflow cannot be expressed equivalently in both systems, so a fair same-task-chain
  comparison is impossible
- Only a single sample count is available, so no overhead-vs-sample-count trend can
  be assessed
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0769
tools:
- name: Watchdog
  role: Executes the configured workflow XML via the command line over scaled sample
    sets; overhead of the WMS itself is the measurement target
  repo: github.com/klugem/watchdog
- name: Snakemake
  role: Python-based WMS in which the same task chain is re-implemented as rules with
    filename-matched dependencies, run over identical sample sets
- name: reportGenerator.jar
  role: Extracts basic reporting of steps performed during execution of a Watchdog
    workflow, used to obtain per-step runtimes
  repo: github.com/klugem/watchdog
- name: Java
  role: Runtime for Watchdog; the distributed jar files are built for Java 11 and
    are called internally by the bash scripts
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-018-2107-4
    title: Watchdog – a workflow management system for the distributed analysis of
      large-scale experimental data
  source_content_hash: sha256:a54ce025bddb3c75d016d63c3285b8e20c02bec199b38edc478afd6bc84178f7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - Snakemake
  - Watchdog
  - reportGenerator.jar
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# cross-wms-workflow-reimplementation

## Summary

Reproduces the Watchdog vs. Snakemake computational-overhead comparison by executing the same task chain under both workflow management systems across scaled sample counts and measuring WMS overhead as wall time minus summed task runtimes.

## When to use

Use when benchmarking the scheduling overhead of a workflow management system (e.g. Watchdog vs. Snakemake) against the runtime of the tasks themselves, or when re-implementing a Watchdog workflow as Snakemake rules.

## When NOT to use

- The goal is to benchmark raw task performance (e.g. mapper speed) rather than workflow-management scheduling overhead
- The workflow cannot be expressed equivalently in both systems, so a fair same-task-chain comparison is impossible
- Only a single sample count is available, so no overhead-vs-sample-count trend can be assessed

## Inputs

- Watchdog workflow XML file (e.g. an example workflow from examples/ such as example_dependencies.xml)
- Scaled sets of input samples of increasing size
- Snakemake rules file re-implementing the same task chain

## Outputs

- wms_overhead_benchmark.csv with per-sample-count overhead (total wall time minus summed task runtimes) for Watchdog and Snakemake
- Per-step execution reports from reportGenerator.jar

## How to apply

Configure a runnable workflow XML by running ./helper_scripts/configureExamples.sh pointed at the Watchdog installation folder, then execute the workflow repeatedly with ./watchdog.sh -x <xml-file> while scaling the number of input samples; the distributed jars are built for Java 11 and are invoked internally by the bash scripts. Extract per-step execution reports with reportGenerator.jar, which provides basic reporting of steps performed during execution of a workflow. Re-implement the identical task chain as Snakemake rules so that dependencies between tasks are automatically inferred by matching filenames, and run it over the identical sample sets. For each sample count, compute WMS overhead as total wall time minus the summed task runtimes for both systems and write the results to wms_overhead_benchmark.csv. Acceptance is qualitative: measured Watchdog and Snakemake overheads must remain negligible relative to total task runtime, matching the trend reported in the paper's Additional file 4.

## Related tools

- **Watchdog** (Executes the configured workflow XML via the command line over scaled sample sets; overhead of the WMS itself is the measurement target) — github.com/klugem/watchdog
- **Snakemake** (Python-based WMS in which the same task chain is re-implemented as rules with filename-matched dependencies, run over identical sample sets)
- **reportGenerator.jar** (Extracts basic reporting of steps performed during execution of a Watchdog workflow, used to obtain per-step runtimes) — github.com/klugem/watchdog
- **Java** (Runtime for Watchdog; the distributed jar files are built for Java 11 and are called internally by the bash scripts)

## Examples

```
./helper_scripts/configureExamples.sh -i /path/to/watchdog && ./watchdog.sh -x examples/example_dependencies.xml
```

## Evaluation signals

- Measured Watchdog and Snakemake overheads remain negligible relative to total task runtime across the entire sample-count range
- The overhead-vs-sample-count trend qualitatively matches Additional file 4 of the paper
- Overhead is computed consistently as total wall time minus summed task runtimes for both systems at every sample count
- The Snakemake re-implementation executes the identical task chain over identical sample sets, with dependencies inferred by filename matching
- wms_overhead_benchmark.csv contains one overhead row per sample count for both Watchdog and Snakemake

## Limitations

- Overhead conclusions are qualitative (negligible relative to task runtime), not a precise quantitative reproduction
- Java version constraints: Watchdog requires JDK 11 or higher, and the jar files are built for Java 11
- The finding is stated to likely generalize to 'any other WMS', but only Watchdog and Snakemake were measured
- Calling the watchdog.jar directly is discouraged because the CTRL+C signal is forwarded to all child processes, which might cause errors on locally running tasks

## Evidence

- [intro] Snakemake allows definition of tasks based on rules and automatically infers dependencies between tasks by matching filenames.: "Snakemake allows definition of tasks based on rules and automatically infers dependencies between tasks by matching filenames."
- [other] The computational overhead of using either Watchdog or Snakemake ... is negligible compared to the actual runtime of the executed tasks: [evidence span withheld — anchor status: not_found]
- [readme] reportGenerator.jar - basic reporting of steps performed during execution of a workflow: "reportGenerator.jar - basic reporting of steps performed during execution of a workflow"
- [readme] The distributed jar files are build for Java 11 and are called internally by the bash scripts.: "The distributed jar files are build for Java 11 and are called internally by the bash scripts."
- [readme] To configure them run ./helper_scripts/configureExamples.sh -i /path/to/install/folder/of/watchdog: "To configure them run `./helper_scripts/configureExamples.sh -i /path/to/install/folder/of/watchdog [-m your@mail-adress.com]`."
