---
name: copy-number-variation-calling
description: Use when low-coverage WGS FASTQ/BAM files from tumor samples (e.g. SRA
  study SRP081035, CCLE BioProject PRJNA523380) must be screened for focal amplifications
  (copy number >5, size >100 kb) as candidate eccDNA regions prior to AmpliconArchitect
  reconstruction.
when_to_use_negative:
- Input is not WGS data (e.g. CIRCLE-seq or CIDER-Seq data) — the paper notes few
  such datasets are available and its pipeline relies exclusively on AmpliconArchitect-based
  WGS analysis
- Goal is recovering small eccDNAs below the 100 kb size cutoff or copy-number cutoff
  of 5, since the pipeline's thresholds target high-level focal amplifications only
- Goal is a general, genome-wide CNV profile (e.g. arm-level aneuploidy) rather than
  focal amplifications as eccDNA candidates
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3173
tools:
- name: AmpliconArchitect
  role: Reconstructs focal amplifications and infers circular cycles from the copy-number-filtered
    candidate regions to identify eccDNA elements from WGS data
  repo: github.com/solo7773/eccDNAdb
- name: ReadDepth
  role: Calls copy-number alterations from the BWA-aligned WGS BAMs; candidates with
    copy number >5 and size >100 kb are retained
- name: BWA
  role: Aligns low-coverage WGS reads to the hg19 human reference genome prior to
    copy-number calling
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41388-022-02286-x
    title: 'eccDNAdb: a database of extrachromosomal circular DNA profiles in human
      cancers'
  source_content_hash: sha256:2151259ad0662da7384d1d8469520f1c6456b592dd7113e27ce3abbd9b969ebf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AmpliconArchitect
  - BWA
  - ReadDepth
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# copy-number-variation-calling

## Summary

Detect focal, high-level copy-number amplifications in low-coverage whole-genome sequencing (WGS) data of human cancers using BWA alignment to hg19 and ReadDepth copy-number calling, as the discovery step that feeds eccDNA reconstruction with AmpliconArchitect in the eccDNAdb study.

## When to use

Use when low-coverage WGS FASTQ/BAM files from tumor samples (e.g. SRA study SRP081035, CCLE BioProject PRJNA523380) must be screened for focal amplifications (copy number >5, size >100 kb) as candidate eccDNA regions prior to AmpliconArchitect reconstruction.

## When NOT to use

- Input is not WGS data (e.g. CIRCLE-seq or CIDER-Seq data) — the paper notes few such datasets are available and its pipeline relies exclusively on AmpliconArchitect-based WGS analysis
- Goal is recovering small eccDNAs below the 100 kb size cutoff or copy-number cutoff of 5, since the pipeline's thresholds target high-level focal amplifications only
- Goal is a general, genome-wide CNV profile (e.g. arm-level aneuploidy) rather than focal amplifications as eccDNA candidates

## Inputs

- Low-coverage WGS FASTQ or BAM files from cancer samples (SRA study SRP081035; CCLE WGS BioProject PRJNA523380)
- hg19 human reference genome
- Blacklist regions derived from normal-sample copy-number analyses

## Outputs

- Aligned hg19 BAM files
- Filtered candidate amplified-region list (copy number >5, size >100 kb, blacklist-trimmed)
- AmpliconArchitect-reconstructed focal amplifications with inferred circular cycles
- eccDNA coordinate catalog (chromosome, start, end, copy count), e.g. wgs_ecdna_catalog.tsv (~754 records in TSV/BED format)

## How to apply

Download the WGS FASTQ/BAM files from the public repositories (SRA study SRP081035 and CCLE BioProject PRJNA523380) and align reads to the hg19 human reference genome with BWA. Call copy-number alterations on the aligned BAMs with ReadDepth and retain only candidate amplified regions meeting the paper's thresholds: copy number >5 and size >100 kb. Trim blacklist regions, derived from normal-sample analyses, from the candidate list to remove recurrent artifacts. Pass the filtered candidate regions to AmpliconArchitect to reconstruct focal amplifications and infer circular cycles, converting the cycles into eccDNA coordinate records (chromosome, start, end, copy count). The rationale is that eccDNAs in this study arise from high-level focal amplifications, so stringent copy-number and size filtering followed by blacklist trimming defines the input space for circular reconstruction.

## Related tools

- **AmpliconArchitect** (Reconstructs focal amplifications and infers circular cycles from the copy-number-filtered candidate regions to identify eccDNA elements from WGS data) — github.com/solo7773/eccDNAdb
- **ReadDepth** (Calls copy-number alterations from the BWA-aligned WGS BAMs; candidates with copy number >5 and size >100 kb are retained)
- **BWA** (Aligns low-coverage WGS reads to the hg19 human reference genome prior to copy-number calling)

## Evaluation signals

- The eccDNA catalog is produced as a parseable TSV/BED file with chromosome, start, end, and copy-count columns
- Number of WGS-derived eccDNA records matches the ~754 eccDNAs reported for the SRP081035 + CCLE (PRJNA523380) data scope (out of 1270 total in eccDNAdb)
- Every output eccDNA region satisfies the filter criteria: copy number >5, size >100 kb, and no overlap with normal-sample blacklist regions
- Reconstructed focal amplifications are supported by inferred circular cycles from AmpliconArchitect, not just copy-number peaks

## Limitations

- All eccDNAs in eccDNAdb were identified by the AmpliconArchitect pipeline only, so eccDNAs detectable by other methods (CIRCLE-seq, CIDER-Seq) are not captured
- Thresholds (copy number >5, size >100 kb) bias detection toward large, high-level focal amplifications; small or low-copy eccDNAs are missed
- Blacklist regions derived from normal samples may remove genuine eccDNA loci or incompletely remove artifacts

## Evidence

- [other] WGS-based pipeline identified 754 eccDNAs: [evidence span withheld — anchor status: not_found]
- [discussion] AmpliconArchitect for eccDNA detection: "AmpliconArchitect is a robust and feasible tool for detecting eccDNA elements from WGS data"
- [discussion] Systematic identification via AmpliconArchitect: "we systematically identified eccDNAs in human cancers by analysis of WGS data using AmpliconArchitect"
- [abstract] Analysis scale across tumor samples: "after analyzing a total number of 3395 tumor samples (of 57 cancers) including patient tissues, patient-derived xenografts, and cancer cell lines"
- [readme] Repository provides reproducible analysis scripts: "The source code for the identification of eccDNAs from whole genome sequencing data of human cancers and for the annotation of eccDNA-derived genes."
