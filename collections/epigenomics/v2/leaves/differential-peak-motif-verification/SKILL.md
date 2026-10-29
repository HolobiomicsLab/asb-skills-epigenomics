---
name: differential-peak-motif-verification
description: Use when two condition-specific differentially bound peak sets (e.g.
  female vs male GBM CC peaks) exist and the goal is to identify which TF motif families
  are enriched in one condition but not the other, reproducing published family-level
  enrichment patterns.
when_to_use_negative:
- Input is a single undivided peak set with no condition/sex grouping — call motifs
  directly on the peaks instead of running a between-group comparison
- The question is differential peak binding itself (pp/tl differential analysis),
  not which motifs explain already-computed differential peaks
- Peaks are not TF-binding calling cards (qbed/ccf-derived) peaks; HOMER motif discovery
  on unrelated feature sets is out of scope for this workflow
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
tools:
- name: pycallingcards
  role: Python package providing the tools module (cc.tools.call_motif, cc.tools.compare_motif)
    used to call and compare motifs on differential peak sets
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: tools.call_motif
  role: Invokes HOMER for de novo motif discovery on the sequences of each sex-specific
    differential peak set
- name: tools.compare_motif
  role: Compares motif enrichment between female and male groups to identify motif
    families with sex-specific enrichment
- name: HOMER
  role: External de novo motif discovery engine invoked by call_motif on peak sequences
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae070
    title: 'Pycallingcards: an integrated environment for visualizing, analyzing,
      and interpreting Calling Cards data'
  source_content_hash: sha256:89bbf4b298bf3d83e1fe923e8521fc968bb825fcd4a19db9c53c647d840f2b42
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
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# differential-peak-motif-verification

## Summary

Verifies sex-specific transcription factor motif enrichment by calling motifs on differentially bound Brd4 peaks from female vs male glioblastoma calling cards data with pycallingcards' HOMER-backed tools, then comparing enrichment between groups to confirm reported TF family patterns.

## When to use

Use when two condition-specific differentially bound peak sets (e.g. female vs male GBM CC peaks) exist and the goal is to identify which TF motif families are enriched in one condition but not the other, reproducing published family-level enrichment patterns.

## When NOT to use

- Input is a single undivided peak set with no condition/sex grouping — call motifs directly on the peaks instead of running a between-group comparison
- The question is differential peak binding itself (pp/tl differential analysis), not which motifs explain already-computed differential peaks
- Peaks are not TF-binding calling cards (qbed/ccf-derived) peaks; HOMER motif discovery on unrelated feature sets is out of scope for this workflow

## Inputs

- Female differentially bound Brd4 peak set from GBM calling cards differential peak analysis (pandas DataFrame)
- Male differentially bound Brd4 peak set from GBM calling cards differential peak analysis (pandas DataFrame)
- pycallingcards peak/Anndata objects produced by pp.call_peaks, pp.make_Anndata, and pp.filter_peaks

## Outputs

- Per-group motif enrichment results from cc.tools.call_motif (HOMER de novo motif discovery)
- Sex-specific motif comparison table (motif_enrichment_comparison.csv) with family-level enrichment differences
- Validation verdict: confirmed IRF/ETS/TEA/MADS female-specific enrichment and male HMG (Sox) difference, or flagged discrepancies

## How to apply

Load the female and male differentially bound Brd4 peak sets (from differential peak analysis of the GBM calling cards data) into pandas DataFrames. Call motifs enriched in each group separately using cc.tools.call_motif, which invokes HOMER for de novo motif discovery on peak sequences. Compare motif enrichment between the female and male groups with cc.tools.compare_motif to isolate motif families with sex-specific enrichment, and write the comparison to motif_enrichment_comparison.csv. Validate against the published pattern: IRF, ETS, TEA, and MADS motif families enriched in female but not male CC data, and the HMG (Sox) family showing an extreme difference in male samples (Supplementary Tables S4-S5); flag discrepancies for expert review rather than accepting silently.

## Related tools

- **pycallingcards** (Python package providing the tools module (cc.tools.call_motif, cc.tools.compare_motif) used to call and compare motifs on differential peak sets) — https://github.com/The-Mitra-Lab/pycallingcards
- **tools.call_motif** (Invokes HOMER for de novo motif discovery on the sequences of each sex-specific differential peak set)
- **tools.compare_motif** (Compares motif enrichment between female and male groups to identify motif families with sex-specific enrichment)
- **HOMER** (External de novo motif discovery engine invoked by call_motif on peak sequences)

## Examples

```
import pycallingcards as cc; female_motifs = cc.tools.call_motif(female_peaks); male_motifs = cc.tools.call_motif(male_peaks); cc.tools.compare_motif(female_motifs, male_motifs)  # write motif_enrichment_comparison.csv
```

## Evaluation signals

- Comparison table shows IRF, ETS, TEA, and MADS motif families present in female but absent from male CC motif results
- HMG (Sox) motif family shows an extreme enrichment difference in male samples, matching Supplementary Tables S4-S5
- motif_enrichment_comparison.csv is produced and contains family-level rows for both female and male groups
- Each call_motif run completes on the correct group's differential peak DataFrame (female peaks in, male peaks in, no group mixing)
- Any deviation from the published family-level pattern is explicitly flagged for expert review rather than silently accepted

## Limitations

- Validation depends on reproducing specific published patterns (Supplementary Tables S4-S5); pipeline, HOMER version, or peak-set differences can cause discrepancies that require expert review
- Motif calling quality depends on the upstream differential peak analysis; filtering (pp.filter_peaks) and peak calling choices propagate into motif results
- HOMER must be installed and accessible for tools.call_motif to invoke it

## Evidence

- [methods] **Tools (tl)** calls motif of the peaks, completes differential peaks and pair differential peaks with gene expression,: "**Tools (tl)** calls motif of the peaks, completes differential peaks and pair differential peaks with gene expression,"
- [abstract] Pycallingcards offers a fully integrated environment for data visualization, motif finding, and comparative analysis with RNA-seq and ChIP-seq datasets.: "Pycallingcards offers a fully integrated environment for data visualization, motif finding, and comparative analysis with RNA-seq and ChIP-seq datasets."
- [abstract] This analysis revealed novel cell-type-specific binding sites and potential sex-linked TF regulators, furthering our understanding of TF binding: "This analysis revealed novel cell-type-specific binding sites and potential sex-linked TF regulators, furthering our understanding of TF binding"
- [intro] **Preprocessing (pp)** helps to preprocess data from qbed data to call peaks, make annotation, make Anndata object and filter peaks.: "**Preprocessing (pp)** helps to preprocess data from qbed data to call peaks, make annotation, make Anndata object and filter peaks."
