---
name: cell-type-specific-gene-expression-identification
description: Use when single-cell expression data for known cell types (e.g., Astro,
  MG, L2/3, Vip) and per-cell nested TLD boundary calls are available and you need
  to designate genes or boundaries as cell-type-specific.
when_to_use_negative:
- Input is bulk Hi-C or bulk RNA-seq data without per-cell labels — the frequency-based
  boundary specificity test requires single-cell boundary calls per cell type
- Cell types have not been pre-identified (e.g., the 14 Lee PFC labels) and no representative
  cell types such as L2/3, Vip, Astro, MG can be picked
- The goal is TLD boundary detection itself rather than associating existing TLD boundaries
  with cell identity
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: deDoc2
  role: Decodes hierarchical TAD-like domains per cell to supply the nested TLD boundaries
    whose per-cell-type frequencies are tested for specificity
  repo: https://github.com/zengguangjie/deDoc2
- name: deDoc
  role: Predecessor structural-information-minimisation tool for bulk Hi-C TAD prediction;
    not used for this single-cell specificity step
  repo: https://github.com/yinxc/structural-information-minimisation
- name: scikit-learn
  role: Supporting machine-learning utilities (TF-IDF/TruncatedSVD/t-SNE pipeline
    context and adjusted mutual information scoring) in the same Lee PFC workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/advs.202300366
    title: DeDoc2 Identifies and Characterizes the Hierarchy and Dynamics of Chromatin
      TAD‐Like Domains in the Single Cells
  source_content_hash: sha256:8ff3a68970748cb9450348250cb4bb98a5f327f55df20716790add93d5dffda2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zengguangjie/deDoc2
  tools:
  - MulticoreT-SNE
  - TruncatedSVD
  - deDoc
  - deDoc2
  - harmonypy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# cell-type-specific-gene-expression-identification

## Summary

Identify cell-type-specific highly expressed genes and cell-type-specific nested TLD boundaries for representative cell types (L2/3, Vip, Astro, MG) from the human prefrontal cortex, linking TLD hierarchy to cellular identity.

## When to use

Use when single-cell expression data for known cell types (e.g., Astro, MG, L2/3, Vip) and per-cell nested TLD boundary calls are available and you need to designate genes or boundaries as cell-type-specific.

## When NOT to use

- Input is bulk Hi-C or bulk RNA-seq data without per-cell labels — the frequency-based boundary specificity test requires single-cell boundary calls per cell type
- Cell types have not been pre-identified (e.g., the 14 Lee PFC labels) and no representative cell types such as L2/3, Vip, Astro, MG can be picked
- The goal is TLD boundary detection itself rather than associating existing TLD boundaries with cell identity

## Inputs

- Single-cell RNA-seq expression data for Astro, MG, L2/3, and Vip cells from the Allen brain atlas (portal.brain-map.org human multiple cortical areas Smart-seq)
- Per-cell nested TLD boundary predictions from deDoc2.w and deDoc2.s on Lee PFC scHi-C contact matrices
- Cell-type labels for the 4238 Lee prefrontal cortex single cells (14 reported cell types)

## Outputs

- CSV table of cell-type-specific highly expressed genes per cell type
- CSV list of cell-type-specific nested TLD boundaries per cell type
- 2D t-SNE embedding coordinates with cell labels (optional companion output)

## How to apply

Obtain single-cell RNA-seq data for the four representative cell types (Astro, MG, L2/3, Vip) from the Allen brain single-cell RNA-seq resource (human multiple cortical areas Smart-seq), and call gene A cell-type-specific highly expressed when its expression exceeds that of the other three cell types. In parallel, decode hierarchical TLDs per cell with deDoc2 (deDoc2.w global partition and deDoc2.s sliding-window partition, default 10 Mb window) on RWR-imputed contact matrices (sqrtVC normalization, restart probability α=0.5, convergence at ||Qt − Qt−1||2 ≤ 10−2). Compute the occurrence frequency of each nested TLD boundary per cell type; call a boundary high-frequency when its frequency exceeds the mean plus three standard deviations, and designate it cell-type-specific when no high-frequency boundary occurs in the three bins before or after that position in the other cell types. Save the cell-type-specific gene tables and boundary lists to CSV files for downstream validation.

## Related tools

- **deDoc2** (Decodes hierarchical TAD-like domains per cell to supply the nested TLD boundaries whose per-cell-type frequencies are tested for specificity) — https://github.com/zengguangjie/deDoc2
- **deDoc** (Predecessor structural-information-minimisation tool for bulk Hi-C TAD prediction; not used for this single-cell specificity step) — https://github.com/yinxc/structural-information-minimisation
- **scikit-learn** (Supporting machine-learning utilities (TF-IDF/TruncatedSVD/t-SNE pipeline context and adjusted mutual information scoring) in the same Lee PFC workflow)

## Examples

```
java -jar deDoc2.jar -inputfile cell_chr18.matrix -binsize 40 -rwr -rp 0.5 -outputfile cell_chr18
```

## Evaluation signals

- Output CSV files for cell-type-specific gene tables and boundary lists exist and are parseable
- High-frequency boundary threshold is verifiably computed as mean + 3 standard deviations of per-cell-type occurrence frequency
- A boundary labeled cell-type-specific has no high-frequency boundary within three bins before or after its position in the other three cell types
- A gene labeled cell-type-specific has expression exceeding that of the other three of the four cell types (Astro, MG, L2/3, Vip)
- The specific boundaries assigned to each of the four cell classes qualitatively align with the cell-type clustering of the t-SNE embedding of TLD features

## Limitations

- Only one representative cell type per cell class (L2/3, Vip, Astro, MG) was analyzed, so specificity calls do not cover all 14 reported cell types
- Cell-type-specific boundary definition depends on the three-bin flank tolerance; boundaries near other cell types' high-frequency peaks within that window are discarded
- Cells with insufficient TLDs (e.g., '2n DNA' stage) must be excluded from subsequent analysis
- No software version numbers were stated for deDoc2 or compared tools

## Evidence

- [results] Cell type-specific highly expressed genes defined via cross-type comparison: "We picked one representative cell type from each of the four cell classes, i.e., L2/3, Vip, Astro, and MG, and defined their cell type‐specific highly expressed genes and nested TLD boundaries"
- [results] deDoc2 two-predictor structure for hierarchical TLDs: "The deDoc2 package consists of two predictors, deDoc2.w and deDoc2.s, to predict higher and lower level TLDs"
- [methods] RWR imputation convergence criterion: [evidence span withheld — anchor status: not_found]
- [methods] sqrtVC normalization rationale for RWR convergence: [evidence span withheld — anchor status: not_found]
- [readme] README description of deDoc2.w and deDoc2.s roles: "The deDoc2.w minimizes the structural entropy in the whole Hi-C contact map, while the deDoc2.s minimizes the structural entropy in the matrices of sliding windows along the genome."
- [readme] Sliding window default parameter: "|-windowsize|Optional|size of the sliding window in deDoc2.s (Mb)|10Mb|"
