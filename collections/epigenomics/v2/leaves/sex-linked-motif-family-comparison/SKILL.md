---
name: sex-linked-motif-family-comparison
description: Use when you have two sex-stratified sets of differentially bound calling
  cards peaks (e.g. female vs male GBM CC Brd4 peaks) and need to identify TF motif
  families enriched in one sex but not the other.
when_to_use_negative:
- The input is bulk qbed/ccf insertion data that has not yet been peak-called and
  differentially analyzed — run cc preprocessing (clean_qbed, call_peaks, make_Anndata,
  filter_peaks, differential peak analysis) first.
- Only one sex/condition is available — a between-sex comparison via cc.tools.compare_motif
  cannot be performed.
- The goal is single-cell clustering or peak-to-gene expression pairing rather than
  motif family enrichment comparison between peak sets.
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: pycallingcards
  role: Provides tools.call_motif (HOMER-backed de novo motif calling on peak sequences)
    and tools.compare_motif for the female-vs-male enrichment comparison
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: HOMER
  role: External motif-discovery engine invoked by cc.tools.call_motif to find motifs
    enriched in the differential peak sequences
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae070
    title: 'Pycallingcards: an integrated environment for visualizing, analyzing,
      and interpreting Calling Cards data'
  source_content_hash: sha256:c83baee66f68a6dc02659850deefb42426ad67f717952bf6cadc049a791b92e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/The-Mitra-Lab/pycallingcards
  tools:
  - HOMER
  - pycallingcards
  - tools.call_motif
  - tools.compare_motif
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# sex-linked-motif-family-comparison

## Summary

A motif-comparison skill that calls enriched TF motifs separately on differentially bound Brd4 peaks from female and male glioblastoma calling cards samples (using pycallingcards' HOMER-backed motif calling) and contrasts them to identify sex-specific motif families such as IRF, ETS, TEA, and MADS.

## When to use

Use when you have two sex-stratified sets of differentially bound calling cards peaks (e.g. female vs male GBM CC Brd4 peaks) and need to identify TF motif families enriched in one sex but not the other.

## When NOT to use

- The input is bulk qbed/ccf insertion data that has not yet been peak-called and differentially analyzed — run cc preprocessing (clean_qbed, call_peaks, make_Anndata, filter_peaks, differential peak analysis) first.
- Only one sex/condition is available — a between-sex comparison via cc.tools.compare_motif cannot be performed.
- The goal is single-cell clustering or peak-to-gene expression pairing rather than motif family enrichment comparison between peak sets.

## Inputs

- Female differentially bound Brd4 peak set from GBM calling cards data (pandas DataFrame / BED-like peak table)
- Male differentially bound Brd4 peak set from GBM calling cards data (pandas DataFrame / BED-like peak table)
- Genome reference for peak sequences used by HOMER motif discovery

## Outputs

- Per-sex motif enrichment results from cc.tools.call_motif (HOMER de novo motif calls on female and male differential peaks)
- Sex-vs-sex comparison table of motif family enrichment (motif_enrichment_comparison.csv)
- Summary of sex-specific TF motif families (IRF, ETS, TEA, MADS female-specific; HMG/Sox male-skewed)

## How to apply

Load the female and male differentially bound peak sets (obtained from differential peak analysis of the GBM calling cards data) into pandas DataFrames. Call motifs enriched in each sex's differential peak set with cc.tools.call_motif, which invokes HOMER for de novo motif discovery on the peak sequences, then repeat for the other sex. Compare the two motif enrichment results with cc.tools.compare_motif to detect motif families with sex-specific enrichment, and write the comparison table to motif_enrichment_comparison.csv. The rationale is that the Pycallingcards tools (tl) module exists specifically to 'call motif of the peaks, completes differential peaks' so motif finding is integrated with differential peak analysis. Judge success by whether the family-level pattern (IRF, ETS, TEA, MADS in female but not male; extreme HMG/Sox difference in male) matches the reported Supplementary Tables S4–S5; otherwise flag discrepancies for expert review.

## Related tools

- **pycallingcards** (Provides tools.call_motif (HOMER-backed de novo motif calling on peak sequences) and tools.compare_motif for the female-vs-male enrichment comparison) — https://github.com/The-Mitra-Lab/pycallingcards
- **HOMER** (External motif-discovery engine invoked by cc.tools.call_motif to find motifs enriched in the differential peak sequences)

## Examples

```
cc.tools.call_motif(female_diff_peaks, outdir='female_motifs'); cc.tools.call_motif(male_diff_peaks, outdir='male_motifs'); cc.tools.compare_motif('female_motifs', 'male_motifs', out='motif_enrichment_comparison.csv')
```

## Evaluation signals

- The motif_enrichment_comparison.csv table shows IRF, ETS, TEA, and MADS motif families enriched in female but absent from male CC differential peaks.
- The HMG (Sox) motif family shows an extreme enrichment difference in male samples, matching the reported pattern.
- Family-level results are consistent with the article's Supplementary Tables S4–S5.
- Motif calls are successfully produced for both the female and the male differential peak sets (HOMER completes on both).
- Any deviation from the reported family pattern is explicitly flagged as a discrepancy for expert review rather than silently accepted.

## Limitations

- Motif calling quality depends on the upstream differential peak calling (CCcaller/MACCs) and peak filtering steps; errors there propagate to the motif comparison.
- The validated finding applies to the reanalyzed glioblastoma CC dataset (Brd4 peaks, female vs male CC samples); other datasets may not reproduce the IRF/ETS/TEA/MADS pattern.
- If the comparison does not show the expected family-level patterns, results must be flagged for expert review rather than treated as definitive.

## Evidence

- [methods] **Tools (tl)** calls motif of the peaks, completes differential peaks and pair differential peaks with gene expression,: "**Tools (tl)** calls motif of the peaks, completes differential peaks and pair differential peaks with gene expression,"
- [abstract] Pycallingcards offers a fully integrated environment for data visualization, motif finding, and comparative analysis with RNA-seq and ChIP-seq datasets.: "Pycallingcards offers a fully integrated environment for data visualization, motif finding, and comparative analysis with RNA-seq and ChIP-seq datasets."
- [abstract] This analysis revealed novel cell-type-specific binding sites and potential sex-linked TF regulators,: "This analysis revealed novel cell-type-specific binding sites and potential sex-linked TF regulators,"
- [abstract] we have reanalyzed previously published mouse cortex and glioblastoma datasets.: "we have reanalyzed previously published mouse cortex and glioblastoma datasets."
- [methods] **Preprocessing (pp)** helps to preprocess data from qbed/ccf data to call peaks, make annotation, make Anndata object and filter peaks.: "**Preprocessing (pp)** helps to preprocess data from qbed/ccf data to call peaks, make annotation, make Anndata object and filter peaks."
