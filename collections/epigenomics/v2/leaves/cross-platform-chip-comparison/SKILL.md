---
name: cross-platform-chip-comparison
description: Use when two genome-wide protein-DNA binding assays (e.g., ChIP-Seq and
  ChIP-chip) must be compared on biological signal quality rather than peak overlap,
  using motif enrichment of the expected TF-binding motif as the figure of merit.
when_to_use_negative:
- Input is a single-platform peak set with no comparable alternative-assay region
  set — use standard peak-calling QC instead
- No expected motif or motif-family database is available for the assayed transcription
  factor, since the comparison relies on enriching the known TF motif family
- The goal is peak-coordinate overlap (Jaccard/intersection) rather than biological
  signal-quality comparison — a direct interval-overlap tool is more appropriate
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0203
tools:
- name: HPeak
  role: Hidden Markov model-based peak caller producing ChIP-Seq peaks and a sequence
    (-seq) output file used as the ChIP-Seq test sequences in the cross-platform comparison
  repo: github.com/cauyrd/HPeak
- name: MatInspector
  role: Motif scanner used with default settings to detect motif family occurrences
    in test and control sequences for chi-square enrichment computation
- name: Genomatix suite
  role: Framework in which MatInspector is run against all 153 vertebrate motif matrix
    families in Genomatix MatBase 7.0
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/1471-2105-11-369
    title: 'HPeak: an HMM-based algorithm for defining read-enriched regions in ChIP-Seq
      data'
  source_content_hash: sha256:022fdc2cc07e3e5296fdcff50c5697c77a818558015ef1dc4c832a30d03fdfc5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Genomatix suite
  - HPeak
  - MatInspector
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cross-platform-chip-comparison

## Summary

Compares ChIP-Seq peaks (called by HPeak) against ChIP-chip enriched regions (e.g., 3,701 STAT1 regions at FDR 0.05 from Euskirchen et al.) by quantifying transcription-factor motif enrichment in test vs. matched control sequences, demonstrating ChIP-Seq's advantage in motif recovery and rank-dependence.

## When to use

Use when two genome-wide protein-DNA binding assays (e.g., ChIP-Seq and ChIP-chip) must be compared on biological signal quality rather than peak overlap, using motif enrichment of the expected TF-binding motif as the figure of merit.

## When NOT to use

- Input is a single-platform peak set with no comparable alternative-assay region set — use standard peak-calling QC instead
- No expected motif or motif-family database is available for the assayed transcription factor, since the comparison relies on enriching the known TF motif family
- The goal is peak-coordinate overlap (Jaccard/intersection) rather than biological signal-quality comparison — a direct interval-overlap tool is more appropriate

## Inputs

- ChIP-Seq peak coordinates and underlying genomic sequences from HPeak (via its -seq sequence output file)
- ChIP-chip enriched region coordinates (e.g., 3,701 STAT1 regions at FDR 0.05 on chr 20, 21, 22, X, Y from Euskirchen et al.) and underlying genomic sequences
- Matched control sequences for each platform's test regions
- Genomatix MatBase 7.0 vertebrate motif matrix library (153 families)

## Outputs

- Per-family chi-square enrichment statistics for test vs. control sequences (motif_enrichment_results.csv)
- Rank-stratified motif enrichment profiles per platform
- Comparison figures: family-level enrichment bar chart (Figure 1A/B style) and rank-dependence decay curves (Figure 1C style)

## How to apply

Extract genomic sequences underlying peaks from each platform (HPeak-called ChIP-Seq peaks and, for ChIP-chip, regions at FDR 0.05 such as the 3,701 STAT1 regions on chromosomes 20, 21, 22, X, Y) plus matched control sequences. Scan all test and control sequences with MatInspector in the Genomatix suite using default settings against all 153 families of vertebrate motif matrices in Genomatix MatBase 7.0. Tabulate motif family occurrence counts in test vs. control sequences and compute chi-square statistics quantifying enrichment per motif family. Judge the platform with the higher specific enrichment — in the STAT1 comparison, the STAT family's negative log-transformed p-value was about twice the second-highest family in ChIP-Seq peaks (Figure 1A/B). Additionally stratify peaks by rank, recompute motif enrichment per rank bin, and contrast the steeper enrichment decay in ChIP-Seq vs. ChIP-chip (Figure 1C). Write family-level chi-square statistics and rank-dependent enrichment profiles to motif_enrichment_results.csv.

## Related tools

- **HPeak** (Hidden Markov model-based peak caller producing ChIP-Seq peaks and a sequence (-seq) output file used as the ChIP-Seq test sequences in the cross-platform comparison) — github.com/cauyrd/HPeak
- **MatInspector** (Motif scanner used with default settings to detect motif family occurrences in test and control sequences for chi-square enrichment computation)
- **Genomatix suite** (Framework in which MatInspector is run against all 153 vertebrate motif matrix families in Genomatix MatBase 7.0)

## Examples

```
perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN -format BED -t stat1_seq.bed -c control.bed -n stat1 -fmin 100 -fmax 300 -r 36 -ann -wig -seq -interfiles
```

## Evaluation signals

- In the ChIP-Seq peak set, the STAT family's negative log-transformed p-value should be approximately twice that of the second-highest motif family, reproducing Figure 1A
- The same STAT-family dominance should be present but weaker in the ChIP-chip region set (Figure 1B)
- Rank-stratified enrichment should show steeper decay of STAT motif enrichment with decreasing peak rank in ChIP-Seq than in ChIP-chip, reproducing the Figure 1C trend
- motif_enrichment_results.csv must contain chi-square statistics for all 153 vertebrate motif matrix families with test and control occurrence counts

## Limitations

- Motif patterns stored in databases may be inaccurate and novel motifs may not exist in motif databases; the actual effect on comparison results remains to be seen
- Very little information on true positive and true negative loci is available, making evaluation criteria on experimental data challenging to determine — interpret comparisons with caution
- Comparison is limited to chromosomes present in the ChIP-chip array design (e.g., chr 20, 21, 22, X, Y for the Euskirchen et al. STAT1 data), so it is not genome-wide unless the lower-resolution platform covers the whole genome

## Evidence

- [results] Motifs in the STAT family are much more enriched relative to motifs from other families in the ChIP-Seq data: [evidence span withheld — anchor status: not_found]
- [discussion] MatInspector default scan against MatBase families: "Motif scan of the test and the control sequences was then performed using MatInspector [52] in the Genomatix suite (Genomatix GmBH, Munich, Germany) with default settings"
- [discussion] 153 motif families in MatBase 7.0: "all 153 families of vertebrate motif matrices found in the Genomatix MatBase 7.0 database"
- [discussion] HPeak sequence output for downstream motif analysis: "These include a wiggle (WIG) format file for visualization, a sequence file for subsequent motif analysis and an annotation file"
- [readme] README command with -seq output flag: "perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN/MOUSE -format BED -t TREATMENT.inp -c CONTROL.inp -n OUTPUTPREFIX -fmin 100 -fmax 300 -r 36 -ann -wig -seq -interfiles"
