---
name: scientific-figure-data-extraction
description: Use when a published article presents a qualitative feature comparison
  as a figure (e.g., Fig. 12 comparing WMSs across setup, workflow design, workflow
  execution, and integration) that must be reproduced as structured table data with
  footnote-derived qualifiers intact.
when_to_use_negative:
- The article already provides the comparison as a parseable table or supplementary
  CSV — then transcribe directly rather than re-extracting from a figure
- The figure requires numeric data extraction or plotting digitization — this skill
  covers qualitative check/cross/partial marks with text qualifiers, not curve or
  point extraction
- The goal is to evaluate or install a WMS rather than reproduce its published comparison
  content — use the individual WMS repositories instead
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Watchdog
  role: Subject WMS compared in Fig. 12; its published feature marks across setup,
    workflow design, execution, and integration are extracted
  repo: https://github.com/klugem/watchdog
- name: Galaxy
  role: Compared WMS in Fig. 12; feature marks and qualifiers extracted from the figure
- name: KNIME
  role: Compared WMS in Fig. 12; its missing distributed execution and paid SGE/dedicated-server
    extensions appear as qualified cells
- name: Snakemake
  role: Compared WMS in Fig. 12; feature marks and qualifiers extracted from the figure
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-018-2107-4
    title: Watchdog – a workflow management system for the distributed analysis of
      large-scale experimental data
  source_content_hash: sha256:973d4ab62e9b428ddecd2f1f836e9ab6766f6a7ee33ba72ec3b7808ea8540753
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# scientific-figure-data-extraction

## Summary

Transcribes a published article's figure-embedded comparison data (here, Fig. 12 of the Watchdog WMS paper comparing Watchdog, Galaxy, KNIME, Snakemake, and a fifth WMS) into a structured, machine-readable CSV while preserving check/cross/partial marks and footnote qualifier annotations instead of flattening qualified capabilities to plain yes/no values.

## When to use

Use when a published article presents a qualitative feature comparison as a figure (e.g., Fig. 12 comparing WMSs across setup, workflow design, workflow execution, and integration) that must be reproduced as structured table data with footnote-derived qualifiers intact.

## When NOT to use

- The article already provides the comparison as a parseable table or supplementary CSV — then transcribe directly rather than re-extracting from a figure
- The figure requires numeric data extraction or plotting digitization — this skill covers qualitative check/cross/partial marks with text qualifiers, not curve or point extraction
- The goal is to evaluate or install a WMS rather than reproduce its published comparison content — use the individual WMS repositories instead

## Inputs

- Published article figure containing a qualitative feature comparison matrix (Watchdog paper Fig. 12, WMS comparison)
- Figure footnotes containing qualifier annotations for specific cells
- Article section text identifying the compared systems (Watchdog, Galaxy, KNIME, Snakemake, plus the fifth WMS in the figure)

## Outputs

- wms_feature_comparison.csv with columns system, category, feature, value, qualifier
- One row per workflow management system per feature category, with check/cross/partial values and footnote-derived qualifier text

## How to apply

Read the target figure (Watchdog paper Fig. 12) together with its footnotes, since the check/cross/partial marks carry meaning only when combined with the qualifier strings. Create one row per workflow management system (Watchdog, Galaxy, KNIME, Snakemake, and the fifth WMS shown in the figure) as row identifiers. Encode each feature assessment using the figure's own check/cross/partial notation, organized under the four category columns: setup, workflow design, workflow execution, and integration. Attach the paper's footnoted qualifier strings to the corresponding system/category cells so that capabilities like KNIME's paid extensions for SGE or dedicated-server execution are not flattened to plain yes/no. Write the assembled comparison as wms_feature_comparison.csv with the columns system, category, feature, value, qualifier.

## Related tools

- **Watchdog** (Subject WMS compared in Fig. 12; its published feature marks across setup, workflow design, execution, and integration are extracted) — https://github.com/klugem/watchdog
- **Galaxy** (Compared WMS in Fig. 12; feature marks and qualifiers extracted from the figure)
- **KNIME** (Compared WMS in Fig. 12; its missing distributed execution and paid SGE/dedicated-server extensions appear as qualified cells)
- **Snakemake** (Compared WMS in Fig. 12; feature marks and qualifiers extracted from the figure)

## Evaluation signals

- Output is a parseable CSV with exactly the columns system, category, feature, value, qualifier
- Exactly five WMS rows/row-identifier sets are present: Watchdog, Galaxy, KNIME, Snakemake, and the fifth WMS compared in Fig. 12
- All four category columns are covered: setup, workflow design, workflow execution, and integration
- Every cell that carries a footnote in the published figure carries the corresponding qualifier string rather than a plain yes/no value (e.g., KNIME's free version lacking distributed execution, paid extensions required for SGE or dedicated-server execution)
- Feature values use the figure's own check/cross/partial notation, not a re-invented scale

## Limitations

- Extraction depends on figure legibility and the completeness of the published footnotes; a qualifier omitted in the source figure cannot be recovered
- The comparison is qualitative (check/cross/partial); no quantitative metrics or version numbers are encoded, and the article provides no specific Watchdog release version
- Qualified capabilities must not be flattened to yes/no — doing so misrepresents, e.g., KNIME's paywalled distributed execution support

## Evidence

- [other] Read the article's Fig. 12 together with its footnotes to extract the per-system feature marks and qualifier annotations.: [evidence span withheld — anchor status: not_found]
- [other] KNIME paid extensions finding: [evidence span withheld — anchor status: not_found]
- [intro] Compared systems in article: "Other commonly used WMSs are KNIME [9], an open-source data analysis platform which allows programmers to extend its basic functionality by adding new Java programs"
- [intro] Snakemake description: "and Snakemake [10], a python-based WMS. Snakemake allows definition of tasks based on rules and automatically infers dependencies between tasks by matching filenames."
- [intro] Galaxy description: "The most well-known example, Galaxy, was initially developed to enable experimentalists without programming experience to perform genomic data analyses in the web browser"
