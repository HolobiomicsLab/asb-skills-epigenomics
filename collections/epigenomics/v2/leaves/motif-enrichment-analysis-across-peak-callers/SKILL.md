---
name: motif-enrichment-analysis-across-peak-callers
description: Use when evaluating one or more ChIP-Seq peak-calling algorithms by the
  prevalence of expected transcription factor motifs in peak sequences relative to
  matched control sequences, rather than by sensitivity alone.
when_to_use_negative:
- The transcription factor motif of interest is novel or not represented in curated
  motif databases — the article notes many database motifs may be inaccurate or novel
  motifs may be absent
- The goal is de novo motif discovery from peaks rather than enrichment testing against
  known matrices
- Evaluating a peak caller's false detection rate — the article notes this motif-enrichment
  framework is not well-suited to quantifying false detection rates
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: HPeak
  role: HMM-based peak caller whose predicted peaks (-seq output FASTA) are scanned
    for expected TF binding motifs; also supplies the treated+control vs treated-only
    comparison
  repo: github.com/cauyrd/HPeak
- name: MatInspector
  role: Motif scan of test (peak) and control sequences using default settings against
    Genomatix MatBase 7.0 vertebrate matrices
- name: Genomatix suite
  role: Platform in which MatInspector motif scanning is performed
- name: Perl / C++ HPeak package
  role: Implementation invoked via HPeak.pl to call peaks and emit sequence files
    for downstream motif analysis
  repo: github.com/cauyrd/HPeak
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/1471-2105-11-369
    title: 'HPeak: an HMM-based algorithm for defining read-enriched regions in ChIP-Seq
      data'
  source_content_hash: sha256:4eacfa6a8260dcf30f667900ca66b0b7de403a38e425c5e22230dd3b1cc0e800
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Genomatix suite
  - HPeak
  - MatInspector
  - Perl / C++ HPeak package
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# motif-enrichment-analysis-across-peak-callers

## Summary

Benchmark ChIP-Seq peak callers by scanning sequences under their predicted peaks for expected transcription factor binding motifs and computing enrichment over control sequences, as done for HPeak versus competing callers on the STAT1 ChIP-Seq dataset.

## When to use

Use when evaluating one or more ChIP-Seq peak-calling algorithms by the prevalence of expected transcription factor motifs in peak sequences relative to matched control sequences, rather than by sensitivity alone.

## When NOT to use

- The transcription factor motif of interest is novel or not represented in curated motif databases — the article notes many database motifs may be inaccurate or novel motifs may be absent
- The goal is de novo motif discovery from peaks rather than enrichment testing against known matrices
- Evaluating a peak caller's false detection rate — the article notes this motif-enrichment framework is not well-suited to quantifying false detection rates

## Inputs

- ChIP-Seq treated sample reads (e.g., STAT1 stimulated sample, BED/ELAND format)
- Matched unstimulated control sample reads
- Peak sequences (FASTA) extracted under each caller's predicted peaks
- Matched control-region sequences (FASTA)
- Vertebrate motif matrices (Genomatix MatBase 7.0, all 153 families)

## Outputs

- Motif enrichment scores of peak vs control sequences per peak caller
- Paired comparison of enrichment with vs without control samples (Figure 2B-style plot)
- Peak-count comparison across callers (e.g., ChIPseeqer 9,561 to FindPeaks 41,127)

## How to apply

Run each peak caller (e.g., HPeak with 25 bp genome bins, directional read extension into HDFs with end-weight down-weighting, and a two-state HMM with GP/ZIP emissions via the Viterbi algorithm; competitors spanning ChIPseeqer to FindPeaks) on the same treated sample, once with and once without the unstimulated control. Export the peak sequences for each run (HPeak emits a sequence file via the -seq flag) and scan them with MatInspector in the Genomatix suite using default settings against all 153 families of vertebrate motif matrices in Genomatix MatBase 7.0. Compute motif enrichment of peak sequences relative to control sequences for each caller and each treated+control vs treated-only run. Rationale: higher prevalence of the expected TF motif under predicted peaks is treated as evidence of peak-calling accuracy and resolution, and including control samples is expected to improve enrichment.

## Related tools

- **HPeak** (HMM-based peak caller whose predicted peaks (-seq output FASTA) are scanned for expected TF binding motifs; also supplies the treated+control vs treated-only comparison) — github.com/cauyrd/HPeak
- **MatInspector** (Motif scan of test (peak) and control sequences using default settings against Genomatix MatBase 7.0 vertebrate matrices)
- **Genomatix suite** (Platform in which MatInspector motif scanning is performed)
- **Perl / C++ HPeak package** (Implementation invoked via HPeak.pl to call peaks and emit sequence files for downstream motif analysis) — github.com/cauyrd/HPeak

## Examples

```
perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN -format BED -t TREATMENT.inp -c CONTROL.inp -n OUTPUTPREFIX -fmin 100 -fmax 300 -r 36 -ann -wig -seq -interfiles
```

## Evaluation signals

- HPeak on STAT1 treated+control samples yields 24,394 peaks; treated-only yields 43,440 peaks
- Competitor peak counts fall within the reported range of 9,561 (ChIPseeqer) to 41,127 (FindPeaks)
- Motif enrichment of peak vs control sequences reproduces the Figure 2B pattern: higher enrichment when control samples are included than when the caller is run on treated samples only
- HPeak's enrichment should exceed that of other compared callers, consistent with the paper's claim of higher motif prevalence without sacrificing sensitivity
- Each peak-caller run produces a sequence file (FASTA) suitable for the MatInspector scan before enrichment is computed

## Limitations

- Many motif patterns stored in databases may not be accurate, and novel motifs may not exist in motif databases, biasing enrichment-based comparisons
- Very little information on true positive and true negative loci exists for experimental ChIP-Seq data, making evaluation criteria challenging and results to be interpreted with caution
- The comparison method is not well-suited for quantifying false detection rates, which may disadvantage some compared peak callers

## Evidence

- [discussion] Motif scan of the test and the control sequences was then performed using MatInspector [52] in the Genomatix suite: "Motif scan of the test and the control sequences was then performed using MatInspector [52] in the Genomatix suite"
- [discussion] all 153 families of vertebrate motif matrices found in the Genomatix MatBase 7.0 database: "all 153 families of vertebrate motif matrices found in the Genomatix MatBase 7.0 database"
- [intro] Overall, we demonstrated that HPeak produces higher motif enrichment in the peaks identified without sacrificing sensitivity: "Overall, we demonstrated that HPeak produces higher motif enrichment in the peaks identified without sacrificing sensitivity"
- [readme] Accurate peak calling with detailed information including summit location and p-value: "Accurate peak calling with detailed information including summit location and p-value"
- [discussion] HPeak partitions the entire genome into small bins of fixed length (25 bp in this study): "HPeak partitions the entire genome into small bins of fixed length (25 bp in this study)"
