---
name: structured-table-authoring
description: Use when a published figure or table encodes per-system feature assessments
  (e.g., check/cross/partial notation with footnoted qualifiers) and the goal is to
  reproduce it as a parseable CSV with one row per system and explicit qualifier text.
when_to_use_negative:
- The source table has no footnoted qualifiers and encodes only plain yes/no marks
  — a direct transcription suffices without this qualifier-preserving scheme
- The comparison target is quantitative benchmark data (runtimes, resource usage)
  rather than categorical feature assessments
- The table schema required by the downstream consumer differs from system/category/feature/value/qualifier
  and reshaping would lose the figure's row-per-system structure
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3071
tools:
- name: Watchdog
  role: Subject system whose Fig. 12 feature marks (GUI workflow construction, distributed
    execution via distributed computer systems, error detection with manual intervention)
    are transcribed into the comparison table
  repo: https://github.com/klugem/watchdog
- name: Galaxy
  role: Compared WMS whose per-category marks and footnote qualifiers are extracted
    as table rows
- name: KNIME
  role: Compared WMS whose qualified cells (free version lacks distributed execution;
    paid extensions required for SGE or dedicated server execution) must carry qualifier
    text
- name: Snakemake
  role: Compared WMS whose per-category marks and footnote qualifiers are extracted
    as table rows
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-018-2107-4
    title: Watchdog – a workflow management system for the distributed analysis of
      large-scale experimental data
  source_content_hash: sha256:d4d70dbe71b1756e386d113ba2a83e819ed68105d31080ce3a5a7c597635c436
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/klugem/watchdog
  tools:
  - Galaxy
  - KNIME
  - Snakemake
  - Watchdog
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3071
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# structured-table-authoring

## Summary

Assembles a machine-readable comparison table (wms_feature_comparison.csv) that reproduces the Watchdog article's Fig. 12 feature comparison of Watchdog, Galaxy, KNIME, Snakemake, and a fifth WMS across setup, workflow design, workflow execution, and integration of new tools, preserving check/cross/partial marks and footnote qualifiers rather than flattening qualified capabilities to plain yes/no.

## When to use

Use when a published figure or table encodes per-system feature assessments (e.g., check/cross/partial notation with footnoted qualifiers) and the goal is to reproduce it as a parseable CSV with one row per system and explicit qualifier text.

## When NOT to use

- The source table has no footnoted qualifiers and encodes only plain yes/no marks — a direct transcription suffices without this qualifier-preserving scheme
- The comparison target is quantitative benchmark data (runtimes, resource usage) rather than categorical feature assessments
- The table schema required by the downstream consumer differs from system/category/feature/value/qualifier and reshaping would lose the figure's row-per-system structure

## Inputs

- Published comparison figure/table with per-system feature marks (Watchdog article Fig. 12) including footnotes
- List of workflow management systems to include as rows (Watchdog, Galaxy, KNIME, Snakemake, plus the fifth WMS in the figure)
- Category taxonomy from the figure: setup, workflow design, workflow execution, integration of new tools

## Outputs

- wms_feature_comparison.csv with columns system, category, feature, value, qualifier
- Exactly five WMS rows covering all four categories (setup, workflow design, workflow execution, integration)

## How to apply

Read the source figure (Fig. 12 of the Watchdog WMS comparison) together with its footnotes to extract per-system feature marks and the qualifier strings attached to qualified cells. Create one row per workflow management system — Watchdog, Galaxy, KNIME, Snakemake, and the fifth WMS shown in the figure — and encode each assessment using the figure's check/cross/partial notation under the four category columns: setup, workflow design, workflow execution, and integration. Crucially, attach each paper's footnoted qualifier text (e.g., that KNIME's free version lacks distributed execution and paid extensions are required for execution on SGE or a dedicated server) to the corresponding system/category cell so qualified capabilities are not flattened to plain yes/no values. Write the result as a CSV with columns system, category, feature, value, qualifier. Rationale: the article explicitly qualifies feature marks via footnotes, so dropping qualifiers would misrepresent the published comparison.

## Related tools

- **Watchdog** (Subject system whose Fig. 12 feature marks (GUI workflow construction, distributed execution via distributed computer systems, error detection with manual intervention) are transcribed into the comparison table) — https://github.com/klugem/watchdog
- **Galaxy** (Compared WMS whose per-category marks and footnote qualifiers are extracted as table rows)
- **KNIME** (Compared WMS whose qualified cells (free version lacks distributed execution; paid extensions required for SGE or dedicated server execution) must carry qualifier text)
- **Snakemake** (Compared WMS whose per-category marks and footnote qualifiers are extracted as table rows)

## Evaluation signals

- Output file is a parseable CSV with exactly the columns system, category, feature, value, qualifier
- Exactly five distinct WMS rows (Watchdog, Galaxy, KNIME, Snakemake, plus the fifth WMS from Fig. 12) each covering all four categories
- Every cell that carries a footnote in Fig. 12 has non-empty qualifier text matching the published footnote wording
- No qualified capability is flattened to a bare check or cross without its qualifier string
- Value column uses the figure's own check/cross/partial notation rather than invented encodings

## Limitations

- The fifth WMS in Fig. 12 is not named in the extracted article text and must be identified from the figure itself; omitting it breaks the five-row validation requirement
- Footnote qualifier text must be transcribed verbatim from the figure; the extracted entity index alone does not contain the full Fig. 12 contents, so source-figure access is required
- Feature marks reflect the 2018 published comparison; later software releases (e.g., KNIME or Snakemake updates) may invalidate specific cells

## Evidence

- [results] KNIME qualifier finding: [evidence span withheld — anchor status: not_found]
- [abstract] Watchdog feature set: "Main features include straightforward processing of replicate data, support for distributed computer systems, customizable error detection and manual intervention into workflow execution."
- [intro] WMS list compared: "Other commonly used WMSs are KNIME [9], an open-source data analysis platform which allows programmers to extend its basic functionality by adding new Java programs"
- [intro] Snakemake description: "and Snakemake [10], a python-based WMS. Snakemake allows definition of tasks based on rules and automatically infers dependencies between tasks by matching filenames."
- [readme] Watchdog execution modes relevant to feature table: "Execution of workflows is possible using either the GUI or a command-line interface"
