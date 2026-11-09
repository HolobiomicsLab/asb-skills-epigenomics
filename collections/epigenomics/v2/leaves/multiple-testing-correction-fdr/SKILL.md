---
name: multiple-testing-correction-fdr
description: Use when testing many motifs or functional terms simultaneously — e.g.
  PASTAA on REM sequences ranked by Cell type score, or g:Profiler on ~1,721 ARID3A-associated
  genes — to control the false discovery rate at BH FDR ≤ 0.05.
when_to_use_negative:
- When the analysis is not a multiple-testing setting — e.g. inspecting a single REM's
  activity or Cell type score in EpiRegio rather than testing many motifs or terms
  at once
- When a raw p-value threshold is explicitly required by the downstream protocol instead
  of a BH-adjusted one
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PASTAA
  role: TF motif enrichment tool on REM sequences ranked by Cell type score whose
    Benjamini–Hochberg adjusted p-values are thresholded at 0.05
  repo: github.com/TeamRegio/ApplicationScenarioExamples
- name: g:Profiler
  role: GO term and KEGG pathway enrichment of ARID3A target genes run with default
    parameters and BH FDR 0.05 significance threshold
- name: EpiRegio
  role: Provides the REM table ranked by Cell type score (Gene Query) and the ARID3A-associated
    genes (Region Query, 50% overlap) that constitute the multiple-hypothesis input
    sets
  repo: github.com/TeamRegio/EpiRegioDB
provenance:
  source_task_ids:
  - task_007
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkaa382
    title: 'EpiRegio: analysis and retrieval of regulatory elements linked to genes'
  source_content_hash: sha256:e5c9e87284b41a103e49b7cfe235c52db81dddb525f40bbe4c43838f53596359
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EpiRegio
  - PASTAA
  - TRAP
  - bedtools
  - g:Profiler
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# multiple-testing-correction-fdr

## Summary

Apply Benjamini–Hochberg false discovery rate correction to declare significance in enrichment analyses performed with EpiRegio-based workflows, using an adjusted p-value cutoff of 0.05 for both PASTAA TF motif enrichment over ranked REM sequences and g:Profiler GO/KEGG enrichment of TF target genes.

## When to use

Use when testing many motifs or functional terms simultaneously — e.g. PASTAA on REM sequences ranked by Cell type score, or g:Profiler on ~1,721 ARID3A-associated genes — to control the false discovery rate at BH FDR ≤ 0.05.

## When NOT to use

- When the analysis is not a multiple-testing setting — e.g. inspecting a single REM's activity or Cell type score in EpiRegio rather than testing many motifs or terms at once
- When a raw p-value threshold is explicitly required by the downstream protocol instead of a BH-adjusted one

## Inputs

- PASTAA output table with per-motif p-values (PASTAA_result.txt) from REM sequences ranked by Cell type score
- g:Profiler gost results for GO terms and KEGG pathways on the deduplicated ARID3A-associated gene ID list (1,721 genes)
- BH adjusted p-value cutoff parameter (<pvalue> = 0.05) for the identifyEnrichedTFs workflow

## Outputs

- Filtered list of significantly enriched TF motifs at Benjamini–Hochberg adjusted p-value ≤ 0.05 (e.g. 230 TFs including SP1, NFKB1, SMAD3)
- Filtered set of significant KEGG pathways (systemic lupus erythematosus, alcoholism) and GO biological processes (chromatin assembly or disassembly, nucleosome organization) at BH FDR ≤ 0.05
- Exported enrichment table gprofiler_results.csv

## How to apply

For the PASTAA motif-enrichment scenario (EndoMT REMs of differentially expressed heart genes ranked by Cell type score), pass a Benjamini–Hochberg adjusted p-value cutoff of 0.05 as the <pvalue> parameter to the identifyEnrichedTFs workflow.sh script; motifs whose BH-adjusted p-value is smaller than or equal to this threshold are assumed significantly enriched, which recovered 230 TFs including SP1, NFKB1, and SMAD3. For the g:Profiler scenario, run gost with default parameters on the deduplicated gene list from the EpiRegio Region Query of ARID3A ChIP-seq peaks ENCFF002CVL (50% overlap) and declare significant only terms at Benjamini–Hochberg FDR 0.05, which yielded the KEGG systemic lupus erythematosus and alcoholism pathways. The rationale is that scanning all JASPAR 2020 human motifs or all GO/KEGG terms involves thousands of concurrent hypothesis tests, so raw p-values must be adjusted before calling enrichment. Validate by confirming the output (e.g. <outputDir>/PASTAA_result.txt) contains only entries with adjusted p-value ≤ 0.05.

## Related tools

- **PASTAA** (TF motif enrichment tool on REM sequences ranked by Cell type score whose Benjamini–Hochberg adjusted p-values are thresholded at 0.05) — github.com/TeamRegio/ApplicationScenarioExamples
- **g:Profiler** (GO term and KEGG pathway enrichment of ARID3A target genes run with default parameters and BH FDR 0.05 significance threshold)
- **EpiRegio** (Provides the REM table ranked by Cell type score (Gene Query) and the ARID3A-associated genes (Region Query, 50% overlap) that constitute the multiple-hypothesis input sets) — github.com/TeamRegio/EpiRegioDB

## Examples

```
./identifyEnrichedTFs/workflow.sh JASPAR2020_HUMAN_transfac.txt hg38.fa REMs.csv 0.05
```

## Evaluation signals

- <outputDir>/PASTAA_result.txt exists and contains only TF motifs whose Benjamini–Hochberg adjusted p-value is ≤ 0.05
- The EndoMT run recovers 230 enriched TFs including SP1, NFKB1, and SMAD3 among the significant motifs
- g:Profiler output contains exactly the KEGG systemic lupus erythematosus and alcoholism pathways plus GO terms chromatin assembly or disassembly and nucleosome organization at BH FDR ≤ 0.05
- Gene IDs are deduplicated before g:Profiler submission, since duplicate gene IDs are considered only once
- gprofiler_results.csv is exported with adjusted p-value columns consistent with the 0.05 cutoff

## Limitations

- No tool versions are stated for PASTAA, EpiRegio, or related software, complicating exact reproduction of the adjusted p-values
- The linear model used in STITCHIT, which forms the basis for the Cell type score ranking that PASTAA consumes, is a simplification of actual gene regulation by REMs
- Only the first 90 unique genes fit in a g:Profiler web URL, so the full list must be pasted into the gost input field

## Evidence

- [methods] If the Benjamini-Hochberg adjusted p-value from PASTAA is smaller than or equal to the parameter <pvalue> the motif is assumed to be significant enriched.: "If the Benjamini-Hochberg adjusted p-value from PASTAA is smaller than or equal to the parameter *<pvalue>* the motif is assumed to be significant enriched."
- [discussion] we used PASTAA, which identifies TF binding motifs that are significantly enriched in high ranked input sequences compared to low ranked sequences: "we used PASTAA, which identifies TF binding motifs that are significantly enriched in high ranked input sequences compared to low ranked sequences"
- [methods] perform a GO term enrichment analysis using g:Profiler (default parameters) of the resulting REMs: "perform a GO term enrichment analysis using g:Profiler (default parameters) of the resulting REMs"
- [readme] the examples and the workflow script can be found in the folder identifyEnrichedTFs. The code itself is stored in src/: [evidence span withheld — anchor status: external_doc]
