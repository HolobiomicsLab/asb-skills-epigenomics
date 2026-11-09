---
name: tf-activation-repression-time-point-detection
description: Use when you have a time-series expression matrix plus TF-gene interaction
  data and need to determine, per time point, which TFs significantly regulate tree-structure
  subpaths and whether each TF is activated or repressed.
when_to_use_negative:
- Input lacks time-series structure (e.g., single-condition expression profiles) —
  TFs cannot be assigned to specific time points without temporal resolution
- No TF-gene interaction (TF-TG) annotations are available for the organism — both
  positive and negative interaction evidence is required for subpath regulation tests
- Goal is clustering of expression modules without temporal dynamics of regulators
  — clustering methods cannot evaluate the temporal dynamics of regulators or pathways
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: TSMiner
  role: Runs the tree-structure model, hypergeometric plus permutation testing with
    BH correction, and activation/repression classification of TFs at each time point;
    v1.1.0+ separately processes positively and negatively correlated TF-TG interactions
  repo: https://github.com/free1234hm/tsminer-tool.git
- name: Java
  role: 'Runtime environment for TSMiner (Java 1.5 or later, 1024 MB heap: java -mx1024M
    -jar TSMiner.jar)'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab629
    title: 'TSMiner: a novel framework for generating time-specific gene regulatory
      networks from time-series expression profiles'
  source_content_hash: sha256:fdde2ab921f4927b000230fe1aa48f9a999fcb895eba03bda4fcc4cb5863b4c4
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
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# tf-activation-repression-time-point-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

TSMiner's TF prediction module identifies which transcription factors (TFs) significantly regulate each subpath of the tree-structure model at each time point, using a hypergeometric test plus a permutation test, and then labels each TF as activated or repressed based on the average fold change of its target genes.

## When to use

Use when you have a time-series expression matrix plus TF-gene interaction data and need to determine, per time point, which TFs significantly regulate tree-structure subpaths and whether each TF is activated or repressed.

## When NOT to use

- Input lacks time-series structure (e.g., single-condition expression profiles) — TFs cannot be assigned to specific time points without temporal resolution
- No TF-gene interaction (TF-TG) annotations are available for the organism — both positive and negative interaction evidence is required for subpath regulation tests
- Goal is clustering of expression modules without temporal dynamics of regulators — clustering methods cannot evaluate the temporal dynamics of regulators or pathways

## Inputs

- Mouse LR time-series RNA-seq expression data (e.g., GSE95135)
- TF-gene interaction data file (positively and negatively correlated interactions)
- Gene ontology / KEGG / Reactome pathway annotation files from the 'Gene annotation file' directory
- Default parameters from Defaults_for_case_analysis.txt

## Outputs

- Time-specific regulatory network of TFs per subpath per time point (saved_model.txt)
- TF activation/repression classification per time point (e.g., 389 activators and 49 repressors across the LR process)
- TF-pathway interactions (e.g., 109 and 47 KEGG pathways interacting with activators and repressors)

## How to apply

Load the time-series expression data, TF-gene interaction data, and Gene ontology/KEGG/Reactome pathway annotation files into TSMiner (Java 1.5+, 1024 MB heap) and press 'Run' so genes are assigned to tree-structure subpaths. For each subpath, TSMiner scores candidate TFs with a hypergeometric test and a permutation test, accepting a TF at a time point only when the permutation-test P-value remains significant after Benjamini-Hochberg (BH) correction. Positively and negatively correlated TF-TG interactions are processed separately, so activation and repression signals are evaluated in independent passes. A TF passing significance is classified as a transcriptional activator or repressor based on the average fold change of its target genes at that time point. Save the resulting time-specific regulatory network as saved_model.txt and confirm it reloads in TSMiner's interactive network map viewer.

## Related tools

- **TSMiner** (Runs the tree-structure model, hypergeometric plus permutation testing with BH correction, and activation/repression classification of TFs at each time point; v1.1.0+ separately processes positively and negatively correlated TF-TG interactions) — https://github.com/free1234hm/tsminer-tool.git
- **Java** (Runtime environment for TSMiner (Java 1.5 or later, 1024 MB heap: java -mx1024M -jar TSMiner.jar))

## Examples

```
java -mx1024M -jar TSMiner.jar -d Case_analysis_of_mouseLR.txt
```

## Evaluation signals

- A TF is assigned to a time point only if its permutation-test P-value is significant after BH correction — check that uncorrected or hypergeometric-only hits are not reported
- Each predicted TF at a time point carries an activated or repressed label derived from the average fold change of its target genes
- Positively and negatively correlated TF-TG interaction results are reported separately; expect fewer hits from negative interactions because known repressors are limited
- Saved model file (saved_model.txt) loads successfully in TSMiner's interactive network map viewer
- On the mouse LR case study, the classifier recovers known LR-associated TFs with higher sensitivity/accuracy than DREM v2.0 and four clustering tools

## Limitations

- Fewer results from negative TF-TG interactions than positive ones because known repressors of many genes are limited; future collection of negatively correlated TF-TG interactions is needed
- Precise regulatory mechanisms of the predicted activation/repression events require more rigorous evaluation beyond the experimental validation reported
- DREM-based tree-structure training dependence: TSMiner addresses DREM's lack of effective initialization, but results still depend on the supplied TF-gene and pathway annotation inputs

## Evidence

- [methods] TF assigned to time points via BH-corrected permutation-test P-values and classified by average fold change of target genes: [evidence span withheld — anchor status: not_found]
