---
name: sox-hmg-motif-family-annotation
description: Use when motif-calling output (e.g. HOMER results via cc.tools.call_motif)
  on CC differential peaks needs family-level annotation and the HMG/Sox family is
  a candidate sex- or condition-specific regulator.
when_to_use_negative:
- Input is raw qbed/ccf insertions that have not yet been peak-called and differentially
  tested — peak calling (pp.call_peaks) and differential peak analysis must precede
  motif annotation
- The goal is de novo motif discovery itself rather than family-level annotation/interpretation
  of existing motif-calling results
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
tools:
- name: pycallingcards
  role: Provides the analysis environment; tools.call_motif invokes HOMER on CC peak
    sequences and tools.compare_motif compares enrichment between female and male
    peak sets
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: HOMER
  role: De novo motif discovery engine invoked by cc.tools.call_motif on the peak
    sequences
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae070
    title: 'Pycallingcards: an integrated environment for visualizing, analyzing,
      and interpreting Calling Cards data'
  source_content_hash: sha256:de2dc67705a9dac4f706775acc9013178e393eaf91d5921f4931ee2bf3989e50
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
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# sox-hmg-motif-family-annotation

## Summary

Annotates and interprets HMG (Sox) motif family enrichment from calling cards (CC) peak motif-calling results, as demonstrated in the pycallingcards GBM analysis where the HMG (Sox) family showed an extreme enrichment difference in male glioblastoma samples relative to female samples.

## When to use

Use when motif-calling output (e.g. HOMER results via cc.tools.call_motif) on CC differential peaks needs family-level annotation and the HMG/Sox family is a candidate sex- or condition-specific regulator.

## When NOT to use

- Input is raw qbed/ccf insertions that have not yet been peak-called and differentially tested — peak calling (pp.call_peaks) and differential peak analysis must precede motif annotation
- The goal is de novo motif discovery itself rather than family-level annotation/interpretation of existing motif-calling results

## Inputs

- Female differentially bound Brd4 peak set from GBM calling cards data (pandas DataFrame)
- Male differentially bound Brd4 peak set from GBM calling cards data (pandas DataFrame)
- HOMER de novo motif discovery results from cc.tools.call_motif on each peak set

## Outputs

- Family-level motif enrichment annotation per sex (e.g. HMG/Sox, IRF, ETS, TEA, MADS)
- Comparison table written to motif_enrichment_comparison.csv

## How to apply

After differential peak analysis of the glioblastoma calling cards data, run motif discovery on the sex-specific peak sets (female and male) using cc.tools.call_motif, which invokes HOMER for de novo motif discovery on peak sequences, then compare enrichment between groups with cc.tools.compare_motif. Map the returned motifs to TF families (e.g. HMG for Sox motifs, plus IRF, ETS, TEA, MADS) and tabulate which families are enriched in one sex but not the other, writing the result to motif_enrichment_comparison.csv. Annotate the HMG (Sox) family specifically, validating that the male samples show the extreme HMG difference reported in Supplementary Tables S4-S5; flag any family-level pattern discrepancies for expert review. Rationale: the pycallingcards reanalysis of the GBM CC dataset revealed potential sex-linked TF regulators, and family-level annotation of motif results is how such sex-linked signals (HMG/Sox in male samples) are surfaced.

## Related tools

- **pycallingcards** (Provides the analysis environment; tools.call_motif invokes HOMER on CC peak sequences and tools.compare_motif compares enrichment between female and male peak sets) — https://github.com/The-Mitra-Lab/pycallingcards
- **HOMER** (De novo motif discovery engine invoked by cc.tools.call_motif on the peak sequences)

## Examples

```
motifs_f = cc.tl.call_motif(peaks_female, reference='hg38'); motifs_m = cc.tl.call_motif(peaks_male, reference='hg38'); cc.tl.compare_motif([motifs_f, motifs_m]).to_csv('motif_enrichment_comparison.csv')
```

## Evaluation signals

- The comparison table shows HMG (Sox) family with an extreme enrichment difference in male samples, consistent with Supplementary Tables S4-S5
- IRF, ETS, TEA, and MADS motif families are enriched in female but not male CC differential peaks
- motif_enrichment_comparison.csv is written and contains family-level enrichment rows for both sexes
- Any deviation from the reported family-level pattern is explicitly flagged for expert review rather than silently accepted

## Limitations

- The sex-specific enrichment pattern is specific to the reanalyzed glioblastoma Brd4 CC dataset; family assignments depend on HOMER de novo motif calling and may vary with peak set size or background choice
- Motif enrichment is on differentially bound peaks only, so non-differential binding signal is not represented in the family annotation

## Evidence

- [abstract] This analysis revealed novel cell-type-specific binding sites and potential sex-linked TF regulators, furthering our understanding of TF binding and gene expression relationships.: "This analysis revealed novel cell-type-specific binding sites and potential sex-linked TF regulators"
- [methods] **Tools (tl)** calls motif of the peaks, completes differential peaks and pair differential peaks with gene expression,: "**Tools (tl)** calls motif of the peaks, completes differential peaks and pair differential peaks with gene expression,"
- [other] Motif analysis of the sex-specific GBM CC peaks identifies four TF families — IRF, ETS, TEA, and MADS — that are found in female but not in male CC data: [evidence span withheld — anchor status: not_found]
- [readme] Pycallingcards offers a fully integrated environment for data visualization, motif finding, and comparative analysis with RNA-seq and ChIP-seq datasets.: "Pycallingcards offers a fully integrated environment for data visualization, motif finding, and comparative analysis"
