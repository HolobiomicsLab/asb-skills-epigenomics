---
name: fisher-exact-test-differential-binding
description: Use when you have cell-by-peak AnnData matrices from CC qbed insertions
  in two sample groups and need condition-biased binding sites plus peak–gene concordance
  P-values.
when_to_use_negative:
- Input is already a gene expression feature table with no calling cards insertions
  — no peaks exist to test; use standard differential expression instead.
- Analysis goal is motif discovery or raw qbed file handling, not group-vs-group peak
  comparison.
- Data are bulk ChIP-seq peak counts requiring count-model differential tools rather
  than Fisher's exact tests on insertion-supported cell counts.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: Pycallingcards
  role: Provides tools.DE_pvalue (Fisher's exact differential peak analysis) and tools.pair_peak_gene_sc
    for peak–gene concordance testing
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: Scanpy
  role: Pycallingcards follows Scanpy structure and interacts with it to handle the
    single-cell AnnData objects used in the differential test
  repo: https://scanpy.readthedocs.io/en/stable/
- name: Mudata
  role: Combines peak and gene expression AnnData objects so paired differential analysis
    can access both modalities
- name: CCcaller
  role: Peak caller whose joint-call output (via preprocessing.call_peaks) produces
    the peaks fed into the differential test
  repo: https://github.com/The-Mitra-Lab/pycallingcards
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae070
    title: 'Pycallingcards: an integrated environment for visualizing, analyzing,
      and interpreting Calling Cards data'
  source_content_hash: sha256:a0a6bd2ea0b485edb2b5bca45d3ee912adc18351dda7d644f5af36f6fb82b676
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/The-Mitra-Lab/pycallingcards
  tools:
  - CCcaller
  - Mudata
  - Pycallingcards
  - Python
  - Scanpy
  - bedtools
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# fisher-exact-test-differential-binding

## Summary

Run differential peak analysis on single-cell calling cards data using Fisher's exact tests (Pycallingcards tools.DE_pvalue) to identify TF binding sites enriched in one condition (e.g. male vs female GBM), then pair differential peaks with gene expression to verify binding–expression concordance.

## When to use

Use when you have cell-by-peak AnnData matrices from CC qbed insertions in two sample groups and need condition-biased binding sites plus peak–gene concordance P-values.

## When NOT to use

- Input is already a gene expression feature table with no calling cards insertions — no peaks exist to test; use standard differential expression instead.
- Analysis goal is motif discovery or raw qbed file handling, not group-vs-group peak comparison.
- Data are bulk ChIP-seq peak counts requiring count-model differential tools rather than Fisher's exact tests on insertion-supported cell counts.

## Inputs

- cell-by-peak AnnData matrices from single-cell calling cards qbed insertions (e.g. male and female GBM Brd4 qbed from GSE148448)
- cell-by-gene expression AnnData (scRNA-seq) aligned to the same cell labels
- optional combined Mudata object of peak and gene expression AnnData

## Outputs

- differential binding table (per-peak Fisher's exact test statistics and P-values between the two groups)
- peak–gene pairing table with concordance P-values (target < .001)
- volcano and dotplot figures of differential peaks and binding–expression relationships

## How to apply

First build the cell-by-peak AnnData from preprocessed qbed insertions via pycallingcards preprocessing (combine_peaks, call_peaks with CCcaller, annotation, make_Anndata), then run differential peak analysis between the two groups with tools.DE_pvalue, which applies Fisher's exact tests to insertion counts per group to flag differentially bound sites. Pair each differential peak with nearby gene expression using tools.pair_peak_gene_sc to test whether binding bias is concordant with expression bias. For the Moudgil et al. GBM Brd4 reanalysis (GSE148448), require peak–gene concordance P-values < .001 and confirm the expected sex-biased pattern (Sema3a, Adam19, Zic1 female-biased; Nkx2-1 male-biased) as in Fig. 4a–g, visualizing with plotting.volcano_plot and plotting.dotplot_sc.

## Related tools

- **Pycallingcards** (Provides tools.DE_pvalue (Fisher's exact differential peak analysis) and tools.pair_peak_gene_sc for peak–gene concordance testing) — https://github.com/The-Mitra-Lab/pycallingcards
- **Scanpy** (Pycallingcards follows Scanpy structure and interacts with it to handle the single-cell AnnData objects used in the differential test) — https://scanpy.readthedocs.io/en/stable/
- **Mudata** (Combines peak and gene expression AnnData objects so paired differential analysis can access both modalities)
- **CCcaller** (Peak caller whose joint-call output (via preprocessing.call_peaks) produces the peaks fed into the differential test) — https://github.com/The-Mitra-Lab/pycallingcards

## Examples

```
import pycallingcards.tools as pytl; de_peaks = pytl.DE_pvalue(peak_cell_counts, exp_cell_counts, 'male', 'female', pvalue_cutoff=0.05); pair = pytl.pair_peak_gene_sc(de_peaks, peak_exp)
```

## Evaluation signals

- A Fisher's exact-test differential binding table is produced with per-peak statistics comparing male vs female samples
- Peak–gene concordance P-values for sex-biased pairs are < .001
- Sema3a, Adam19, and Zic1 show more Brd4 binding and higher expression in female cells; Nkx2-1 shows more binding and higher expression in male cells, as in Fig. 4a–g
- Volcano plot (plotting.volcano_plot) shows a populated set of significantly differential peaks, and dotplot_sc renders the binding–expression relationships

## Limitations

- The package implements two peak callers (CCcaller and MACCs); differential results depend on upstream peak-calling parameters such as CCcaller maxbetween
- Fisher's exact tests operate on calling cards insertion cell counts, so findings are hypothesis-generating — the paper describes pair analysis as finding 'potential hypothesis' for binding–expression relationships
- Annotation with preprocessing.annotation requires bedtools/pybedtools to be installed

## Evidence

- [methods] Differential peak analysis is then followed and pair analysis is completed to find out potential hypothesis: "Differential peak analysis is then followed and pair analysis is completed to find out potential hypothesis"
- [readme] **Tools (tl)** calls motif of the peaks, completes differential peaks and pair differential peaks with gene expression,: "**Tools (tl)** calls motif of the peaks, completes differential peaks and pair differential peaks with gene expression,"
- [abstract] This analysis revealed novel cell-type-specific binding sites and potential sex-linked TF regulators, furthering our understanding of TF binding and gene expression relationships.: "This analysis revealed novel cell-type-specific binding sites and potential sex-linked TF regulators, furthering our"
