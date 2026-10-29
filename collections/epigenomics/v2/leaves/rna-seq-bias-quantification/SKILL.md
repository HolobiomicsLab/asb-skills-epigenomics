---
name: rna-seq-bias-quantification
description: Use when RNA-Seq expression estimates (e.g. FPKM from Cufflinks or eXpress)
  show poor agreement across technical replicate libraries or against qRT-PCR, and
  sequence/positional fragment bias is suspected as the cause.
when_to_use_negative:
- The dataset lacks technical replicates or independent qRT-PCR measurements, so correction
  quality cannot be validated as the article's benchmark design requires.
- GC content is assumed a priori to be an adequate bias proxy — the article shows
  this fails in yeast (SRA020818_RH), so GC-only normalization is not a substitute
  for likelihood-based bias correction.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: eXpress-D
  role: Distributed ambiguous fragment assignment engine (with Spark) implementing
    the likelihood-based bias and expression estimation
  repo: https://github.com/adarob/express-d
- name: Python
  role: Converts standard SAM and FASTA input into the mseq input format and computes
    per-transcript bias log fold changes and correlations
- name: Cufflinks
  role: Runs with and without --bias-correction on TopHat BAM alignments to produce
    paired corrected/uncorrected FPKM estimates
- name: TopHat
  role: Splice-aware aligner producing the BAM fragment alignments used as input to
    bias correction
provenance:
  source_task_ids:
  - task_008
  - task_004
  source_papers:
  - doi: 10.1186/gb-2011-12-3-r22
    title: Improving RNA-Seq expression estimates by correcting for fragment bias
  source_content_hash: sha256:fb08c08862838956be346c947d44beb653484a59054020b604b2da48a7f0010f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/adarob/express-d
  tools:
  - Cufflinks
  - Python
  - TopHat
  - eXpress-D
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# rna-seq-bias-quantification

## Summary

Quantify and correct the non-uniform distribution of cDNA fragments within transcripts in RNA-Seq using a likelihood-based joint estimation of bias parameters and expression levels, and validate the correction via replicate agreement and qRT-PCR correlation.

## When to use

Use when RNA-Seq expression estimates (e.g. FPKM from Cufflinks or eXpress) show poor agreement across technical replicate libraries or against qRT-PCR, and sequence/positional fragment bias is suspected as the cause.

## When NOT to use

- The dataset lacks technical replicates or independent qRT-PCR measurements, so correction quality cannot be validated as the article's benchmark design requires.
- GC content is assumed a priori to be an adequate bias proxy — the article shows this fails in yeast (SRA020818_RH), so GC-only normalization is not a substitute for likelihood-based bias correction.

## Inputs

- RNA-Seq read data from SRA accessions (e.g. SRA008403, SRA010153, SRA012427, SRA020818_RH)
- BAM fragment alignments from a splice-aware aligner (TopHat)
- Reference human genome annotation / transcriptome
- SAM and FASTA files (convertible to mseq format via supplied Python script)
- qRT-PCR (and optionally NanoString) expression measurements for validation

## Outputs

- Uncorrected and bias-corrected FPKM expression estimates per transcript
- Per-transcript bias values (log fold change in effective length)
- Per-dataset GC-bias correlation table (e.g. bias_correlations.csv)
- Pairwise replicate R2 and qRT-PCR correlation values before/after correction (e.g. replicate_correlations.csv)

## How to apply

Align reads (e.g. MAQC UHR libraries from SRA008403 single-end and SRA010153 paired-end preps) to a reference transcriptome with a splice-aware aligner such as TopHat, then run Cufflinks twice per alignment — once without and once with --bias-correction — to obtain paired uncorrected and corrected FPKM estimates. Quantify per-transcript bias as the log fold change in effective length between bias-corrected and uncorrected estimates (computed with Python/pandas/numpy), then correlate this bias signal with GC content per dataset (e.g. SRA012427 human vs SRA020818_RH yeast) to test whether GC correction proxies sequence-specific bias. Assess correction quality by computing pairwise R2 across replicate libraries and by correlating corrected versus uncorrected estimates against independently performed qRT-PCR measurements. The rationale, per the article, is that bias estimation and expression estimation are fundamentally linked and must be solved together via a likelihood-based approach.

## Related tools

- **eXpress-D** (Distributed ambiguous fragment assignment engine (with Spark) implementing the likelihood-based bias and expression estimation) — https://github.com/adarob/express-d
- **Python** (Converts standard SAM and FASTA input into the mseq input format and computes per-transcript bias log fold changes and correlations)
- **Cufflinks** (Runs with and without --bias-correction on TopHat BAM alignments to produce paired corrected/uncorrected FPKM estimates)
- **TopHat** (Splice-aware aligner producing the BAM fragment alignments used as input to bias correction)

## Examples

```
cufflinks --bias-correction aligned.bam  # run with and without this flag on each of five UHR TopHat alignments to obtain paired corrected/uncorrected FPKM estimates
```

## Evaluation signals

- bias_correlations.csv exists with per-dataset correlation rows; human (SRA012427) correlation is strong/positive while yeast (SRA020818_RH) correlation is weak.
- Average within-SRA010153 pairwise replicate R2 exceeds 0.96 under bias correction.
- Correlation of expression estimates with qRT-PCR improves after bias correction relative to uncorrected Cufflinks runs.
- Corrected estimates show improved agreement across libraries generated with different sample preparations and sequencing technologies.
- Log fold change in effective length values are finite and computed for every transcript in each dataset.

## Limitations

- GC content is not always an effective marker of sequence-specific bias (fails in yeast SRA020818_RH), so GC-based normalization cannot replace the likelihood-based correction.
- The method must remain robust across single- and paired-end reads, strand specific and non-specific protocols, and a variety of priming and fragmentation methods; assumptions fitting one protocol may not transfer.

## Evidence

- [intro] likelihood based approach for simultaneous estimation of bias parameters and expression levels: "Our main result is the development of a likelihood based approach for simultaneous estimation of bias parameters and expression levels using the likelihood framework of [6]."
- [abstract] improved replicability across libraries and technologies: "correction of bias leads to improved replicability of results across libraries and sequencing technologies"
- [abstract] improvement measured by qRT-PCR correlation: "We find improvements in expression estimates as measured by correlation with independently performed qRT-PCR"
- [supplementary] mseq format conversion script: "Python script that converts standard SAM and FASTA input into the mseq input format."
- [readme] eXpress-D wiki documentation: "For current documentation, see the [eXpress-D Wiki](https://github.com/adarob/express-d/wiki)."
