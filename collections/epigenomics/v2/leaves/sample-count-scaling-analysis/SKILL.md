---
name: sample-count-scaling-analysis
description: Use when a runnable Watchdog workflow XML and an equivalent Snakemake
  rule chain are available, and the goal is to measure how WMS overhead grows as the
  number of input samples is scaled across multiple benchmark runs.
when_to_use_negative:
- The workflow steps themselves are the object of study (e.g. read mapping or differential
  gene expression accuracy) rather than the WMS scheduling overhead
- Only a single sample count is available, so no scaling trend of overhead vs. runtime
  can be established
- The task chain cannot be expressed in both Watchdog XML and Snakemake rules, making
  an apples-to-apples wall-time comparison impossible
tools:
- name: Watchdog
  role: Executes the benchmark workflow XML via its command-line tool; its jar files
    and example workflows are configured and run at increasing sample counts
  repo: https://github.com/klugem/watchdog
- name: Snakemake
  role: Python-based WMS used as the comparison system; tasks are defined as rules
    with dependencies automatically inferred by matching filenames
- name: Java
  role: Runtime for the distributed Watchdog jar files (watchdog.jar, reportGenerator.jar)
    called internally by the bash scripts
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-018-2107-4
    title: Watchdog – a workflow management system for the distributed analysis of
      large-scale experimental data
  source_content_hash: sha256:3708afce3e9568eb208f8385692d1411dea1c487d5cdceb81ab21dff04659c6b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/klugem/watchdog
  tools:
  - Java
  - Snakemake
  - Watchdog
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# sample-count-scaling-analysis

## Summary

Benchmarks the computational overhead of a workflow management system (Watchdog vs. Snakemake) by scaling the number of input samples and comparing total wall time to summed task runtimes. Rationale: quantify whether WMS bookkeeping cost matters relative to the actual executed tasks, as reported in Additional file 4 of the Watchdog paper.

## When to use

Use when a runnable Watchdog workflow XML and an equivalent Snakemake rule chain are available, and the goal is to measure how WMS overhead grows as the number of input samples is scaled across multiple benchmark runs.

## When NOT to use

- The workflow steps themselves are the object of study (e.g. read mapping or differential gene expression accuracy) rather than the WMS scheduling overhead
- Only a single sample count is available, so no scaling trend of overhead vs. runtime can be established
- The task chain cannot be expressed in both Watchdog XML and Snakemake rules, making an apples-to-apples wall-time comparison impossible

## Inputs

- Watchdog example workflow XML (e.g. workflow1_basic_information_extraction.xml) configured to the installation folder
- Input sample sets at multiple sample counts for scaling the workflow execution
- Equivalent Snakemake rule set re-implementing the same task chain (dependencies inferred by matching filenames)
- Java 11 runtime (JDK 11 or higher) for the distributed Watchdog jar files

## Outputs

- Per-step execution reports from reportGenerator.jar
- wms_overhead_benchmark.csv containing Watchdog vs. Snakemake overhead (total wall time minus summed task runtimes) per sample count

## How to apply

First configure the example workflows shipped with Watchdog via ./helper_scripts/configureExamples.sh, pointing at the Watchdog installation folder, so a runnable workflow XML is produced. Execute the workflow repeatedly with ./watchdog.sh -x <xml-file> while scaling the number of input samples; the runs use the distributed Java 11 jar files called internally by the bash scripts. Extract per-step execution reports with reportGenerator.jar, which provides basic reporting of steps performed during execution of a workflow. Re-implement the identical task chain as Snakemake rules so that dependencies between tasks are automatically inferred by matching filenames, and run Snakemake over the same sample sets. For each sample count, compute WMS overhead as total wall time minus summed task runtimes for both systems and write the comparison to wms_overhead_benchmark.csv. Validate by confirming that measured overheads remain negligible relative to total task runtime across the sample-count range, qualitatively matching the trend reported in Additional file 4.

## Related tools

- **Watchdog** (Executes the benchmark workflow XML via its command-line tool; its jar files and example workflows are configured and run at increasing sample counts) — https://github.com/klugem/watchdog
- **Snakemake** (Python-based WMS used as the comparison system; tasks are defined as rules with dependencies automatically inferred by matching filenames)
- **Java** (Runtime for the distributed Watchdog jar files (watchdog.jar, reportGenerator.jar) called internally by the bash scripts)

## Examples

```
./helper_scripts/configureExamples.sh -i /path/to/install/folder/of/watchdog && ./watchdog.sh -x path2/xml-file.xml
```

## Evaluation signals

- For every sample count, WMS overhead (total wall time minus summed task runtimes) is a small fraction of total task runtime for both Watchdog and Snakemake
- The overhead-vs-sample-count trend qualitatively matches the comparison reported in Additional file 4
- reportGenerator.jar emits per-step execution reports that account for all steps performed during each workflow execution
- The Snakemake run resolves the same task dependencies as the Watchdog XML via filename matching over identical sample sets
- Both systems complete successfully on every sample count in the scaling range (no scheduling or dependency errors)

## Limitations

- Overhead is measured for Watchdog and Snakemake; generalization to any other WMS is an assumption ("and likely any other WMS"), not directly measured
- Requires JDK 11 or higher; running jars directly (java -jar jars/watchdog.jar) is discouraged because the CTRL+C signal is forwarded to child processes, which might cause errors on locally running tasks
- The conclusion is qualitative (overhead negligible relative to task runtime); the paper does not state a numeric overhead threshold

## Evidence

- [readme] To run a xml-file using the command line tool, call ./watchdog.sh -x xml-file: "In order to run a xml-file using the command line tool, call

`./watchdog.sh -x xml-file` # (or java -jar _jars/watchdog.jar_ -x xml-file)"
- [readme] reportGenerator.jar provides basic reporting of steps performed during execution of a workflow: "reportGenerator.jar - basic reporting of steps performed during execution of a workflow"
- [intro] Snakemake allows definition of tasks based on rules and automatically infers dependencies between tasks by matching filenames: "Snakemake allows definition of tasks based on rules and automatically infers dependencies between tasks by matching filenames."
- [discussion] Watchdog was developed to support the automated and distributed analysis of large-scale experimental data: "we present the WMS Watchdog, which was developed to support the automated and distributed analysis of large-scale experimental data"
