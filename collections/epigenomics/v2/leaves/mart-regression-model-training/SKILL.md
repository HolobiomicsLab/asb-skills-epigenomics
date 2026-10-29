---
name: mart-regression-model-training
description: 'Use when reproducing the mseq learned-feature baseline or training a
  MART regression to model RNA-Seq fragment bias: you have Bowtie alignments to a
  transcriptome plus transcript FASTA and need uncorrected vs corrected expression
  estimates validated against qRT-PCR.'
when_to_use_negative:
- If the goal is the paper's own method — likelihood-based simultaneous estimation
  of bias parameters and expression levels (eXpress/eXpress-D) — rather than the mseq
  MART baseline it is compared against.
- When reads cannot be remapped to a transcriptome with Bowtie or transcript FASTA
  is unavailable, since the MART model is trained on the 100 most highly expressed
  mapped transcripts.
- When reproducing the Genominator count-based method with bias learned per chromosome
  instead of the mseq MART regression.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3474
tools:
- name: mseq
  role: Implements the MART regression model trained on the 100 most highly expressed
    transcripts to learn fragment bias and emit uncorrected and corrected expression
    estimates
- name: Bowtie
  role: Remaps the MAQC HBR RNA-Seq reads to the transcriptome to provide the alignments
    used for MART model training
- name: Python
  role: Runs the supplementary script that converts standard SAM and FASTA input into
    the mseq input format
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/gb-2011-12-3-r22
    title: Improving RNA-Seq expression estimates by correcting for fragment bias
  source_content_hash: sha256:2e4d877677f82e1584e25924a468729ae16520933e9dbc20fd998e0d2deff345
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bowtie
  - Genominator
  - Python
  - mseq
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# mart-regression-model-training

## Summary

Trains the MART (Multiple Additive Regression Trees) regression model used by the mseq method on sequence-derived transcript features of the 100 most highly expressed transcripts to learn fragment bias, producing uncorrected and bias-corrected RNA-Seq expression estimates. Serves as the learned-feature baseline that the paper's likelihood-based simultaneous bias/expression approach improves on (mseq R2 0.730 -> 0.755 on MAQC HBR qRT-PCR).

## When to use

Use when reproducing the mseq learned-feature baseline or training a MART regression to model RNA-Seq fragment bias: you have Bowtie alignments to a transcriptome plus transcript FASTA and need uncorrected vs corrected expression estimates validated against qRT-PCR.

## When NOT to use

- If the goal is the paper's own method — likelihood-based simultaneous estimation of bias parameters and expression levels (eXpress/eXpress-D) — rather than the mseq MART baseline it is compared against.
- When reads cannot be remapped to a transcriptome with Bowtie or transcript FASTA is unavailable, since the MART model is trained on the 100 most highly expressed mapped transcripts.
- When reproducing the Genominator count-based method with bias learned per chromosome instead of the mseq MART regression.

## Inputs

- RNA-Seq read alignments to the transcriptome in SAM format (e.g., MAQC HBR reads remapped with Bowtie)
- Transcript sequences in FASTA format
- mseq input format files produced by the supplementary Python conversion script
- Independently measured qRT-PCR reference expression values (MAQC HBR) for validation

## Outputs

- Trained MART regression model over transcript sequence features (learned fragment-bias model)
- Uncorrected and bias-corrected per-transcript expression estimates
- R-squared values vs qRT-PCR for uncorrected and corrected estimates, written to comparison_r2_values.csv

## How to apply

Remap the RNA-Seq reads (e.g., MAQC HBR) to the transcriptome with Bowtie, then use the supplementary Python conversion script to turn the SAM alignments and FASTA transcript sequences into the mseq input format. Select the 100 most highly expressed transcripts and train the mseq MART regression model on their sequence-derived features, so read counts are modeled as a learned function of fragment-sequence bias rather than assuming a parametric bias model. Apply the fitted model to produce both uncorrected (baseline) and bias-corrected expression estimates for all transcripts. Validate by computing R-squared between each estimate set and the independently measured qRT-PCR reference values and writing all four numbers to comparison_r2_values.csv; the reported mseq values on MAQC HBR are 0.730 uncorrected and 0.755 corrected, and corrected must exceed uncorrected. Rationale: the paper argues bias estimation and expression estimation are fundamentally linked, so a learned-feature regression is a stronger comparison point than the count-based Genominator method (0.711 uncorrected / 0.715 corrected).

## Related tools

- **mseq** (Implements the MART regression model trained on the 100 most highly expressed transcripts to learn fragment bias and emit uncorrected and corrected expression estimates)
- **Bowtie** (Remaps the MAQC HBR RNA-Seq reads to the transcriptome to provide the alignments used for MART model training)
- **Python** (Runs the supplementary script that converts standard SAM and FASTA input into the mseq input format)

## Examples

```
bowtie -S transcriptome maqc_hbr.fastq maqc_hbr.sam && python sam_fasta_to_mseq.py --sam maqc_hbr.sam --fasta transcripts.fa --train-mart --top 100
```

## Evaluation signals

- Reproduced mseq R2 vs MAQC HBR qRT-PCR must be ~0.730 uncorrected and ~0.755 corrected, within a small numeric tolerance
- Corrected R2 must exceed uncorrected R2 (0.755 > 0.730), matching the reported improvement from bias correction
- In the same comparison run, the Genominator baseline must land at ~0.711 uncorrected / 0.715 corrected
- Confirm the MART model was trained on exactly the 100 most highly expressed transcripts after Bowtie remapping, not on all transcripts
- All four R2 values (Genominator and mseq, uncorrected and corrected) must be present in comparison_r2_values.csv

## Limitations

- The MART model is trained only on the 100 most highly expressed transcripts, so the learned bias function may not transfer to lowly expressed transcripts.
- mseq learns bias as a step separate from expression estimation, whereas the paper argues the two problems are fundamentally linked and must be solved together, making this a baseline rather than the recommended method.
- Reported gains are modest (R2 0.730 -> 0.755 on MAQC HBR); improvement can be small relative to the Genominator baseline's 0.711 -> 0.715.

## Evidence

- [supplementary] Python script that converts standard SAM and FASTA input into the mseq input format.: "Python script that converts standard SAM and FASTA input into the mseq input format."
- [abstract] We find improvements in expression estimates as measured by correlation with independently performed qRT-PCR: "We find improvements in expression estimates as measured by correlation with independently performed qRT-PCR"
- [other] train the mseq MART regression model on the 100 most highly expressed transcripts: [evidence span withheld — anchor status: not_found]
- [other] mseq 0.730 uncorrected / 0.755 corrected: [evidence span withheld — anchor status: not_found]
- [intro] Using the same data, we also show that our method improves on the approaches of [8,10].: "Using the same data, we also show that our method improves on the approaches of [8,10]."
- [results] the Genominator-based method of [8] yielded R2 = 0.711 before bias correction and R2 = 0.715 after bias correction.: [evidence span withheld — anchor status: not_found]
