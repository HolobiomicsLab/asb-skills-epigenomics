---
name: taqman-probe-transcript-mapping
description: Use when validating RNA-Seq expression estimates against independently
  measured qRT-PCR data, and you must restrict the comparison to transcripts with
  unique TaqMan probe mappings (e.g., the 907 uniquely probed transcripts on MAQC
  HBR).
when_to_use_negative:
- Transcripts whose TaqMan probes map to multiple loci — these must be dropped, not
  correlated, since the qRT-PCR signal cannot be assigned unambiguously
- Benchmarking against data types other than independently measured qRT-PCR (e.g.,
  NanoString measurements), which require their own mapping and normalization treatment
- Expression estimates from alignments other than the same mapped library (SRA012427)
  — corrected and uncorrected estimates must come from identical input alignments
  for the comparison to be valid
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Cufflinks
  role: Produces the transcript-level expression estimates (with and without fragment-bias
    correction) that are correlated against qRT-PCR
- name: Python
  role: Scripting for transcript filtering, Pearson R2 computation, permutation testing,
    and writing qpcr_correlation_results.csv
- name: NanoString
  role: Independent expression measurements included in the supplementary data alongside
    the qRT-PCR benchmark
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/gb-2011-12-3-r22
    title: Improving RNA-Seq expression estimates by correcting for fragment bias
  source_content_hash: sha256:20c7bc4a64eb4d7257f1f00430fda7a78be543456e7fb09f568067bf3d906ece
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cufflinks
  - NanoString
  - Python
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# taqman-probe-transcript-mapping

## Summary

Maps independently performed TaqMan qRT-PCR measurements onto transcript identifiers and retains only the 907 transcripts whose probes map uniquely, so that RNA-Seq expression estimates (e.g., from Cufflinks with and without fragment-bias correction) can be validated by squared Pearson correlation against qRT-PCR.

## When to use

Use when validating RNA-Seq expression estimates against independently measured qRT-PCR data, and you must restrict the comparison to transcripts with unique TaqMan probe mappings (e.g., the 907 uniquely probed transcripts on MAQC HBR).

## When NOT to use

- Transcripts whose TaqMan probes map to multiple loci — these must be dropped, not correlated, since the qRT-PCR signal cannot be assigned unambiguously
- Benchmarking against data types other than independently measured qRT-PCR (e.g., NanoString measurements), which require their own mapping and normalization treatment
- Expression estimates from alignments other than the same mapped library (SRA012427) — corrected and uncorrected estimates must come from identical input alignments for the comparison to be valid

## Inputs

- qRT-PCR (TaqMan) measurement table with probe-to-transcript annotations for the MAQC Human Brain Reference dataset
- Cufflinks expression estimates from SRA012427 alignments without bias correction
- Cufflinks expression estimates from the same SRA012427 alignments with fragment-bias correction enabled

## Outputs

- Filtered set of 907 transcripts whose TaqMan probes map uniquely
- R2 between qRT-PCR and uncorrected Cufflinks estimates (expected 0.753)
- R2 between qRT-PCR and bias-corrected Cufflinks estimates (expected 0.807)
- Permutation-test P-value for the R2 improvement (expected 0.0007), written to qpcr_correlation_results.csv

## How to apply

Load the independently performed qRT-PCR (TaqMan) measurements and their probe-to-transcript annotations, and retain only transcripts whose TaqMan probes map uniquely — on the MAQC Human Brain Reference (HBR) dataset this yields 907 transcripts. Compute the squared Pearson correlation (R2) between qRT-PCR values and uncorrected Cufflinks expression estimates over these transcripts, then compute R2 against the bias-corrected Cufflinks estimates over exactly the same transcript set. Assess significance of the R2 improvement (0.753 uncorrected vs 0.807 corrected) with a permutation test comparing corrected and uncorrected correlations. Rationale: bias correction must be judged against an independent standard, and restricting to uniquely probed transcripts avoids ambiguous measurement-to-transcript assignments that would corrupt the correlation.

## Related tools

- **Cufflinks** (Produces the transcript-level expression estimates (with and without fragment-bias correction) that are correlated against qRT-PCR)
- **Python** (Scripting for transcript filtering, Pearson R2 computation, permutation testing, and writing qpcr_correlation_results.csv)
- **NanoString** (Independent expression measurements included in the supplementary data alongside the qRT-PCR benchmark)

## Examples

```
python -c "from scipy.stats import pearsonr; r,_=pearsonr(qrt_pcr, cufflinks_expr); print(r**2)"  # over the 907 uniquely probed transcripts
```

## Evaluation signals

- Exactly 907 transcripts are retained after keeping only uniquely mapping TaqMan probes on MAQC HBR
- Uncorrected R2 equals the reported 0.753 over the 907-transcript set
- Bias-corrected R2 (0.807) exceeds the uncorrected R2 (0.753)
- Permutation-test P-value equals the reported 0.0007
- Corrected and uncorrected correlations are computed over the identical transcript set

## Limitations

- The validation is limited to the 907 transcripts whose TaqMan probes map uniquely; transcripts with multi-mapping probes are excluded from the benchmark
- The reported R2 values (0.753, 0.807) and P-value (0.0007) are specific to the MAQC Human Brain Reference dataset and SRA012427 alignments; results will differ on other libraries
- Fragment non-uniformity means uncorrected estimates are systematically biased; correlation with qRT-PCR improves only when bias correction is applied during expression estimation

## Evidence

- [abstract] We find improvements in expression estimates as measured by correlation with independently performed qRT-PCR: "We find improvements in expression estimates as measured by correlation with independently performed qRT-PCR"
- [intro] We demonstrate that our method improves expression estimates in comparison with independently obtained qRT-PCR on a benchmark dataset.: "We demonstrate that our method improves expression estimates in comparison with independently obtained qRT-PCR on a benchmark dataset."
- [supplementary] including the NanoString expression measurements: "including the NanoString expression measurements"
- [abstract] The biochemistry of RNA-Seq library preparation results in cDNA fragments that are not uniformly distributed within the transcripts they represent.: "The biochemistry of RNA-Seq library preparation results in cDNA fragments that are not uniformly distributed within the transcripts they represent."
