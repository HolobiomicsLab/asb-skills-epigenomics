---
name: liver-regeneration-case-analysis-reproduction
description: Use when reproducing the TSMiner mouse LR case analysis (GSE95135 partial
  hepatectomy RNA-seq) or when rebuilding the text-mined reference sets of 113 LR-associated
  TFs and 58 KEGG pathways for sensitivity evaluation.
when_to_use_negative:
- Input expression profiles are not time-series (no ordered time points), so no time-specific
  regulatory network can be constructed
- The goal is a novel regulatory-network discovery for a different organism or perturbation
  rather than reproducing the documented mouse LR case analysis with default parameters
- Only static TF-gene associations are available without pathway gene sets (KEGG/Reactome)
  — TSMiner's TF-pathway interaction output depends on the gene annotation files
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: TSMiner
  role: 'Runs the packaged mouse LR case analysis: loads the time-series expression,
    TF-gene interaction, and GO/KEGG/Reactome annotation inputs, processes positive
    and negative TF-TG correlations separately, and outputs the time-specific regulatory
    network model'
  repo: https://github.com/free1234hm/tsminer-tool.git
- name: Java
  role: Runtime required to execute TSMiner.jar; must be Java 1.5 or later with at
    least 1024 MB heap (-mx1024M)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkab629
    title: 'TSMiner: a novel framework for generating time-specific gene regulatory
      networks from time-series expression profiles'
  source_content_hash: sha256:cfc1d6879390ad1f42ea3bc3808f98750d4df734028f54ecc21baeba1f1e45b2
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# liver-regeneration-case-analysis-reproduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the TSMiner case analysis of mouse liver regeneration (LR): running the packaged example on the mouse LR RNA-seq time-series data and, in parallel, rebuilding the curated reference sets (113 known LR-associated TFs and 58 known LR-associated KEGG pathways) used to benchmark TSMiner's sensitivity against DREM 2.0 and clustering tools.

## When to use

Use when reproducing the TSMiner mouse LR case analysis (GSE95135 partial hepatectomy RNA-seq) or when rebuilding the text-mined reference sets of 113 LR-associated TFs and 58 KEGG pathways for sensitivity evaluation.

## When NOT to use

- Input expression profiles are not time-series (no ordered time points), so no time-specific regulatory network can be constructed
- The goal is a novel regulatory-network discovery for a different organism or perturbation rather than reproducing the documented mouse LR case analysis with default parameters
- Only static TF-gene associations are available without pathway gene sets (KEGG/Reactome) — TSMiner's TF-pathway interaction output depends on the gene annotation files

## Inputs

- Mouse liver regeneration (LR) time-series expression dataset (RNA-seq, GSE95135) in the 'Time-series data' directory format
- TF-gene interaction data from the 'TF-gene file' directory (positive and negatively correlated TF-TG interactions)
- Gene ontology files and KEGG/Reactome pathway files from the 'Gene annotation file' directory
- Default parameter file Defaults_for_case_analysis.txt
- MEDLINE abstracts queried with keywords 'liver regeneration' and 'partial hepatectomy' (for reference-set text mining), plus a defined background gene/pathway set

## Outputs

- Saved time-specific regulatory network model: case_analysis_of_mouseLR_model.txt in the 'Saved model' directory format
- Interactive network map of time-specific activated/repressed TF-pathway regulatory events across the LR time course
- Curated reference sets: 113 known LR-associated TFs and 58 known LR-associated KEGG pathways used for sensitivity/accuracy evaluation

## How to apply

Verify the environment first: Java 1.5 or later installed and at least 1024 MB of Java heap memory available. From the TSMiner directory, launch the packaged example with 'java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt' (or double-click Case_analysis_of_mouseLR.cmd on Windows) and press the 'Run' button in the main interface. Load the three input types per Defaults_for_case_analysis.txt: the mouse LR time-series expression dataset from the 'Time-series data' directory, TF-gene interaction data from the 'TF-gene file' directory, and Gene ontology plus KEGG/Reactome pathway files from the 'Gene annotation file' directory; note that TSMiner v1.1.0+ separately processes positively and negatively correlated TF-TG interactions. Inspect the resulting time-specific regulatory network via the interactive network map visualization, then save it as case_analysis_of_mouseLR_model.txt in the 'Saved model' directory format. To rebuild the benchmark reference sets, text-mine MEDLINE abstracts with the keywords 'liver regeneration' and 'partial hepatectomy' against a defined background set, then manually curate hits to obtain the 113 TFs and 58 KEGG pathways against which sensitivity is scored.

## Related tools

- **TSMiner** (Runs the packaged mouse LR case analysis: loads the time-series expression, TF-gene interaction, and GO/KEGG/Reactome annotation inputs, processes positive and negative TF-TG correlations separately, and outputs the time-specific regulatory network model) — https://github.com/free1234hm/tsminer-tool.git
- **Java** (Runtime required to execute TSMiner.jar; must be Java 1.5 or later with at least 1024 MB heap (-mx1024M))

## Examples

```
java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt
```

## Evaluation signals

- The run completes without error under the default parameter file Defaults_for_case_analysis.txt
- A saved model file named case_analysis_of_mouseLR_model.txt exists in the 'Saved model' directory and is non-empty on disk
- The interactive network map renders time-specific regulatory networks for both positively and negatively correlated TF-TG interaction sets
- Sensitivity is computed against the curated reference sets of 113 known LR-associated TFs and 58 known LR-associated KEGG pathways, allowing comparison with DREM (v. 2.0) and four clustering tools

## Limitations

- Fewer results were obtained from the negative TF-TG interactions than from the positive interactions because the known repressors of many genes were limited
- Requires Java 1.5 or later and a minimum of 1024 MB Java heap memory; the packaged example assumes the file layout of the TSMiner release directories
- Precise regulatory mechanisms require more rigorous evaluation, even though experimental results strongly supported the predictions

## Evidence
