---
name: footnoted-qualifier-annotation
description: Decision support for comparing workflow management systems across setup,
  workflow design, workflow execution, and integration categories when the source
  figure attaches footnotes to individual feature cells.
when_to_use_negative:
- The comparison figure contains no footnotes or qualifier annotations, so plain feature
  marks suffice without this skill's qualifier-attachment step
- The goal is a runtime benchmark of WMSs rather than a static feature-table transcription
  from a published figure
- Input is already a structured comparison table — re-extracting it from the figure
  would duplicate existing data
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0769
tools:
- name: Watchdog
  role: Subject WMS being compared; its features (GUI workflow construction, distributed
    execution, error detection) anchor one row of the table
  repo: https://github.com/klugem/watchdog
- name: Galaxy
  role: Comparison WMS whose feature marks and qualifiers are transcribed from the
    figure
- name: KNIME
  role: Comparison WMS whose qualified cell (paid extensions required for SGE/dedicated-server
    execution) must retain its footnote text
- name: Snakemake
  role: Comparison WMS whose python-based rule/dependency features are transcribed
    from the figure
- name: Java
  role: Implementation language of Watchdog; relevant to setup-category features such
    as platform independence
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-018-2107-4
    title: Watchdog – a workflow management system for the distributed analysis of
      large-scale experimental data
  source_content_hash: sha256:bb92cfbe4d5a8d68f8985f58b33ad1cdce052124a104bd6772f22f6e6a590535
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
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# footnoted-qualifier-annotation

## Summary

Method for reproducing a published feature-comparison table (Watchdog vs. Galaxy, KNIME, Snakemake, Nextflow in Fig. 12) by transcribing check/cross/partial marks per system and category and preserving footnoted qualifier strings so qualified capabilities are not flattened to plain yes/no values.

## When to use

Decision support for comparing workflow management systems across setup, workflow design, workflow execution, and integration categories when the source figure attaches footnotes to individual feature cells.

## When NOT to use

- The comparison figure contains no footnotes or qualifier annotations, so plain feature marks suffice without this skill's qualifier-attachment step
- The goal is a runtime benchmark of WMSs rather than a static feature-table transcription from a published figure
- Input is already a structured comparison table — re-extracting it from the figure would duplicate existing data

## Inputs

- Published feature-comparison figure (Fig. 12) with check/cross/partial marks and footnotes
- Footnote text accompanying the figure (qualifier strings per system/category)
- List of compared workflow management systems (Watchdog, Galaxy, KNIME, Snakemake, plus the fifth WMS in the figure)

## Outputs

- wms_feature_comparison.csv with columns system, category, feature, value, qualifier
- One row per workflow management system covering setup, workflow design, workflow execution, and integration categories

## How to apply

Read the source figure (Fig. 12 of the Watchdog paper) together with its footnotes, extracting each system's feature marks using the figure's check/cross/partial notation under the four categories: setup, workflow design, workflow execution, and integration of new tools. Create one row per workflow management system — Watchdog, Galaxy, KNIME, Snakemake, and the fifth WMS shown in the figure — as row identifiers. Attach the paper's footnoted qualifier strings to the corresponding system/category cells; for example, KNIME's free version lacks distributed execution and paid extensions are required for workflow execution on the SGE or a dedicated server, so that cell must carry this qualifier text rather than a bare cross. Emit the result as wms_feature_comparison.csv with columns system, category, feature, value, qualifier, then verify the CSV parses and covers all five systems across all four categories with every qualified cell retaining its footnote-derived text.

## Related tools

- **Watchdog** (Subject WMS being compared; its features (GUI workflow construction, distributed execution, error detection) anchor one row of the table) — https://github.com/klugem/watchdog
- **Galaxy** (Comparison WMS whose feature marks and qualifiers are transcribed from the figure)
- **KNIME** (Comparison WMS whose qualified cell (paid extensions required for SGE/dedicated-server execution) must retain its footnote text)
- **Snakemake** (Comparison WMS whose python-based rule/dependency features are transcribed from the figure)
- **Java** (Implementation language of Watchdog; relevant to setup-category features such as platform independence)

## Evaluation signals

- Output file wms_feature_comparison.csv parses as valid CSV
- The table contains exactly five WMS rows (Watchdog, Galaxy, KNIME, Snakemake, and the fifth WMS from Fig. 12)
- All four category columns (setup, workflow design, workflow execution, integration) are covered for every system
- Every cell that the figure marks with a footnote carries the verbatim footnote-derived qualifier text, not a plain yes/no value
- Feature marks use the figure's own check/cross/partial notation rather than invented symbols

## Limitations

- Fidelity depends on the published figure and its footnotes; the full figure content is not present in the article's extracted text, so marks must be read from Fig. 12 directly
- Qualified capabilities risk being flattened to plain yes/no if footnote strings are dropped during transcription
- Comparison reflects the 2018 publication state; WMS features (e.g. KNIME paid extensions, Watchdog's Java 11 requirement in current releases) may have changed since

## Evidence

- [full_text] KNIME qualifier finding: [evidence span withheld — anchor status: not_found]
- [abstract] Watchdog Java implementation: "Watchdog is implemented in Java and thus platform-independent"
- [abstract] Watchdog features relevant to comparison categories: "Main features include straightforward processing of replicate data, support for distributed computer systems, customizable error detection and manual intervention into workflow execution."
- [abstract] Watchdog GUI for workflow design category: "It provides a graphical user interface (GUI) for workflow construction using pre-defined modules as well as a helper script for creating new module definitions."
- [intro] Compared WMSs from intro: "Other commonly used WMSs are KNIME [9], an open-source data analysis platform which allows programmers to extend its basic functionality by adding new Java programs"
- [readme] Module creation tooling (integration category): "moduleMaker.jar - provides a graphical user interface for module creation"
