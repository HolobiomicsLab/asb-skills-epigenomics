---
name: rna-seq-expression-estimation
description: Use when RNA-Seq alignments (SAM) and a reference (FASTA/GTF annotation)
  must yield expression estimates whose fragment sequence, priming, or position bias
  is learned from the data rather than assumed uniform, e.g. MAQC HBR or strand-specific
  SRA020818 yeast libraries.
when_to_use_negative:
- When reads are already summarized into a feature/count table with no alignment-level
  fragment information — the likelihood framework operates on fragment alignments
  (SAM)
- When the library preparation is known to produce uniformly distributed fragments
  and only a raw count baseline is needed, since bias estimation adds complexity without
  expected gain
- When no reference transcriptome/annotation is available, as simultaneous bias and
  expression estimation is transcriptome-guided
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: eXpress-D
  role: Runs the likelihood-based simultaneous estimation of bias parameters and expression
    levels on mseq-formatted fragment alignments (distributed ambiguous fragment assignment)
  repo: https://github.com/adarob/express-d/wiki
- name: Python
  role: Provides the conversion script that converts standard SAM and FASTA input
    into the mseq input format
- name: Bowtie
  role: Remaps MAQC HBR reads to the transcriptome before training/applying the mseq
    MART regression model
- name: mseq
  role: MART regression model trained on the 100 most highly expressed transcripts
    to produce uncorrected and corrected expression estimates for benchmark comparison
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1186/gb-2011-12-3-r22
    title: Improving RNA-Seq expression estimates by correcting for fragment bias
  source_content_hash: sha256:95d15a1d34f409db2324ed042804f9b6d402e922e6b26ed0a26a2c4ee90f63b7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bowtie
  - Genominator
  - Python
  - eXpress-D
  - mseq
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# rna-seq-expression-estimation

## Summary

Likelihood-based simultaneous estimation of fragment-bias parameters and RNA-Seq expression levels (eXpress-D / mseq), which corrects non-uniform cDNA fragment distribution within transcripts and improves the correlation of estimates with independently performed qRT-PCR.

## When to use

Use when RNA-Seq alignments (SAM) and a reference (FASTA/GTF annotation) must yield expression estimates whose fragment sequence, priming, or position bias is learned from the data rather than assumed uniform, e.g. MAQC HBR or strand-specific SRA020818 yeast libraries.

## When NOT to use

- When reads are already summarized into a feature/count table with no alignment-level fragment information — the likelihood framework operates on fragment alignments (SAM)
- When the library preparation is known to produce uniformly distributed fragments and only a raw count baseline is needed, since bias estimation adds complexity without expected gain
- When no reference transcriptome/annotation is available, as simultaneous bias and expression estimation is transcriptome-guided

## Inputs

- SAM alignment files of RNA-Seq reads (single- or paired-end, strand-specific or non-specific)
- FASTA reference / transcript sequences (e.g. Ensembl S. cerevisiae release 59 annotation)
- Python conversion script (SAM+FASTA -> mseq input format)
- Optionally, independent qRT-PCR reference expression values (e.g. MAQC HBR) for validation

## Outputs

- Expression estimates per transcript, with and without learned bias correction
- Learned fragment-bias parameters
- R-squared values against qRT-PCR (e.g. mseq 0.730 uncorrected / 0.755 corrected; Genominator 0.711 / 0.715)
- Fraction of protocol-specific discrepancy explained by correction (33.5% for NSR vs qRT-PCR)

## How to apply

First convert standard SAM alignments and the FASTA reference into the mseq input format using the provided Python conversion script from the supplementary material. Then run eXpress-D's likelihood-based simultaneous estimation of bias parameters and expression levels — for the yeast benchmark against the Ensembl S. cerevisiae release 59 annotation; for the MAQC HBR benchmark remap reads to the transcriptome with Bowtie and train the mseq MART regression model on the 100 most highly expressed transcripts. As a decision point, also run the same estimation without bias correction (the RH control setting) so corrected and uncorrected estimates can be compared. Validate by computing R-squared against independent qRT-PCR measurements: expect mseq ~0.730 uncorrected vs ~0.755 corrected on MAQC HBR, and note that bias correction explained 33.5% of the NSR-vs-qRT-PCR discrepancy. The method must remain robust to single- and paired-end reads, strand-specific and non-specific protocols, and varied priming/fragmentation methods.

## Related tools

- **eXpress-D** (Runs the likelihood-based simultaneous estimation of bias parameters and expression levels on mseq-formatted fragment alignments (distributed ambiguous fragment assignment)) — https://github.com/adarob/express-d/wiki
- **Python** (Provides the conversion script that converts standard SAM and FASTA input into the mseq input format)
- **Bowtie** (Remaps MAQC HBR reads to the transcriptome before training/applying the mseq MART regression model)
- **mseq** (MART regression model trained on the 100 most highly expressed transcripts to produce uncorrected and corrected expression estimates for benchmark comparison)

## Evaluation signals

- Corrected expression estimates show improved R-squared correlation with independently performed qRT-PCR (mseq 0.730 -> 0.755 on MAQC HBR)
- The 33.5% fraction of the NSR-vs-qRT-PCR discrepancy explained by bias correction is reproduced
- Corrected estimates outperform the uncorrected RH control setting for the same library
- The Python conversion script output validates as valid mseq input format before estimation
- Bias correction improves replicability across libraries prepared with different protocols (strand-specific SRA020818 vs NSR-primed)

## Limitations

- Requires robustness across single- and paired-end reads, strand-specific and non-specific protocols, and a variety of priming and fragmentation methods; unusual protocols may behave differently
- Bias correction explained only 33.5% of the NSR-vs-qRT-PCR discrepancy, so a substantial portion remains unexplained
- The mseq regression variant depends on having enough highly expressed transcripts (the 100 most highly expressed) for training

## Evidence

- [intro] Our main result is the development of a likelihood based approach for simultaneous estimation of bias parameters and expression levels: "Our main result is the development of a likelihood based approach for simultaneous estimation of bias parameters and expression levels using the likelihood framework of [6]."
- [abstract] This non-uniformity must be accounted for when estimating expression levels: "This non-uniformity must be accounted for when estimating expression levels, and we show how to perform the needed corrections using a likelihood based approach"
- [abstract] We find improvements in expression estimates as measured by correlation with independently performed qRT-PCR: "We find improvements in expression estimates as measured by correlation with independently performed qRT-PCR"
- [supplementary] Python script that converts standard SAM and FASTA input into the mseq input format: "Python script that converts standard SAM and FASTA input into the mseq input format."
- [readme] Distributed ambiguous fragment assignment using eXpress and Spark: "Distributed ambiguous fragment assignment using [eXpress](http://bio.math.berkeley.edu/eXpress) and [Spark](http://spark.incubator.apache.org/)."
