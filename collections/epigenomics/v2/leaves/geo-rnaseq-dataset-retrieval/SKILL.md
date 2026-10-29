---
name: geo-rnaseq-dataset-retrieval
description: Use when a paper's RNA-seq differential expression claims must be reproduced
  or re-analyzed and the reads are deposited under a named GEO accession (e.g. GSE136191)
  with companion scripts on GitHub.
when_to_use_negative:
- The input is already a processed gene-level count matrix or the authors' final DE
  gene list — retrieval of raw reads is unnecessary.
- The target dataset is ChIP-seq-only occupancy analysis (GSE136050) with no transcript-level
  differential expression goal.
- The study did not deposit reads in GEO or scripts in a public repository, so no
  reproduction path exists.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Perl
  role: Language of the authors' deposited read-processing, reformatting, and peak/gene
    annotation scripts that the retrieved GEO data are fed into
  repo: https://github.com/Juliacvk/LuxR_H-NS
- name: reformatFasta.pl
  role: Cleans and reformats multi-FASTA reference files after data retrieval
  repo: https://github.com/Juliacvk/LuxR_H-NS
- name: FileUtil.pm
  role: Perl helper module for loading retrieved data files in the pipeline
  repo: https://github.com/Juliacvk/LuxR_H-NS
- name: NCBI GEO (GSE136191, GSE136050)
  role: Depository holding the raw RNA-seq and ChIP-seq reads to download
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkz1089
    title: Global H-NS counter-silencing by LuxR activates quorum sensing gene expression
  source_content_hash: sha256:7ebdc953f9b0928a0514d7f8939b8f4f80fb3358409ff2119a83c9ca01faf6e1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/Juliacvk/LuxR_H-NS
  repo_url: https://github.com/Juliacvk/LuxR_H-NS
  tools:
  - FileUtil.pm
  - NCBI GEO (GSE136191, GSE136050)
  - Perl
  - reformatFasta.pl
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# geo-rnaseq-dataset-retrieval

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retrieves raw RNA-seq reads and the authors' deposited analysis scripts from NCBI GEO and GitHub so that a published differential expression comparison (here, the V. harveyi H-NS/LuxR study) can be reproduced from the original FASTQ files rather than summarized counts.

## When to use

Use when a paper's RNA-seq differential expression claims must be reproduced or re-analyzed and the reads are deposited under a named GEO accession (e.g. GSE136191) with companion scripts on GitHub.

## When NOT to use

- The input is already a processed gene-level count matrix or the authors' final DE gene list — retrieval of raw reads is unnecessary.
- The target dataset is ChIP-seq-only occupancy analysis (GSE136050) with no transcript-level differential expression goal.
- The study did not deposit reads in GEO or scripts in a public repository, so no reproduction path exists.

## Inputs

- NCBI GEO accession GSE136191 (RNA-seq raw reads for wild-type, Δhns, ΔluxR, ΔluxR Δhns at OD600=1.0)
- NCBI GEO accession GSE136050 (companion ChIP-seq data, if needed)
- V. harveyi reference genome (multi-FASTA, cleaned via reformatFasta.pl)
- Authors' Perl scripts from github.com/Juliacvk/LuxR_H-NS

## Outputs

- Downloaded raw RNA-seq FASTQ files per strain/condition
- Cleaned and reformatted reference multi-FASTA files
- Gene-level count matrix supporting downstream differential expression (wild-type vs Δhns; ΔluxR vs ΔluxR Δhns)

## How to apply

Locate the NCBI GEO accessions cited in the paper's supplementary section — for this study, GSE136191 for RNA-seq (and GSE136050 for ChIP-seq, if re-analyzing occupancy data). Download the raw reads for the relevant strain/condition subseries (wild-type, Δhns, ΔluxR, and ΔluxR Δhns grown to OD600=1.0) and clone the authors' Perl repository, github.com/Juliacvk/LuxR_H-NS, since 'All Perl scripts have been deposited in Github'. Use the README's file inventory (FileUtil.pm, reformatFasta.pl, findOverlaps.pl, getPeakAnno.pl, getGenePeakAnno2.pl, findClosest2.pl) to identify which scripts handle FASTA/read reformatting versus peak-to-gene annotation before mapping to the V. harveyi reference genome. The rationale is that reproducing the >2-fold, P<0.05 regulon calls (726 genes wild-type; 824 genes in the ΔluxR background) requires the exact read-processing and counting pipeline the authors deposited, not just the accession metadata.

## Related tools

- **Perl** (Language of the authors' deposited read-processing, reformatting, and peak/gene annotation scripts that the retrieved GEO data are fed into) — https://github.com/Juliacvk/LuxR_H-NS
- **reformatFasta.pl** (Cleans and reformats multi-FASTA reference files after data retrieval) — https://github.com/Juliacvk/LuxR_H-NS
- **FileUtil.pm** (Perl helper module for loading retrieved data files in the pipeline) — https://github.com/Juliacvk/LuxR_H-NS
- **NCBI GEO (GSE136191, GSE136050)** (Depository holding the raw RNA-seq and ChIP-seq reads to download)

## Examples

```
git clone https://github.com/Juliacvk/LuxR_H-NS && wget -r ftp://ftp.ncbi.nlm.nih.gov/geo/series/GSE136nnn/GSE136191/suppl/
```

## Evaluation signals

- All four strain samples at OD600=1.0 (wild-type, Δhns, ΔluxR, ΔluxR Δhns) from GSE136191 are present and readable as FASTQ.
- The cloned repository contains the six Perl files listed in the README: FileUtil.pm, reformatFasta.pl, findOverlaps.pl, getPeakAnno.pl, getGenePeakAnno2.pl, findClosest2.pl.
- Downstream differential expression reproduced with the retrieved data yields 726 H-NS-regulated genes (596 repressed) wild-type vs Δhns and 824 genes (430 repressed) in the ΔluxR background, with luxCDABE recovered in the ΔluxR comparison.
- The luxC fold change matches the reported ~1.3-fold change, confirming the retrieved reads and pipeline match the publication.

## Limitations

- The deposited Perl scripts are described only by a short README inventory; no usage documentation or parameter defaults are provided, so script behavior must be inferred from the paper's workflow.
- Reproduction is constrained to the deposited strains/conditions (OD600=1.0); other growth phases or accessions are not covered.
- No changelog was found and metadata (DOI, title, authors) is absent from the raw README, so provenance must be established from the paper itself.

## Evidence
