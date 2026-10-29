---
name: maven-build-management
description: Use when a new Watchdog executor plugin has been written in Java (e.g.,
  a cloud executor without SSH) and the modified source must be compiled into runnable
  jar files for the watchdog.sh command-line tool.
when_to_use_negative:
- When installing a stock Watchdog release without code modifications — use the release
  archive, conda (`conda install -c bioconda -c conda-forge watchdog-wms`), or Docker
  (`docker pull klugem/watchdog-wms`) instead of rebuilding
- When the goal is to create or validate workflow modules rather than extend the execution
  layer — module creation uses moduleMaker.jar, not Maven
tools:
- name: Maven
  role: Build system invoked via `mvn` to download dependencies into jars/libs and
    rebuild Watchdog's jar files after plugin code changes
- name: Java
  role: Language in which Watchdog and new executor plugins are implemented; JDK 11+
    required for the build
- name: Watchdog
  role: Target application whose source is built; new executor types are added through
    its plugin system without modifying the core
  repo: github.com/klugem/watchdog
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-018-2107-4
    title: Watchdog – a workflow management system for the distributed analysis of
      large-scale experimental data
  source_content_hash: sha256:40a72ebd2755bc02552f31ec9246b1cdb479928c7c2b7deff5cda59de023d9a8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - Maven
  - Watchdog
  - XML
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# maven-build-management

## Summary

Builds the Watchdog workflow management system and its executor plugins from Java source using Maven, which resolves all dependencies into the jars/libs directory and regenerates the shipped jar files. This is the required step for compiling newly implemented executor types (e.g., cloud executors that bypass SSH) into the executable command-line tool.

## When to use

Use when a new Watchdog executor plugin has been written in Java (e.g., a cloud executor without SSH) and the modified source must be compiled into runnable jar files for the watchdog.sh command-line tool.

## When NOT to use

- When installing a stock Watchdog release without code modifications — use the release archive, conda (`conda install -c bioconda -c conda-forge watchdog-wms`), or Docker (`docker pull klugem/watchdog-wms`) instead of rebuilding
- When the goal is to create or validate workflow modules rather than extend the execution layer — module creation uses moduleMaker.jar, not Maven

## Inputs

- Watchdog Java source checkout from github.com/klugem/watchdog
- New executor plugin source code implementing the Watchdog plugin API
- Maven build configuration (pom.xml) in the repository root

## Outputs

- Rebuilt jar files in jars/ (notably watchdog.jar for the command-line tool) with dependencies in jars/libs
- Executable Watchdog installation ready to run configured XML workflows via ./watchdog.sh -x

## How to apply

Run the Maven build (`mvn`) from the Watchdog source checkout: Maven downloads all dependencies into jars/libs and rebuilds the jar files, including the watchdog.jar command-line tool built for Java 11. First ensure JDK 11 or higher is installed, since the distributed jars are built for Java 11 and called internally by the bash scripts. After a successful build, configure the target example workflow XML (e.g., example_basic_sleep.xml) to use the new executor and run it via ./watchdog.sh -x rather than invoking java -jar directly, because CTRL+C signals forwarded to child processes by direct jar invocation can kill locally running tasks. The plugin-API approach (used by the authors for the Slurm executor) avoids modifying Watchdog's core code when adding new execution backends.

## Related tools

- **Maven** (Build system invoked via `mvn` to download dependencies into jars/libs and rebuild Watchdog's jar files after plugin code changes)
- **Java** (Language in which Watchdog and new executor plugins are implemented; JDK 11+ required for the build)
- **Watchdog** (Target application whose source is built; new executor types are added through its plugin system without modifying the core) — github.com/klugem/watchdog

## Examples

```
./watchdog.sh -x example_basic_sleep.xml
```

## Evaluation signals

- `mvn` completes without errors and the jar files in jars/ are regenerated with dependencies present in jars/libs
- ./watchdog.sh -x example_basic_sleep.xml runs the configured example workflow end-to-end under the new executor
- All tasks in the executed workflow finish in a success state with no error-detection alerts in the execution log
- The built jars run under Java 11 (the version they are distributed for) without version-mismatch errors

## Limitations

- The build requires JDK 11 or higher; the GUI components (workflow designer, moduleMaker) additionally depend on the JavaFX SDK 11 or higher, which Watchdog ships in jars/libs/modules/ by default
- Running the resulting jar directly via java -jar is discouraged because CTRL+C signals are forwarded to all child processes, which can cause errors on locally running tasks
- No specific version number is stated for the Watchdog software release in the article, complicating reproducibility of builds

## Evidence

- [readme] Maven build of Watchdog source: "Watchdog can be build from the source files using Maven. The command `mvn` downloads all dependencies into _jars/libs_ and rebuilds the jar files."
- [readme] JDK requirement: "Watchdog is written in Java and requires JDK 11 or higher."
- [readme] Why to launch via bash script, not java -jar: "To call the jar file directly is not recommended as CTRL+C signal will be forwarded to all child processes, which might cause errors on locally running tasks."
- [intro] Java implementation basis for plugin development: "Although Watchdog is implemented in Java, there is no restriction on which programs can be included as modules."
