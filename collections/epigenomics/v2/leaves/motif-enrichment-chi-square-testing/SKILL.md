---
name: motif-enrichment-chi-square-testing
description: Use when comparing motif content of peak sets from different ChIP-Seq
  peak callers — e.g., testing whether the NRSF motif separates HPeak-unique peaks
  from peaks unique to PeakFinder, FindPeaks, MACS, SISSRs, CisGenome, or ChIPseeqer.
when_to_use_negative:
- Input is raw sequencing reads without called peak regions — peak calling (e.g.,
  HPeak on 25 bp binned HDF coverage with the two-state HMM) must be done first to
  produce the FASTA peak sequences this skill scans
- The transcription factor's binding motif is not represented among the 153 vertebrate
  motif matrix families in Genomatix MatBase 7.0, or the motif of interest is novel/absent
  from motif databases
- The goal is a single peak set's absolute motif enrichment rather than a differential
  comparison between caller-unique peak sets
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0102
tools:
- name: HPeak
  role: HMM-based peak caller whose -seq output provides the peak-region FASTA sequences
    and whose caller-unique peaks (e.g., HPeak-unique vs PeakFinder) form the contrast
    being tested
  repo: https://github.com/cauyrd/HPeak
- name: MatInspector
  role: Motif scanner used with default settings to quantify motif occurrence in test
    versus control peak sequences
- name: Genomatix suite
  role: Platform in which MatInspector motif scans are run against Genomatix MatBase
    7.0 matrices
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/1471-2105-11-369
    title: 'HPeak: an HMM-based algorithm for defining read-enriched regions in ChIP-Seq
      data'
  source_content_hash: sha256:dc76bbffd956ed98a22246d8f4346ffd2e3cedecc415d79b030c55d6c254620f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cauyrd/HPeak
  tools:
  - ChIPseeqer
  - CisGenome
  - FindPeaks
  - Genomatix suite
  - HPeak
  - MACS
  - MatInspector
  - PeakFinder
  - SISSRs
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# motif-enrichment-chi-square-testing

## Summary

Quantifies transcription-factor motif enrichment in ChIP-Seq peak-caller-specific peak sets by scanning peak FASTA sequences with MatInspector (Genomatix suite, Genomatix MatBase 7.0 matrices) against length-matched random controls, then applying Chi-square statistics to test whether a factor's motif (e.g., NRSF) distinguishes peaks unique to one caller (HPeak) from peaks unique to another.

## When to use

Use when comparing motif content of peak sets from different ChIP-Seq peak callers — e.g., testing whether the NRSF motif separates HPeak-unique peaks from peaks unique to PeakFinder, FindPeaks, MACS, SISSRs, CisGenome, or ChIPseeqer.

## When NOT to use

- Input is raw sequencing reads without called peak regions — peak calling (e.g., HPeak on 25 bp binned HDF coverage with the two-state HMM) must be done first to produce the FASTA peak sequences this skill scans
- The transcription factor's binding motif is not represented among the 153 vertebrate motif matrix families in Genomatix MatBase 7.0, or the motif of interest is novel/absent from motif databases
- The goal is a single peak set's absolute motif enrichment rather than a differential comparison between caller-unique peak sets

## Inputs

- Peak-region FASTA sequences exported from each peak caller's output (HPeak -seq output; PeakFinder, FindPeaks, MACS, SISSRs, CisGenome, ChIPseeqer peaks)
- NRSF ChIP-Seq dataset (GEO GSE13047) processed by the seven peak callers
- Length-matched random control sequences
- Genomatix MatBase 7.0 vertebrate motif matrix families (153 families)

## Outputs

- Per-peak-set motif occurrence counts for NRSF and other motifs from MatInspector scans
- Chi-square statistics comparing HPeak-unique versus FindPeaks-unique motif enrichment
- peak_caller_comparison_results.csv (peak counts for all seven callers plus Chi-square results)

## How to apply

Export peak-region sequences as FASTA from each peak caller's output (HPeak emits a sequence file for subsequent motif analysis alongside its WIG and annotation files). Scan test and control sequences with MatInspector in the Genomatix suite using all 153 families of vertebrate motif matrices from Genomatix MatBase 7.0 with default settings, and generate length-matched random control sequences for comparison. Count motif occurrences per caller-unique peak set (e.g., the 2,323 peaks significant by HPeak but not PeakFinder on the NRSF data), then compute Chi-square statistics comparing NRSF motif enrichment in HPeak-unique peaks versus FindPeaks-unique peaks. The rationale is that a caller producing more biologically real peaks should enrich the expected transcription-factor binding motif relative to control; HPeak claims this advantage over other available ChIP-Seq analysis approaches. Assemble peak counts for all seven callers with the Chi-square results into a comparison table (peak_caller_comparison_results.csv).

## Related tools

- **HPeak** (HMM-based peak caller whose -seq output provides the peak-region FASTA sequences and whose caller-unique peaks (e.g., HPeak-unique vs PeakFinder) form the contrast being tested) — https://github.com/cauyrd/HPeak
- **MatInspector** (Motif scanner used with default settings to quantify motif occurrence in test versus control peak sequences)
- **Genomatix suite** (Platform in which MatInspector motif scans are run against Genomatix MatBase 7.0 matrices)

## Examples

```
perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN -format BED -t TREATMENT.inp -c CONTROL.inp -n OUTPUTPREFIX -fmin 100 -fmax 300 -r 36 -ann -wig -seq -interfiles
```

## Evaluation signals

- Reproduced peak counts for all seven callers fall within the reported range of 1,935 peaks (PeakFinder) to 5,243 peaks (SISSRs)
- Computed Chi-square statistics for HPeak-unique versus FindPeaks-unique NRSF motif enrichment match the values in the paper's comparison tables
- Among the 2,323 HPeak-significant but not PeakFinder-significant peaks on NRSF data, the NRSF motif ranks as the most enriched relative to length-matched random controls
- MatInspector scan used all 153 vertebrate motif matrix families from Genomatix MatBase 7.0 with default settings
- HPeak peaks show higher expected-motif prevalence relative to control sequences than peaks from the other compared callers

## Limitations

- Many motif patterns stored in the database may not be accurate and there may be novel motifs that do not exist in motif databases; the effect of inaccurate motif patterns on comparison results remains to be seen
- Little information on true positive and true negative loci makes criteria for evaluating peak-calling algorithms on experimental ChIP-Seq data challenging; results should be interpreted with caution
- The method is not well-suited for quantifying false detection rate, which may disadvantage some compared methods

## Evidence

- [discussion] Motif scan of the test and the control sequences was then performed using MatInspector [52] in the Genomatix suite (Genomatix GmBH, Munich, Germany) with default settings: "Motif scan of the test and the control sequences was then performed using MatInspector [52] in the Genomatix suite (Genomatix GmBH, Munich, Germany) with default settings"
- [discussion] all 153 families of vertebrate motif matrices found in the Genomatix MatBase 7.0 database: "all 153 families of vertebrate motif matrices found in the Genomatix MatBase 7.0 database"
- [abstract] HPeak showed a higher prevalence of the expected transcription factor binding motifs in ChIP-enriched sequences relative to the control sequences when compared to other currently available ChIP-Seq: "HPeak showed a higher prevalence of the expected transcription factor binding motifs in ChIP-enriched sequences relative to the control sequences when compared to other currently available ChIP-Seq"
- [discussion] many of the motif patterns stored in the database may not be accurate and there maybe novel motifs that do not exist in motif databases: "many of the motif patterns stored in the database may not be accurate and there maybe novel motifs that do not exist in motif databases"
- [discussion] These include a wiggle (WIG) format file for visualization, a sequence file for subsequent motif analysis and an annotation file: "These include a wiggle (WIG) format file for visualization, a sequence file for subsequent motif analysis and an annotation file"
