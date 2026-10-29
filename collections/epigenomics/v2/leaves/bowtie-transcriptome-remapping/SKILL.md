---
name: bowtie-transcriptome-remapping
description: Use when reproducing transcript-level expression baselines (e.g. mseq)
  that need reads aligned directly to transcript sequences instead of a genome, or
  when comparing against count-based methods on MAQC HBR qRT-PCR benchmark data.
when_to_use_negative:
- The target method (e.g. eXpress/Cufflinks-style likelihood estimation) performs
  its own alignment handling — this remapping is specifically to feed the mseq baseline,
  not the paper's main method
- Reads must be compared at gene level via a count-based approach such as Genominator,
  which uses its own count-based pipeline rather than transcriptome remapping
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3520
tools:
- name: Bowtie
  role: Aligns MAQC HBR reads against the transcriptome to generate transcript-level
    SAM alignments consumed by mseq
- name: Python
  role: Conversion script that transforms standard SAM and FASTA input into the mseq
    input format
- name: mseq
  role: MART regression model trained on the 100 most highly expressed transcripts;
    applied to remapped reads to yield uncorrected and bias-corrected expression estimates
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/gb-2011-12-3-r22
    title: Improving RNA-Seq expression estimates by correcting for fragment bias
  source_content_hash: sha256:95cf5cf16ecb182018f033413fcc382e73bac730dabcc39395e993a3d64e8fc7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bowtie
  - Genominator
  - Python
  - mseq
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# bowtie-transcriptome-remapping

## Summary

Remaps RNA-Seq reads to the transcriptome (rather than the genome) with Bowtie to produce transcript-level alignments required for the mseq bias-correction baseline, enabling head-to-head R-squared comparison against the Genominator count-based method on the MAQC HBR qRT-PCR benchmark.

## When to use

Use when reproducing transcript-level expression baselines (e.g. mseq) that need reads aligned directly to transcript sequences instead of a genome, or when comparing against count-based methods on MAQC HBR qRT-PCR benchmark data.

## When NOT to use

- The target method (e.g. eXpress/Cufflinks-style likelihood estimation) performs its own alignment handling — this remapping is specifically to feed the mseq baseline, not the paper's main method
- Reads must be compared at gene level via a count-based approach such as Genominator, which uses its own count-based pipeline rather than transcriptome remapping

## Inputs

- MAQC HBR RNA-Seq read alignments / raw reads (SAM format)
- Transcriptome sequences in FASTA format
- Independently measured qRT-PCR reference expression values for the MAQC HBR benchmark

## Outputs

- Transcript-level SAM alignments from Bowtie
- mseq input-format files produced by the Python conversion script
- Uncorrected and bias-corrected transcript expression estimates
- R-squared values versus qRT-PCR (target: 0.730 uncorrected, 0.755 corrected for mseq), e.g. written to comparison_r2_values.csv

## How to apply

Take the MAQC HBR RNA-Seq read data and the transcript sequences (FASTA) used in the paper's Comparison With Previous Methods section. Build a Bowtie index on the transcriptome rather than the genome, since the mseq MART regression model operates on transcript-level fragment observations. Align the reads with Bowtie to obtain transcript-mapped SAM alignments, then convert the SAM and FASTA inputs into the mseq input format using the Python conversion script from the supplementary material. Train the mseq MART regression model on the 100 most highly expressed transcripts with bias parameters learned per chromosome, producing expression estimates both before and after bias correction. Finally compute R-squared against the independently measured qRT-PCR reference values and compare with the Genominator baseline (0.711 uncorrected / 0.715 corrected) and mseq targets (0.730 uncorrected / 0.755 corrected).

## Related tools

- **Bowtie** (Aligns MAQC HBR reads against the transcriptome to generate transcript-level SAM alignments consumed by mseq)
- **Python** (Conversion script that transforms standard SAM and FASTA input into the mseq input format)
- **mseq** (MART regression model trained on the 100 most highly expressed transcripts; applied to remapped reads to yield uncorrected and bias-corrected expression estimates)

## Examples

```
bowtie -v 2 transcriptome_index -1 maqc_hbr_1.fq -2 maqc_hbr_2.fq maqc_hbr_tx.sam && python convert_sam_to_mseq.py --sam maqc_hbr_tx.sam --fasta transcripts.fa --out mseq_input.txt
```

## Evaluation signals

- Reproduced mseq R-squared vs MAQC qRT-PCR matches reported 0.730 (uncorrected) and 0.755 (corrected) within small numeric tolerance
- Bias-corrected estimates improve on uncorrected ones (0.755 > 0.730), consistent with the reported direction of improvement
- Bowtie alignments are to the transcriptome, not the genome — verify alignment targets correspond to transcript FASTA entries
- All four baseline R-squared values (Genominator 0.711/0.715, mseq 0.730/0.755) are written to comparison_r2_values.csv
- mseq corrected R-squared exceeds the Genominator baseline (0.715 corrected), as reported in the paper's comparison

## Limitations

- mseq is a baseline from a prior approach [8]; the paper shows its own likelihood-based method improves on it, so this skill reproduces a comparison baseline rather than state-of-the-art estimation
- Values are dataset-specific to MAQC HBR with bias learned per chromosome; do not expect identical R-squared values on other datasets or sequencing protocols
- Requires the supplementary Python conversion script to bridge SAM/FASTA to the mseq input format

## Evidence

- [intro] Objective: show the method improves on the approaches of [8,10] using the same data: "Using the same data, we also show that our method improves on the approaches of [8,10]."
- [intro] MAQC qRT-PCR validation benchmark: "We demonstrate that our method improves expression estimates in comparison with independently obtained qRT-PCR on a benchmark dataset."
- [supplementary] Python SAM/FASTA conversion script: "Python script that converts standard SAM and FASTA input into the mseq input format."
- [abstract] Bias correction improves expression estimate correlation: "We find improvements in expression estimates as measured by correlation with independently performed qRT-PCR"
