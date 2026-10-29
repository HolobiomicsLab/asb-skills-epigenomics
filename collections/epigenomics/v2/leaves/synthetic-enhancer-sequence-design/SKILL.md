---
name: synthetic-enhancer-sequence-design
description: Use when BOM's save_shap_multi() SHAP scores identify the most predictive
  motifs for two conditions and you need concrete synthetic enhancer DNA sequences
  for reporter/MPRA validation of cell-type-specific activity.
when_to_use_negative:
- When motif grammar (order, orientation, or spacing) is expected to matter — BOM's
  representation and the random implantation procedure are explicitly unordered
- When the analysis goal is predicting activity of native regulatory elements rather
  than experimentally validating BOM's motif predictions with synthetic constructs
- When informative sequences are not captured by current TF motif annotations, since
  the design is limited to known PWMs from the provided motif set
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0749
tools:
- name: Bag-of-Motifs (BOM)
  role: Produces the SHAP importance scores (via save_shap_multi()) used to select
    the top-5 predictive motifs implanted into each SRE
  repo: https://github.com/ewonglab/BOM_package
- name: FIMO
  role: FIMO-style PWM-to-sequence matching is used to instantiate concrete motif
    sequences from the gimme.vertebrate.v5.0.meme PWMs
  repo: https://meme-suite.org/meme/doc/install.html
- name: Gimme Maelstrom / gimmemotifs motif set
  role: gimme.vertebrate.v5.0.meme provides the position weight matrices from which
    motif sequences are matched
  repo: https://github.com/vanheeringen-lab/gimmemotifs
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-65362-2
    title: Motif-based models accurately predict cell type-specific distal regulatory
      elements
  source_content_hash: sha256:dce4eb9715d1bee5c8222e8405880b6d8f026c808f88d27e5a0cb209609e7766
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ewonglab/BOM_package
  tools:
  - Bag-of-Motifs (BOM)
  - FIMO
  - Gimme Maelstrom / gimmemotifs motif set
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# synthetic-enhancer-sequence-design

## Summary

Design synthetic regulatory elements (SREs) by implanting concrete motif sequences for the top-5 SHAP-ranked BOM motifs per cell condition into a common enhancer template, to experimentally test whether the predicted motif sets confer cell-type-specific enhancer activity.

## When to use

Use when BOM's save_shap_multi() SHAP scores identify the most predictive motifs for two conditions and you need concrete synthetic enhancer DNA sequences for reporter/MPRA validation of cell-type-specific activity.

## When NOT to use

- When motif grammar (order, orientation, or spacing) is expected to matter — BOM's representation and the random implantation procedure are explicitly unordered
- When the analysis goal is predicting activity of native regulatory elements rather than experimentally validating BOM's motif predictions with synthetic constructs
- When informative sequences are not captured by current TF motif annotations, since the design is limited to known PWMs from the provided motif set

## Inputs

- Per-condition SHAP values from BOM's save_shap_multi() output (for ranking top-5 motifs per cell type)
- Position weight matrices from gimme.vertebrate.v5.0.meme (gimme.vertebrate v5.0 motif set)
- Reference genome sequence for the 260 bp muscle-enhancer template at chr2:211,153,192-211,153,452

## Outputs

- designed_SREs.fasta containing exactly 11 records: 10 designed SRE sequences (5 motifs x 2 cell types) plus the unmodified 260 bp template sequence

## How to apply

Load the per-condition SHAP values produced by save_shap_multi() and rank motifs to select the top-5 predictive motifs separately for each cell type (e.g., HepG2 and GM12878). Retrieve the 260 bp template enhancer sequence (e.g., chr2:211,153,192-211,153,452) from the reference genome. For each selected motif, instantiate a concrete sequence by matching its position weight matrix from the gimme.vertebrate.v5.0.meme file (FIMO-style PWM-to-sequence matching). For each of the 10 designed constructs (5 motifs x 2 cell types), insert two copies of each assigned motif at random positions and in random order into the template backbone, preserving valid nucleotide sequence. The rationale is that BOM represents regulatory elements as unordered motif counts, so disorder of implanted motifs is intentional; synthetic enhancers built this way were shown to drive cell-type-specific expression.

## Related tools

- **Bag-of-Motifs (BOM)** (Produces the SHAP importance scores (via save_shap_multi()) used to select the top-5 predictive motifs implanted into each SRE) — https://github.com/ewonglab/BOM_package
- **FIMO** (FIMO-style PWM-to-sequence matching is used to instantiate concrete motif sequences from the gimme.vertebrate.v5.0.meme PWMs) — https://meme-suite.org/meme/doc/install.html
- **Gimme Maelstrom / gimmemotifs motif set** (gimme.vertebrate.v5.0.meme provides the position weight matrices from which motif sequences are matched) — https://github.com/vanheeringen-lab/gimmemotifs

## Examples

```
library(BagOfMotifs); save_shap_multi()  # then rank per-condition SHAP values and implant top-5 PWM-matched motifs into the 260 bp template to write designed_SREs.fasta
```

## Evaluation signals

- designed_SREs.fasta exists and is in valid FASTA format
- The file contains exactly 11 records: 10 designed SREs plus the unmodified template sequence
- Each designed SRE contains two instances of each of its five assigned (top-5 SHAP-ranked) motifs
- The assigned motif sets differ between the two cell types, matching the separately ranked top-5 SHAP motifs per condition
- The unmodified 260 bp template record matches the reference genome coordinates of the backbone (e.g., chr2:211,153,192-211,153,452)

## Limitations

- BOM's reliance on known TF motifs neglects informative sequences not captured by current motif annotations
- TFs do not bind all their motifs, so implanted PWM-matched sequences may not be bound in vivo
- BOM is constrained to cis-regulatory information within the input sequence, and the random implantation ignores motif grammar/order which may affect enhancer activity

## Evidence

- [abstract] Synthetic enhancers built from the most predictive motifs drive cell-type-specific expression: "We validate BOM’s predictions experimentally by constructing synthetic enhancers from the most predictive motifs, demonstrating that these motif sets drive cell-type-specific expression."
- [methods] gimme.vertebrate.v5.0.meme motif set path: "motifs_path <- paste0(extdata_path,"/gimme.vertebrate.v5.0.meme")"
- [readme] SHAP estimation step feeding motif ranking: "# Estimate SHAP 
save_shap_multi()"
