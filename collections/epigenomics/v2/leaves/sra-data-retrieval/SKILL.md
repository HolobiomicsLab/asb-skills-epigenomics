---
name: sra-data-retrieval
description: Use when an RNA-Seq reproducibility or bias-correction analysis needs
  the original public reads from named SRA accessions (single-end or paired-end library
  preps) before alignment with a splice-aware aligner.
when_to_use_negative:
- Alignments already exist (accepted_hits.bam files are already produced) — the retrieval
  step is only needed to obtain raw reads for re-alignment.
- The target dataset is not deposited in a public SRA accession (e.g. locally generated
  or restricted-access sequencing data).
- The downstream task does not require raw reads, such as re-running only Cufflinks
  bias correction on existing BAM fragment alignments.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3277
- http://edamontology.org/topic_3523
tools:
- name: TopHat
  role: Splice-aware aligner that consumes the retrieved SRA FASTQ reads against the
    hg18 index to produce accepted_hits.bam fragment alignments
- name: Cufflinks
  role: Expression estimator (with and without --bias-correction) applied to alignments
    derived from the retrieved SRA libraries
- name: eXpress-D
  role: Downstream distributed ambiguous fragment assignment pipeline whose alignment
    inputs (accepted_hits.bam) originate from the retrieved SRA reads
  repo: https://github.com/adarob/express-d/wiki
- name: Python
  role: Scripting for downstream steps such as computing pairwise R2 replicate correlations
    and qRT-PCR comparisons from the estimates derived from retrieved data
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1186/gb-2011-12-3-r22
    title: Improving RNA-Seq expression estimates by correcting for fragment bias
  source_content_hash: sha256:118ae9a4e745b6d33328af166c5555cdc8963064019921fe7c7c0637e148f224
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cufflinks
  - Python
  - TopHat
  - eXpress-D
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3277
  - http://edamontology.org/topic_3523
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# sra-data-retrieval

## Summary

Retrieve raw RNA-Seq FASTQ read data from NCBI SRA accessions (e.g. SRA012427 MAQC Human Brain Reference, SRA008403 and SRA010153 MAQC UHR libraries) to serve as the starting alignment input for the fragment-bias-correction pipeline built on TopHat and Cufflinks/eXpress-D.

## When to use

Use when an RNA-Seq reproducibility or bias-correction analysis needs the original public reads from named SRA accessions (single-end or paired-end library preps) before alignment with a splice-aware aligner.

## When NOT to use

- Alignments already exist (accepted_hits.bam files are already produced) — the retrieval step is only needed to obtain raw reads for re-alignment.
- The target dataset is not deposited in a public SRA accession (e.g. locally generated or restricted-access sequencing data).
- The downstream task does not require raw reads, such as re-running only Cufflinks bias correction on existing BAM fragment alignments.

## Inputs

- SRA accession identifiers (SRA012427, SRA008403, SRA010153)
- NCBI SRA public archive as the data source
- Human reference genome NCBI36/hg18 with RefSeq refGene annotation from the UCSC Genome Browser (used alongside the reads)

## Outputs

- FASTQ read files for each SRA library (single-end and paired-end)
- Metadata on library preparation type (single-end vs paired-end) per accession, used to parameterize downstream TopHat/Cufflinks runs

## How to apply

Identify the SRA accessions named in the study — SRA012427 for the Illumina MAQC Human Brain Reference sample, and SRA008403 (single-end UHR library prep) plus SRA010153 (four paired-end UHR library preps) for replicate-correlation analyses — and download the FASTQ reads from the SRA public archive as step 1 of the workflow. Note for each accession whether the library is single-end or paired-end, since this determines how fragments are treated downstream in Cufflinks FPKM estimation with and without --bias-correction. Pair the retrieved reads with the matching human reference genome (NCBI36/hg18) and RefSeq refGene annotation from the UCSC Genome Browser before running TopHat 1.1.0 with --no-novel-juncs. The retrieved FASTQ files are the sole raw-data input; all downstream validation (accepted_hits.bam existence, BAM validity, replicate R2 > 0.96 under bias correction, improved qRT-PCR correlation) depends on fetching the correct accession and library type.

## Related tools

- **TopHat** (Splice-aware aligner that consumes the retrieved SRA FASTQ reads against the hg18 index to produce accepted_hits.bam fragment alignments)
- **Cufflinks** (Expression estimator (with and without --bias-correction) applied to alignments derived from the retrieved SRA libraries)
- **eXpress-D** (Downstream distributed ambiguous fragment assignment pipeline whose alignment inputs (accepted_hits.bam) originate from the retrieved SRA reads) — https://github.com/adarob/express-d/wiki
- **Python** (Scripting for downstream steps such as computing pairwise R2 replicate correlations and qRT-PCR comparisons from the estimates derived from retrieved data)

## Evaluation signals

- Confirm the FASTQ files for the expected accession (e.g. SRA012427) are present and non-empty after download.
- Confirm library-type metadata matches expectations: SRA008403 is a single-end UHR prep and SRA010153 comprises four paired-end UHR preps.
- Downstream check: TopHat on the retrieved reads yields a valid accepted_hits.bam containing mapped alignments (file_exists / file_format_is / script_runs checks).
- Downstream check: Cufflinks bias-corrected FPKM estimates from the retrieved libraries reach average within-SRA010153 pairwise R2 above 0.96, indicating the correct libraries were retrieved and aligned.
- Cross-check that expression estimates from the retrieved reads improve correlation with independently performed qRT-PCR after bias correction, as reported in the paper.

## Limitations

- Retrieval quality cannot be assessed in isolation: errors in accession choice or library-type assignment only surface later as poor replicate correlations or degraded qRT-PCR agreement.
- The workflow depends on external archives (SRA, UCSC Genome Browser downloads for hg18 refGene annotation); version drift (e.g. TopHat 1.1.0, NCBI36/hg18 annotation) can affect reproducibility.
- Only accessions explicitly named in the study (SRA012427, SRA008403, SRA010153) are covered; other MAQC or RNA-Seq datasets require analogous but unverified retrieval parameters.

## Evidence

- [other] Download the SRA012427 Illumina MAQC HBR FASTQ reads from the SRA public archive.: [evidence span withheld — anchor status: not_found]
- [other] Download the MAQC UHR RNA-Seq read data from SRA accession SRA008403 (single-end library prep) and SRA accession SRA010153 (four paired-end library preps): [evidence span withheld — anchor status: not_found]
- [other] the RefSeq refGene NCBI36/hg18 annotation downloaded from the UCSC Genome Browser: [evidence span withheld — anchor status: not_found]
- [abstract] We find improvements in expression estimates as measured by correlation with independently performed qRT-PCR: "We find improvements in expression estimates as measured by correlation with independently performed qRT-PCR"
- [readme] For current documentation, see the eXpress-D Wiki: "For current documentation, see the [eXpress-D Wiki](https://github.com/adarob/express-d/wiki)."
