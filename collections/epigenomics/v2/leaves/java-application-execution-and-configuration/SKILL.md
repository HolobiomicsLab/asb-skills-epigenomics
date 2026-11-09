---
name: java-application-execution-and-configuration
description: Use when a scientific workflow requires running a packaged Java tool
  (e.g. TSMiner.jar) that needs Java 1.5+, a 1024 MB heap, and manual-specified input/parameter
  files on a platform-independent system.
when_to_use_negative:
- Input is not time-series expression data — TSMiner is specifically designed to construct
  time-specific regulatory networks from time-series expression profiles, and clustering
  methods that cannot evaluate temporal dynamics of regulators are a different analysis
- Available memory cannot provide the 1024 MB minimum Java heap, or Java 1.5+ is not
  installed and cannot be installed
- The task is signal prediction or cascaded network analysis handled by DREM extensions
  (iDREM, SDREM) rather than TSMiner's TF–pathway network construction
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0203
tools:
- name: TSMiner
  role: Java application being launched and configured; constructs time-specific regulatory
    networks from time-series expression profiles
  repo: https://github.com/free1234hm/tsminer-tool.git
- name: Java
  role: Runtime environment required to execute TSMiner.jar; language in which TSMiner's
    core algorithms and interfaces are written
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab629
    title: 'TSMiner: a novel framework for generating time-specific gene regulatory
      networks from time-series expression profiles'
  source_content_hash: sha256:7b89c56f40f573c0ef323627df06a8aead907e53042e61f35e3a0bce3835066f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/free1234hm/tsminer-tool
  repo_url: https://github.com/free1234hm/tsminer-tool.git
  tools:
  - Java
  - TSMiner
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# java-application-execution-and-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Launch and configure the Java-based TSMiner executable (TSMiner.jar) from the command line with the required heap allocation and input files, following the settings in TSMiner manual.pdf to construct time-specific regulatory networks.

## When to use

Use when a scientific workflow requires running a packaged Java tool (e.g. TSMiner.jar) that needs Java 1.5+, a 1024 MB heap, and manual-specified input/parameter files on a platform-independent system.

## When NOT to use

- Input is not time-series expression data — TSMiner is specifically designed to construct time-specific regulatory networks from time-series expression profiles, and clustering methods that cannot evaluate temporal dynamics of regulators are a different analysis
- Available memory cannot provide the 1024 MB minimum Java heap, or Java 1.5+ is not installed and cannot be installed
- The task is signal prediction or cascaded network analysis handled by DREM extensions (iDREM, SDREM) rather than TSMiner's TF–pathway network construction

## Inputs

- TSMiner.jar executable (v1.0.0–v1.2.0)
- Java Runtime Environment 1.5 or later
- Time-series gene expression data file (e.g. mouse LR RNA-seq dataset)
- TF-gene interaction data file
- Gene ontology / KEGG / Reactome pathway annotation files
- Configuration file for bounded case analysis (Case_analysis_of_mouseLR.txt) and Defaults_for_case_analysis.txt
- TSMiner manual.pdf for input/parameter settings

## Outputs

- Time-specific TF–pathway regulatory network model saved as saved_model.txt in the 'Saved model' output location
- Interactive network map viewable in TSMiner's visualization interface

## How to apply

Verify Java 1.5 or later is installed and that at least 1024 MB of Java heap memory is available, since TSMiner's core algorithms and interfaces are written in Java. Launch the tool from the command line with 'java -mx1024M -jar TSMiner.jar' (or double-click TSMiner.jar on Microsoft Windows). Supply the three input types — the time-series expression data, TF-gene interaction data, and Gene ontology/KEGG/Reactome pathway annotation files distributed in the package directories ('Time-series data', 'TF-gene file', 'Gene annotation file') — following the data input and parameter settings described in TSMiner manual.pdf. For a bounded demonstration with default parameters, run 'java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt' (or the Case_analysis_of_mouseLR.cmd file on Windows) and press the 'Run' button, using the defaults listed in Defaults_for_case_analysis.txt. Version choice matters for behavior: v1.1.0+ separately processes positively and negatively correlated TF-TG interactions; v1.2.0 updates the main display interface.

## Related tools

- **TSMiner** (Java application being launched and configured; constructs time-specific regulatory networks from time-series expression profiles) — https://github.com/free1234hm/tsminer-tool.git
- **Java** (Runtime environment required to execute TSMiner.jar; language in which TSMiner's core algorithms and interfaces are written)

## Examples

```
java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt
```

## Evaluation signals

- TSMiner.jar launches without OutOfMemoryError, confirming the 1024 MB heap (-mx1024M) allocation is honored
- The bounded case analysis completes when the 'Run' button is pressed with defaults from Defaults_for_case_analysis.txt
- The resulting time-specific regulatory network is saved as saved_model.txt in the 'Saved model' output location
- The saved model loads successfully in the TSMiner interactive network map viewer, confirming output validity
- The version in use supports the intended behavior (v1.1.0+ for separate processing of positively/negatively correlated TF-TG interactions)

## Limitations

- Requires Java 1.5 or later and a minimum of 1024 MB Java heap memory; platform independent but packaged as a jar
- Data input and parameter settings are only documented in TSMiner manual.pdf; no changelog was found in the repository
- The README's 'Download link:' entry is empty — the tool must be obtained from the GitHub repository (https://github.com/free1234hm/tsminer-tool.git)
- Users must review and agree to the DREM (GPL v3.0), Piccolo (BSD), and Batik (Apache) licenses bundled in the license folder before using TSMiner

## Evidence
