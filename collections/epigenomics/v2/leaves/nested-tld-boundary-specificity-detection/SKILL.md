---
name: nested-tld-boundary-specificity-detection
description: Use when single-cell TLD predictions from multiple cell types (e.g.,
  Lee PFC scHi-C cells) are available and the goal is to identify which nested TLD
  boundaries distinguish specific cell types.
when_to_use_negative:
- Input is bulk Hi-C data with a single contact matrix per condition — boundary-frequency
  statistics across single cells cannot be computed
- Cells lack sufficient TLDs (e.g., '2n DNA' stage), where deDoc2 fails to identify
  a sufficient number of TLDs and data must be excluded
- The goal is TLD prediction itself rather than comparing boundary occurrence across
  cell types
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3678
tools:
- name: deDoc2
  role: Predicts the hierarchical (nested) TLDs per single cell whose boundaries are
    tested for cell-type specificity; deDoc2.w for higher-level and deDoc2.s for lower-level
    TLDs
  repo: https://github.com/zengguangjie/deDoc2
- name: scikit-learn
  role: Downstream clustering and adjusted mutual information scoring on TLD features
    used alongside boundary-specificity results
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/advs.202300366
    title: DeDoc2 Identifies and Characterizes the Hierarchy and Dynamics of Chromatin
      TAD‐Like Domains in the Single Cells
  source_content_hash: sha256:e674f8d2b120ed975f6f2f67307e41366a2afdfb900b56fa3ecea70f34e81fb2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zengguangjie/deDoc2
  tools:
  - MulticoreT-SNE
  - TruncatedSVD
  - deDoc2
  - harmonypy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3678
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# nested-tld-boundary-specificity-detection

## Summary

Detects cell-type-specific nested TAD-like domain (TLD) boundaries by computing per-cell-type occurrence frequencies of TLD boundaries predicted by deDoc2, flagging high-frequency boundaries (mean + 3 SD) and designating boundaries as cell-type-specific only when absent from neighboring positions in other cell types.

## When to use

Use when single-cell TLD predictions from multiple cell types (e.g., Lee PFC scHi-C cells) are available and the goal is to identify which nested TLD boundaries distinguish specific cell types.

## When NOT to use

- Input is bulk Hi-C data with a single contact matrix per condition — boundary-frequency statistics across single cells cannot be computed
- Cells lack sufficient TLDs (e.g., '2n DNA' stage), where deDoc2 fails to identify a sufficient number of TLDs and data must be excluded
- The goal is TLD prediction itself rather than comparing boundary occurrence across cell types

## Inputs

- Per-cell nested TLD boundary lists from deDoc2.w and deDoc2.s predictions (binsize 40 kb, 10 Mb sliding window)
- Cell-type labels for the 4238 Lee PFC single cells (e.g., L2/3, Vip, Astro, MG)
- Optionally, cell-type-specific highly expressed gene tables from the Allen brain scRNA-seq resource for cross-referencing

## Outputs

- Cell-type-specific nested TLD boundary lists per cell type
- CSV files of high-frequency boundary occurrence frequencies per cell type

## How to apply

First decode hierarchical TLDs per cell using deDoc2 (deDoc2.w for higher-level and deDoc2.s for lower-level TLDs, default 10 Mb sliding window) on RWR-imputed, sqrtVC-normalized single-cell contact matrices. For each cell type (e.g., L2/3, Vip, Astro, MG from Lee's 4238-cell human prefrontal cortex dataset), compute the occurrence frequency of each nested TLD boundary across cells. Call a boundary high-frequency when its frequency exceeds the mean plus three standard deviations for that cell type. Designate a boundary cell-type-specific only when no high-frequency boundary occurs in the three bins before or after that position in any of the other cell types, which guards against calling shared or shifted boundaries as specific. Save the resulting cell-type-specific boundary lists to CSV for downstream comparison with cell-type-specific highly expressed genes (e.g., from the Allen brain single-cell RNA-seq resource).

## Related tools

- **deDoc2** (Predicts the hierarchical (nested) TLDs per single cell whose boundaries are tested for cell-type specificity; deDoc2.w for higher-level and deDoc2.s for lower-level TLDs) — https://github.com/zengguangjie/deDoc2
- **scikit-learn** (Downstream clustering and adjusted mutual information scoring on TLD features used alongside boundary-specificity results)

## Examples

```
java -jar deDoc2.jar -inputfile ./cell1.chr18.matrix -binsize 40 -rwr -rp 0.5 -windowsize 10
```

## Evaluation signals

- Verify the output CSV files exist and are parseable
- Check each flagged boundary exceeds the mean + 3 SD occurrence-frequency threshold for its cell type
- Confirm no high-frequency boundary exists within three bins before or after a designated specific boundary in other cell types
- Qualitatively check that cell-type-specific boundaries align with cell-type-specific highly expressed genes from the Allen brain scRNA-seq resource
- Check cell-type assignment of boundaries is consistent with the 14 reported Lee PFC cell-type labels

## Limitations

- Boundary specificity is defined relative to the analyzed cell-type panel; boundaries may appear specific only because the comparison set lacks the relevant cell type
- Cells with insufficient TLDs (e.g., '2n DNA' stage) must be excluded, reducing coverage
- No software version numbers for deDoc2 are stated in the article, hindering exact reproduction

## Evidence

- [results] Cell type-specific highly expressed genes and nested TLD boundaries defined for L2/3, Vip, Astro, and MG cells: "We picked one representative cell type from each of the four cell classes, i.e., L2/3, Vip, Astro, and MG, and defined their cell type‐specific highly expressed genes and nested TLD boundaries"
- [other] High-frequency and cell-type-specific boundary criteria: [evidence span withheld — anchor status: not_found]
- [results] deDoc2 two-predictor structure for nested TLDs: "The deDoc2 package consists of two predictors, deDoc2.w and deDoc2.s, to predict higher and lower level TLDs"
- [results] Exclusion of cells with insufficient TLDs: "As we failed to identify a sufficient number of TLDs from “2n DNA” stage, we excluded those data from subsequent analysis"
- [methods] deDoc2.s sliding-window partition of the contact matrix: "A sliding window is used to split the Hi‐C contact matrix and deDoc2.w is applied to the split matrix"
