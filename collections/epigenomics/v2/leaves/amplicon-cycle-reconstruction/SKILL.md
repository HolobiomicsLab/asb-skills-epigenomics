---
name: amplicon-cycle-reconstruction
description: Use when low-coverage WGS FASTQ/BAM files from cancer samples (e.g. SRA
  SRP081035, CCLE PRJNA523380) must be converted into a catalog of eccDNA coordinates
  (chromosome, start, end, copy count), as in the eccDNAdb study of 3395 tumor samples.
when_to_use_negative:
- The goal is to identify eccDNAs from techniques other than WGS/AmpliconArchitect
  — the article notes all eccDNAs in eccDNAdb were identified by the AmpliconArchitect
  pipeline only, and few sequencing datasets are available for CIRCLE-seq and CIDER-Seq
- Input is already a curated eccDNA list (e.g. the 516 eccDNAs collected by manual
  literature curation) rather than raw WGS data
- Samples are not human cancers — eccDNAdb currently covers human cancers only, with
  more species and diseases planned for future updates
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: AmpliconArchitect
  role: Reconstructs focal amplifications and infers circular cycles from filtered
    copy-number candidate regions to produce eccDNA coordinate records
- name: BWA
  role: Aligns low-coverage WGS reads to the hg19 human reference genome prior to
    copy-number calling
- name: ReadDepth
  role: Calls copy-number alterations and yields candidate regions filtered at copy
    number >5 and size >100 kb
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41388-022-02286-x
    title: 'eccDNAdb: a database of extrachromosomal circular DNA profiles in human
      cancers'
  source_content_hash: sha256:cf2e8e9faf38fe08920978c1577013264f7d1b86b9ca66fbf3baee15afe48333
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AmpliconArchitect
  - BWA
  - ReadDepth
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# amplicon-cycle-reconstruction

## Summary

Reconstruct focal amplification cycles as extrachromosomal circular DNA (eccDNA) records from low-coverage cancer WGS data, following the eccDNAdb pipeline: BWA alignment to hg19, ReadDepth copy-number filtering, and AmpliconArchitect cycle inference.

## When to use

Use when low-coverage WGS FASTQ/BAM files from cancer samples (e.g. SRA SRP081035, CCLE PRJNA523380) must be converted into a catalog of eccDNA coordinates (chromosome, start, end, copy count), as in the eccDNAdb study of 3395 tumor samples.

## When NOT to use

- The goal is to identify eccDNAs from techniques other than WGS/AmpliconArchitect — the article notes all eccDNAs in eccDNAdb were identified by the AmpliconArchitect pipeline only, and few sequencing datasets are available for CIRCLE-seq and CIDER-Seq
- Input is already a curated eccDNA list (e.g. the 516 eccDNAs collected by manual literature curation) rather than raw WGS data
- Samples are not human cancers — eccDNAdb currently covers human cancers only, with more species and diseases planned for future updates

## Inputs

- Low-coverage WGS FASTQ or BAM files from cancer samples (e.g. SRA study SRP081035; CCLE WGS BioProject PRJNA523380)
- hg19 human reference genome
- Blacklist regions derived from normal-sample analyses

## Outputs

- eccDNA coordinate catalog (wgs_ecdna_catalog.tsv) with per-record chromosome, start, end, and copy count, in parseable TSV/BED format

## How to apply

Align WGS reads to the hg19 human reference with BWA, then call copy-number alterations with ReadDepth and retain only candidate regions with copy number >5 and size >100 kb. Trim blacklist regions previously identified from normal-sample analyses from the candidate list before reconstruction, to avoid false eccDNA calls. Run AmpliconArchitect on the filtered candidate regions to reconstruct focal amplifications and infer circular cycles — the article notes AmpliconArchitect is 'a robust and feasible tool for detecting eccDNA elements from WGS data'. Convert the reconstructed cycles into eccDNA coordinate records (chromosome, start, end, copy count) written to a parseable TSV/BED catalog (wgs_ecdna_catalog.tsv); in eccDNAdb this scope yielded ~754 WGS-derived eccDNAs out of the 1270 total, the remainder coming from manual literature curation. Validate by confirming the catalog parses and the record count matches the expected ~754 for this data scope.

## Related tools

- **AmpliconArchitect** (Reconstructs focal amplifications and infers circular cycles from filtered copy-number candidate regions to produce eccDNA coordinate records)
- **BWA** (Aligns low-coverage WGS reads to the hg19 human reference genome prior to copy-number calling)
- **ReadDepth** (Calls copy-number alterations and yields candidate regions filtered at copy number >5 and size >100 kb)

## Evaluation signals

- The output catalog exists as a parseable TSV/BED file with chromosome, start, end, and copy-count columns
- The number of eccDNA records matches the ~754 WGS-derived eccDNAs reported for this data scope (out of 1270 total eccDNAdb entries, with 516 from literature curation)
- All candidate regions entering AmpliconArchitect satisfy copy number >5 and size >100 kb
- Blacklist regions from normal-sample analyses have been removed from the candidate list before reconstruction
- Every output eccDNA record traces back to a reconstructed circular cycle from AmpliconArchitect, not to literature curation

## Limitations

- All eccDNAs in eccDNAdb were identified by the AmpliconArchitect pipeline only, which the authors plan to overcome in future updates
- Few sequencing datasets are currently available for CIRCLE-seq and CIDER-Seq techniques, so cross-method validation is limited
- Currently restricted to human cancers; more species and diseases are planned for future updates

## Evidence

- [discussion] AmpliconArchitect is a robust and feasible tool for detecting eccDNA elements from WGS data: "AmpliconArchitect is a robust and feasible tool for detecting eccDNA elements from WGS data [25]."
- [discussion] Systematic eccDNA identification from WGS with AmpliconArchitect: "we systematically identified eccDNAs in human cancers by analysis of WGS data using AmpliconArchitect"
- [abstract] eccDNAdb composition and scale: "eccDNAdb currently includes 1270 eccDNAs, which were identified in 480 samples (of 42 cancers)"
- [readme] Repository provides ordered analysis scripts for reproducing the pipeline: "The source code for the identification of eccDNAs from whole genome sequencing data of human cancers and for the annotation of eccDNA-derived genes."
