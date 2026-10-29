---
name: transcription-factor-motif-scanning
description: Use when you have FASTA sequences under predicted ChIP-enriched peaks
  (plus matched control sequences) and need to verify enrichment of an expected TF
  binding motif family, or compare motif enrichment across peak ranks or platforms.
when_to_use_negative:
- The binding motif of interest is novel or not represented in motif databases — the
  article notes many stored motif patterns may be inaccurate and novel motifs may
  not exist in databases
- You have peak intervals but no matched control sequences, so no test-vs-control
  chi-square enrichment can be computed
- You need de novo motif discovery rather than scanning against known curated matrices
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0797
tools:
- name: MatInspector
  role: Scans test and control peak sequences for motif family matches under default
    settings
- name: Genomatix suite
  role: Suite hosting MatInspector and the Genomatix MatBase 7.0 motif matrix database
- name: HPeak
  role: Upstream peak caller whose -seq output supplies the FASTA sequences of ChIP-enriched
    regions for motif scanning
  repo: github.com/cauyrd/HPeak
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1186/1471-2105-11-369
    title: 'HPeak: an HMM-based algorithm for defining read-enriched regions in ChIP-Seq
      data'
  source_content_hash: sha256:2542cea137c9fd3ac95b29e4c2aa2d8ef25b734bb5faa5187f8d8cd4f7c4d99e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Genomatix suite
  - HPeak
  - MatInspector
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# transcription-factor-motif-scanning

## Summary

Scan genomic sequences underlying ChIP-Seq peaks (e.g., HPeak-called STAT1 peaks) and matched control sequences for transcription factor binding motifs using MatInspector against Genomatix MatBase 7.0, then quantify per-family enrichment with chi-square statistics to validate peak-calling quality.

## When to use

Use when you have FASTA sequences under predicted ChIP-enriched peaks (plus matched control sequences) and need to verify enrichment of an expected TF binding motif family, or compare motif enrichment across peak ranks or platforms.

## When NOT to use

- The binding motif of interest is novel or not represented in motif databases — the article notes many stored motif patterns may be inaccurate and novel motifs may not exist in databases
- You have peak intervals but no matched control sequences, so no test-vs-control chi-square enrichment can be computed
- You need de novo motif discovery rather than scanning against known curated matrices

## Inputs

- FASTA sequence file of genomic regions under predicted peaks (e.g., HPeak -seq output)
- Matched control genomic sequences
- Genomatix MatBase 7.0 motif matrix library (153 vertebrate motif matrix families)

## Outputs

- Motif family occurrence counts in test versus control sequences
- Per-family chi-square enrichment statistics
- Rank-stratified motif enrichment profiles
- motif_enrichment_results.csv

## How to apply

Extract the genomic sequences under the predicted peaks — HPeak's '-seq' output produces exactly this sequence file for subsequent motif analysis — together with matched control sequences. Scan all test and control sequences with MatInspector in the Genomatix suite using default settings against all 153 families of vertebrate motif matrices in Genomatix MatBase 7.0. Tabulate motif family occurrence counts in test versus control sequences and compute chi-square statistics quantifying enrichment per motif family. For validation, check that the expected family (e.g., STAT family for STAT1 ChIP-Seq) ranks first and that its negative log transformed p-value is about twice as large as the second highest family. Optionally stratify peaks by rank and recompute enrichment within each rank bin, expecting steeper enrichment decay for higher-resolution ChIP-Seq peaks than ChIP-chip regions. Write family-level chi-square statistics and rank-dependent enrichment profiles to a results table (e.g., motif_enrichment_results.csv).

## Related tools

- **MatInspector** (Scans test and control peak sequences for motif family matches under default settings)
- **Genomatix suite** (Suite hosting MatInspector and the Genomatix MatBase 7.0 motif matrix database)
- **HPeak** (Upstream peak caller whose -seq output supplies the FASTA sequences of ChIP-enriched regions for motif scanning) — github.com/cauyrd/HPeak

## Evaluation signals

- STAT family (or the expected TF family) has the highest chi-square enrichment, with negative log transformed p-value about twice the second highest family
- Motif enrichment of peak sequences exceeds enrichment of matched control sequences
- Rank-stratified enrichment shows monotonic decay from top-ranked to lower-ranked peaks (steeper for ChIP-Seq than ChIP-chip)
- All 153 vertebrate motif matrix families from Genomatix MatBase 7.0 were scanned, not a single-matrix query
- Family-level statistics and rank profiles are present in motif_enrichment_results.csv

## Limitations

- Many motif patterns stored in the database may not be accurate, and there may be novel motifs that do not exist in motif databases
- The actual effect of inaccurate motif patterns on comparison results remains to be seen; further studies seem warranted
- Enrichment is family-level (e.g., all STAT matrices pooled), not a single-motif specificity test

## Evidence

- [discussion] Motif scan with MatInspector defaults: "Motif scan of the test and the control sequences was then performed using MatInspector [52] in the Genomatix suite"
- [discussion] 153 vertebrate motif matrix families: "all 153 families of vertebrate motif matrices found in the Genomatix MatBase 7.0 database"
- [abstract] Motif enrichment advantage of HPeak peaks: "HPeak showed a higher prevalence of the expected transcription factor binding motifs in ChIP-enriched sequences relative to the control sequences"
- [discussion] HPeak sequence output for motif analysis: "These include a wiggle (WIG) format file for visualization, a sequence file for subsequent motif analysis and an annotation file"
- [readme] README downstream analysis outputs: "Prepares necessary outputs ready for down-steam analysis such as enrichment curve and heatmap"
