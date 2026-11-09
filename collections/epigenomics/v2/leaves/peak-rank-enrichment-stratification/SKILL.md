---
name: peak-rank-enrichment-stratification
description: Use when comparing peak-calling outputs from ChIP-Seq and ChIP-chip (or
  across peak callers) and you need rank-resolved evidence that higher-ranked peaks
  are progressively enriched for the expected TF motif family.
when_to_use_negative:
- Input is already a per-peak motif-enrichment table — recomputing family-level chi-square
  counts would be redundant
- The transcription factor has no known motif in MatBase or comparable motif databases,
  making family-level enrichment meaningless
- Peak sets lack confidence ranks or scores, so rank-bin stratification cannot be
  performed
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: HPeak
  role: HMM-based peak caller producing ranked ChIP-Seq peaks plus FASTA sequence
    files for downstream motif analysis
  repo: github.com/cauyrd/HPeak
- name: MatInspector
  role: Scans test and control peak sequences for motif occurrences from all 153 vertebrate
    matrix families under default settings
- name: Genomatix suite
  role: Platform hosting MatInspector and the Genomatix MatBase 7.0 motif database
    used for family-level enrichment counts
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/1471-2105-11-369
    title: 'HPeak: an HMM-based algorithm for defining read-enriched regions in ChIP-Seq
      data'
  source_content_hash: sha256:976d84aa040bef9a8af120e69beebee549c0c8650e054b3cd29bfc35c3e566ff
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Genomatix suite
  - HPeak
  - MatInspector
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# peak-rank-enrichment-stratification

## Summary

A comparison strategy that stratifies called peaks (or ChIP-enriched regions) by rank and recomputes transcription factor motif-family enrichment within each rank bin, to quantify how motif enrichment decays with peak confidence and to contrast platforms such as ChIP-Seq (HPeak) versus ChIP-chip.

## When to use

Use when comparing peak-calling outputs from ChIP-Seq and ChIP-chip (or across peak callers) and you need rank-resolved evidence that higher-ranked peaks are progressively enriched for the expected TF motif family.

## When NOT to use

- Input is already a per-peak motif-enrichment table — recomputing family-level chi-square counts would be redundant
- The transcription factor has no known motif in MatBase or comparable motif databases, making family-level enrichment meaningless
- Peak sets lack confidence ranks or scores, so rank-bin stratification cannot be performed

## Inputs

- HPeak-called ChIP-Seq peak list with ranks (WIG/FASTA/annotation outputs from HPeak)
- ChIP-chip enriched region list (e.g. Euskirchen et al. 3,701 STAT1 regions, FDR 0.05, chromosomes 20, 21, 22, X, Y)
- Genomic FASTA sequences underlying test and matched control regions
- Vertebrate motif matrix families from Genomatix MatBase 7.0

## Outputs

- Family-level chi-square enrichment statistics for test versus control sequences per platform
- Rank-dependent motif-enrichment profile per rank bin per platform
- motif_enrichment_results.csv

## How to apply

Extract genomic FASTA sequences underlying ChIP-Seq peaks called by HPeak and ChIP-enriched regions (e.g. the 3,701 STAT1 ChIP-chip regions from Euskirchen et al. at FDR 0.05 on chromosomes 20, 21, 22, X, Y), plus matched control sequences. Scan test and control sequences with MatInspector in the Genomatix suite using default settings against all 153 families of vertebrate motif matrices in Genomatix MatBase 7.0, tabulate motif-family occurrence counts in test versus control, and compute chi-square statistics per family. Then stratify peaks by rank, recompute family-level chi-square enrichment within each rank bin, and compare the enrichment-decay profiles between platforms. Correct application reproduces the paper's trends: STAT-family enrichment in ChIP-Seq peaks about twice the runner-up family (negative log-transformed p-value roughly 2x), and steeper enrichment decay with rank for ChIP-Seq than for ChIP-chip.

## Related tools

- **HPeak** (HMM-based peak caller producing ranked ChIP-Seq peaks plus FASTA sequence files for downstream motif analysis) — github.com/cauyrd/HPeak
- **MatInspector** (Scans test and control peak sequences for motif occurrences from all 153 vertebrate matrix families under default settings)
- **Genomatix suite** (Platform hosting MatInspector and the Genomatix MatBase 7.0 motif database used for family-level enrichment counts)

## Examples

```
perl HPeak.pl -sp HUMAN -format BED -t treatment.bed -c control.bed -n stat1_out -fmin 100 -fmax 300 -r 36 -seq
```

## Evaluation signals

- STAT-family chi-square enrichment (negative log-transformed p-value) in ChIP-Seq peaks is about twice the second-highest motif family
- Rank-binned enrichment shows a steeper decay in ChIP-Seq peaks than in ChIP-chip regions
- Every motif family has occurrence counts tabulated for both test and control sequences on both platforms
- Output CSV contains one row per motif family with chi-square statistics and one profile per rank bin
- ChIP-chip comparison restricted to the specified chromosomes (20, 21, 22, X, Y) at FDR 0.05

## Limitations

- Little information on true positive and true negative loci makes criteria for evaluating peak-calling algorithms on experimental ChIP-Seq data challenging to determine; results should be interpreted with caution
- Motif patterns stored in databases may be inaccurate and novel motifs may not exist in motif databases; the actual effect on comparison results remains to be seen
- HPeak's method is not well-suited for quantifying false detection rate, which may disadvantage some compared methods in enrichment-based comparisons

## Evidence

- [discussion] Motif scan with MatInspector in Genomatix suite: "Motif scan of the test and the control sequences was then performed using MatInspector [52] in the Genomatix suite (Genomatix GmBH, Munich, Germany) with default settings"
- [discussion] ChIP-Seq advantage in motif enrichment under predicted peaks: "the ChIP-Seq technique has a clear advantage over the ChIP-chip technique in terms of enriching for an expected motif under the predicted peaks"
- [readme] HMM-based peak definition basis of ranked peaks: "HPeak is a hidden Markov model-based approach that can accurately pinpoint regions to where significantly more sequence reads map."
- [discussion] Motif database accuracy caveat: "many of the motif patterns stored in the database may not be accurate and there maybe novel motifs that do not exist in motif databases"
