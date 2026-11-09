---
name: tf-motif-enrichment-analysis
description: Use when you have two or more sets of differentially bound calling cards
  peaks (e.g., female vs male GBM CC differential peaks, BED/qbed-derived) and need
  to find TF motif families enriched in one group but not another.
when_to_use_negative:
- Input is raw qbed/ccf insertions rather than called, differential peaks — motif
  calling should follow peak calling, differential peak analysis, and peak filtering
  (pp.call_peaks, pp.filter_peaks) first
- The goal is TF binding site discovery itself, not motif enrichment comparison —
  use the Pycallingcards peak callers (CCcaller, MACCs) instead
- Peaks are from ChIP-seq or RNA-seq data rather than calling cards assays; this skill's
  workflow is grounded in CC peak inputs
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: pycallingcards
  role: Provides tools.call_motif (HOMER-based de novo motif discovery on peak sequences)
    and tools.compare_motif (between-group motif enrichment comparison) within the
    tl module
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: HOMER
  role: Motif discovery engine invoked by cc.tools.call_motif to find motifs enriched
    in differential peak sequences
- name: pandas
  role: Holds the female/male differentially bound peak DataFrames that are passed
    into call_motif
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae070
    title: 'Pycallingcards: an integrated environment for visualizing, analyzing,
      and interpreting Calling Cards data'
  source_content_hash: sha256:2b864b4faa09b0ba7fcec2fcb8450209b0c52f7ff60d60854e6966d02d5e2357
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/The-Mitra-Lab/pycallingcards
  tools:
  - HOMER
  - pandas
  - pycallingcards
  - tools.call_motif
  - tools.compare_motif
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# tf-motif-enrichment-analysis

## Summary

De novo and comparative TF motif enrichment on calling cards (CC) peaks, using Pycallingcards' tools module (call_motif via HOMER) to discover motifs in differentially bound peak sets and compare_motif to identify sex- or condition-specific motif families.

## When to use

Use when you have two or more sets of differentially bound calling cards peaks (e.g., female vs male GBM CC differential peaks, BED/qbed-derived) and need to find TF motif families enriched in one group but not another.

## When NOT to use

- Input is raw qbed/ccf insertions rather than called, differential peaks — motif calling should follow peak calling, differential peak analysis, and peak filtering (pp.call_peaks, pp.filter_peaks) first
- The goal is TF binding site discovery itself, not motif enrichment comparison — use the Pycallingcards peak callers (CCcaller, MACCs) instead
- Peaks are from ChIP-seq or RNA-seq data rather than calling cards assays; this skill's workflow is grounded in CC peak inputs

## Inputs

- Differentially bound peak sets (e.g., female and male GBM Brd4 CC peaks) as pandas DataFrames
- Peak sequences derived from calling cards qbed/peak data
- HOMER motif discovery backend

## Outputs

- Per-group enriched motif family lists (e.g., IRF, ETS, TEA, MADS for female GBM CC peaks)
- Cross-group motif enrichment comparison table written to motif_enrichment_comparison.csv
- Sex-specific (or condition-specific) TF family enrichment calls with discrepancy flags for expert review

## How to apply

Load the differential peak sets (e.g., female and male differentially bound Brd4 peaks from the GBM calling cards data) into pandas DataFrames. Call motifs enriched in each peak set separately using cc.tools.call_motif, which invokes HOMER for de novo motif discovery on the peak sequences. Then run cc.tools.compare_motif to compare motif enrichment between the groups and identify motif families with sex-specific enrichment. Summarize the enriched motif families per group and write the comparison to motif_enrichment_comparison.csv. Validate against the reported pattern — IRF, ETS, TEA, and MADS families enriched in female but not male CC data, and an extreme HMG (Sox) family difference in male samples (Supplementary Tables S4-S5) — flagging any discrepancies for expert review.

## Related tools

- **pycallingcards** (Provides tools.call_motif (HOMER-based de novo motif discovery on peak sequences) and tools.compare_motif (between-group motif enrichment comparison) within the tl module) — https://github.com/The-Mitra-Lab/pycallingcards
- **HOMER** (Motif discovery engine invoked by cc.tools.call_motif to find motifs enriched in differential peak sequences)
- **pandas** (Holds the female/male differentially bound peak DataFrames that are passed into call_motif)

## Examples

```
import pycallingcards as cc; motifs_f = cc.tools.call_motif(female_differential_peaks); motifs_m = cc.tools.call_motif(male_differential_peaks); comparison = cc.tools.compare_motif(motifs_f, motifs_m)
```

## Evaluation signals

- The female GBM CC comparison shows enrichment of the IRF, ETS, TEA, and MADS motif families that are absent from the male CC results
- The male GBM CC results show an extreme difference in the HMG (Sox) motif family, consistent with Supplementary Tables S4-S5
- motif_enrichment_comparison.csv is produced and contains family-level enrichment columns for both female and male groups
- Each group's motif call was run independently through cc.tools.call_motif on its own differential peak set before compare_motif was applied

## Limitations

- Motif calling depends on HOMER being installed and correctly configured alongside pycallingcards (which also requires bedtools/pybedtools for annotation)
- Validation of family-level patterns is specific to the GBM CC reanalysis; other datasets may show different motif families and require expert review of discrepancies
- The skill compares motif enrichment across pre-computed differential peak sets; it does not itself perform the differential peak analysis

## Evidence

- [abstract] Pycallingcards offers a fully integrated environment for data visualization, motif finding, and comparative analysis with RNA-seq and ChIP-seq datasets.: "Pycallingcards offers a fully integrated environment for data visualization, motif finding, and comparative analysis with RNA-seq and ChIP-seq datasets."
- [methods] **Tools (tl)** calls motif of the peaks, completes differential peaks and pair differential peaks with gene expression,: "**Tools (tl)** calls motif of the peaks, completes differential peaks and pair differential peaks with gene expression,"
- [readme] Pycallingcards is composed of five different part: datasets, reading (rd), preprocessing (pp), tools (tl) and plotting (pl).: "Pycallingcards is composed of five different part: datasets, reading (rd), preprocessing (pp), tools (tl) and plotting (pl)."
- [abstract] Reanalysis of published mouse cortex and glioblastoma datasets revealed novel cell-type-specific binding sites and potential sex-linked TF regulators.: [evidence span withheld — anchor status: not_found]
