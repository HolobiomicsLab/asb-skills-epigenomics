---
name: motif-family-enrichment-testing
description: Use when you need to validate called peaks or ChIP-enriched regions by
  checking enrichment of an expected TF motif family (e.g. STAT) against all vertebrate
  motif families, or compare motif enrichment across platforms (ChIP-Seq vs ChIP-chip)
  and peak-rank bins.
when_to_use_negative:
- Input is already a motif enrichment table — the chi-square testing step assumes
  raw motif occurrence counts from sequence scans
- Peak regions lack matched control sequences — family-level chi-square enrichment
  cannot be computed without a control background
- Expected binding motif may be absent from MatBase (the paper notes motif databases
  may be inaccurate and may lack novel motifs)
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3168
tools:
- name: MatInspector
  role: Scans test and control sequences for motif family matches under default settings
    to produce occurrence counts for chi-square testing
- name: Genomatix suite
  role: Provides the MatInspector scanning environment and Genomatix MatBase 7.0 database
    of 153 vertebrate motif matrix families
- name: HPeak
  role: Calls ChIP-Seq peaks and emits the sequence (FASTA) file used as test sequences
    for motif enrichment analysis
  repo: http://www.sph.umich.edu/csg/qin/HPeak
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/1471-2105-11-369
    title: 'HPeak: an HMM-based algorithm for defining read-enriched regions in ChIP-Seq
      data'
  source_content_hash: sha256:b6663ac5ac6f6650e79f643fdd56d1f819316db63483b0945363f88885b89685
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Genomatix suite
  - HPeak
  - MatInspector
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# motif-family-enrichment-testing

## Summary

Tests whether transcription-factor binding motif families are over-represented in ChIP-enriched sequences versus matched controls by scanning with MatInspector against Genomatix MatBase families and computing chi-square statistics, to validate peak-calling quality (e.g. STAT1 motif enrichment in ChIP-Seq vs ChIP-chip regions).

## When to use

Use when you need to validate called peaks or ChIP-enriched regions by checking enrichment of an expected TF motif family (e.g. STAT) against all vertebrate motif families, or compare motif enrichment across platforms (ChIP-Seq vs ChIP-chip) and peak-rank bins.

## When NOT to use

- Input is already a motif enrichment table — the chi-square testing step assumes raw motif occurrence counts from sequence scans
- Peak regions lack matched control sequences — family-level chi-square enrichment cannot be computed without a control background
- Expected binding motif may be absent from MatBase (the paper notes motif databases may be inaccurate and may lack novel motifs)

## Inputs

- FASTA sequences underlying test peaks (e.g. HPeak '-seq' output for STAT1 ChIP-Seq peaks)
- FASTA sequences for comparison regions (e.g. 3,701 ChIP-chip regions at FDR 0.05, chr 20/21/22/X/Y)
- Matched control sequences
- Genomatix MatBase 7.0 vertebrate motif matrices (153 families)

## Outputs

- Per-family motif occurrence counts in test and control sequences
- Chi-square enrichment statistics per motif family (negative log-transformed p-values)
- Rank-stratified motif enrichment profile per platform (e.g. Figure 1C trend)
- motif_enrichment_results.csv (family-level chi-square statistics and rank-dependent enrichment profiles)

## How to apply

Extract genomic sequences underlying the called peaks (e.g. STAT1 ChIP-Seq peaks from HPeak, output in HPeak's '-seq' FASTA format) and the comparison regions (e.g. 3,701 STAT1 ChIP-chip regions from Euskirchen et al. on chromosomes 20, 21, 22, X, Y at FDR 0.05), together with matched control sequences. Scan all test and control sequences with MatInspector in the Genomatix suite using default settings against all 153 families of vertebrate motif matrices in Genomatix MatBase 7.0. Tabulate motif family occurrence counts in test versus control sequences and compute chi-square statistics per motif family. Judge success by the expected motif family's enrichment rank/magnitude: for STAT1, the STAT family's negative log-transformed p-value should be about twice the second-highest family in ChIP-Seq peaks, and stratifying by peak rank should show steeper enrichment decay for ChIP-Seq than ChIP-chip.

## Related tools

- **MatInspector** (Scans test and control sequences for motif family matches under default settings to produce occurrence counts for chi-square testing)
- **Genomatix suite** (Provides the MatInspector scanning environment and Genomatix MatBase 7.0 database of 153 vertebrate motif matrix families)
- **HPeak** (Calls ChIP-Seq peaks and emits the sequence (FASTA) file used as test sequences for motif enrichment analysis) — http://www.sph.umich.edu/csg/qin/HPeak

## Examples

```
perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN -format BED -t STAT1_treat.bed -c STAT1_ctrl.bed -n STAT1 -r 36 -seq
```

## Evaluation signals

- The expected motif family (e.g. STAT) has the largest chi-square enrichment of all 153 families, with negative log-transformed p-value about twice the second-highest family for ChIP-Seq peaks
- Motif family counts are tabulated for both test and matched control sequences and every family receives a chi-square statistic
- Rank-stratified analysis reproduces the expected trend of steeper motif-enrichment decay with peak rank in ChIP-Seq than in ChIP-chip
- Output CSV contains family-level chi-square statistics plus rank-dependent enrichment profiles for both platforms
- HPeak peak sequences show higher prevalence of the expected TF motif relative to controls than peaks from other peak-calling approaches

## Limitations

- Many motif patterns stored in databases may be inaccurate and novel motifs may not exist in motif databases
- The actual effect of inaccurate motif patterns on comparison results remains to be seen and further studies seem warranted
- Little information on true positive and true negative loci makes it challenging to define evaluation criteria for experimental ChIP-Seq data, so performance results should be interpreted with caution

## Evidence

- [discussion] Motif scan of the test and the control sequences was then performed using MatInspector [52] in the Genomatix suite: "Motif scan of the test and the control sequences was then performed using MatInspector [52] in the Genomatix suite"
- [discussion] all 153 families of vertebrate motif matrices found in the Genomatix MatBase 7.0 database: "all 153 families of vertebrate motif matrices found in the Genomatix MatBase 7.0 database"
- [discussion] the ChIP-Seq technique has a clear advantage over the ChIP-chip technique in terms of enriching for an expected motif under the predicted peaks: "the ChIP-Seq technique has a clear advantage over the ChIP-chip technique in terms of enriching for an expected motif under the predicted peaks"
- [discussion] a sequence file for subsequent motif analysis: "These include a wiggle (WIG) format file for visualization, a sequence file for subsequent motif analysis and an annotation file"
- [readme] Accurate peak calling with detailed information including summit location and p-value: "Accurate peak calling with detailed information including summit location and p-value"
