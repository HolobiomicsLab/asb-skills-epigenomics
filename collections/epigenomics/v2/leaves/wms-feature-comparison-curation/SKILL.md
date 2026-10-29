---
name: wms-feature-comparison-curation
description: Use when a task requires reproducing or auditing the published feature
  comparison of workflow management systems (Watchdog vs. Galaxy, KNIME, Snakemake)
  from Fig. 12 of the Watchdog article, including its footnote qualifiers such as
  KNIME's paid-extension requirement.
when_to_use_negative:
- The task asks for runtime or performance benchmarks of WMSs rather than the qualitative
  feature matrix of Fig. 12
- The source figure is a different (non-Fig. 12) comparison whose footnotes and system
  set do not match the five-system, four-category layout this skill expects
- The user needs a machine-actionable workflow definition (e.g., a Watchdog XML workflow)
  rather than a curated comparison table
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Watchdog
  role: Subject of the comparison; Java-based WMS whose setup, design, execution,
    and module-integration features are encoded in the matrix
  repo: https://github.com/klugem/watchdog
- name: Galaxy
  role: Compared WMS in Fig. 12; web-browser-based system for experimentalists without
    programming experience
- name: KNIME
  role: Compared WMS in Fig. 12; open-source data analysis platform whose distributed-execution
    limitation and paid-extension qualifier must be captured
- name: Snakemake
  role: Compared WMS in Fig. 12; python-based rule system that infers task dependencies
    by matching filenames
- name: Java
  role: Watchdog's implementation language; grounds the setup-category entries (JDK/JavaFX
    requirements) in the comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-018-2107-4
    title: Watchdog – a workflow management system for the distributed analysis of
      large-scale experimental data
  source_content_hash: sha256:87e768da5374cb100ed699af8b37b9255d14381c8693de0926aa301313147932
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/klugem/watchdog
  tools:
  - Galaxy
  - Java
  - KNIME
  - Snakemake
  - Watchdog
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# wms-feature-comparison-curation

## Summary

Curates the Watchdog article's WMS feature comparison (Fig. 12) into a structured CSV matrix that compares Watchdog against Galaxy, KNIME, Snakemake, and one additional WMS across setup, workflow design, workflow execution, and integration of new tools, preserving footnote-derived qualifiers instead of flattening capabilities to yes/no.

## When to use

Use when a task requires reproducing or auditing the published feature comparison of workflow management systems (Watchdog vs. Galaxy, KNIME, Snakemake) from Fig. 12 of the Watchdog article, including its footnote qualifiers such as KNIME's paid-extension requirement.

## When NOT to use

- The task asks for runtime or performance benchmarks of WMSs rather than the qualitative feature matrix of Fig. 12
- The source figure is a different (non-Fig. 12) comparison whose footnotes and system set do not match the five-system, four-category layout this skill expects
- The user needs a machine-actionable workflow definition (e.g., a Watchdog XML workflow) rather than a curated comparison table

## Inputs

- Published Fig. 12 feature comparison from the Watchdog article (BMC Bioinformatics, DOI 10.1186/s12859-018-2107-4) including its footnotes
- Article text describing the compared WMSs (Watchdog, Galaxy, KNIME, Snakemake)
- Target file specification: wms_feature_comparison.csv with columns system, category, feature, value, qualifier

## Outputs

- wms_feature_comparison.csv — parseable CSV with exactly five WMS rows covering all four categories (setup, workflow design, workflow execution, integration), check/cross/partial values, and footnote-derived qualifier text

## How to apply

Read Fig. 12 of the Watchdog article together with its footnotes, extracting the per-system marks (check/cross/partial notation) under the four category columns: setup, workflow design, workflow execution, and integration of new tools. Create one row per WMS (Watchdog, Galaxy, KNIME, Snakemake, and the fifth WMS shown in the figure) as row identifiers. Encode each assessment using the figure's own notation, and attach the footnote qualifier strings to qualified cells — e.g., KNIME's free version lacks distributed execution and paid extensions are required for workflow execution on the SGE or on a dedicated server — so qualified capabilities are not flattened to plain yes/no. Write the assembled matrix to wms_feature_comparison.csv with columns system, category, feature, value, qualifier. Validate that the output is a parseable CSV with exactly five WMS rows covering all four categories and that every qualified cell carries footnote-derived qualifier text matching the published figure.

## Related tools

- **Watchdog** (Subject of the comparison; Java-based WMS whose setup, design, execution, and module-integration features are encoded in the matrix) — https://github.com/klugem/watchdog
- **Galaxy** (Compared WMS in Fig. 12; web-browser-based system for experimentalists without programming experience)
- **KNIME** (Compared WMS in Fig. 12; open-source data analysis platform whose distributed-execution limitation and paid-extension qualifier must be captured)
- **Snakemake** (Compared WMS in Fig. 12; python-based rule system that infers task dependencies by matching filenames)
- **Java** (Watchdog's implementation language; grounds the setup-category entries (JDK/JavaFX requirements) in the comparison)

## Examples

```
python -c "import pandas as pd; df = pd.read_csv('wms_feature_comparison.csv'); assert set(df.category) == {'setup','workflow design','workflow execution','integration'}; df.to_csv('wms_feature_comparison.csv', index=False)"
```

## Evaluation signals

- Output is a parseable CSV named wms_feature_comparison.csv with columns system, category, feature, value, qualifier
- Exactly five WMS rows (Watchdog, Galaxy, KNIME, Snakemake, plus the fifth WMS from Fig. 12), each covering all four categories
- Every qualified cell carries footnote-derived qualifier text, e.g., the KNIME workflow-execution entry notes that paid extensions are required for SGE or dedicated-server execution
- Values use the figure's own check/cross/partial notation rather than normalized yes/no
- Qualifier strings match the published Fig. 12 footnote content verbatim in meaning

## Limitations

- Coverage is limited to the four categories and the five systems shown in Fig. 12; features absent from the figure cannot be curated
- No specific version number is stated for the Watchdog software release in the article, so version-pinning the comparison is not possible
- Flattening qualified cells to plain yes/no loses the footnote distinctions (e.g., capability available only via paid KNIME extensions)

## Evidence

- [full_text] KNIME qualifier on distributed execution: [evidence span withheld — anchor status: not_found]
- [intro] Compared WMS landscape: "The most well-known example, Galaxy, was initially developed to enable experimentalists without programming experience to perform genomic data analyses in the web browser"
- [intro] KNIME extension mechanism: "Other commonly used WMSs are KNIME [9], an open-source data analysis platform which allows programmers to extend its basic functionality by adding new Java programs"
- [readme] Watchdog setup requirements grounding: "Watchdog is written in Java and requires JDK 11 or higher."
