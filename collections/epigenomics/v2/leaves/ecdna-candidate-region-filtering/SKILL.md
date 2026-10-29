---
name: ecdna-candidate-region-filtering
description: Use when low-coverage WGS of tumor samples has been aligned (e.g., BWA
  to hg19) and focal amplification candidates must be narrowed before AmpliconArchitect
  eccDNA reconstruction.
when_to_use_negative:
- Input is not WGS-derived copy-number data — the paper's pipeline identified eccDNAs
  only via AmpliconArchitect on WGS, not from CIRCLE-seq or CIDER-Seq data
- You need to capture small eccDNAs below the 100 kb size cutoff, since the filtering
  thresholds will exclude them
- Candidate regions have already been filtered and reconstructed by AmpliconArchitect
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3173
tools:
- name: ReadDepth
  role: Calls copy-number alterations from BWA-aligned low-coverage WGS; its output
    is the candidate list filtered at copy number >5 and size >100 kb
- name: AmpliconArchitect
  role: Consumes the filtered candidate regions to reconstruct focal amplifications
    and infer circular cycles
  repo: github.com/solo7773/eccDNAdb
- name: BWA
  role: Aligns WGS reads to the hg19 reference genome prior to copy-number calling
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41388-022-02286-x
    title: 'eccDNAdb: a database of extrachromosomal circular DNA profiles in human
      cancers'
  source_content_hash: sha256:fef23e43068ad361b07e0ef99ca9d5a794f8e96da8e18df64e4b7930e457ee3f
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
yes_rate: 0.9
n_claims: 10
---

# ecdna-candidate-region-filtering

## Summary

Filters copy-number alteration calls from low-coverage WGS to retain candidate regions (copy number >5, size >100 kb) as AmpliconArchitect input for eccDNA reconstruction, removing blacklist regions derived from normal-sample analyses.

## When to use

Use when low-coverage WGS of tumor samples has been aligned (e.g., BWA to hg19) and focal amplification candidates must be narrowed before AmpliconArchitect eccDNA reconstruction.

## When NOT to use

- Input is not WGS-derived copy-number data — the paper's pipeline identified eccDNAs only via AmpliconArchitect on WGS, not from CIRCLE-seq or CIDER-Seq data
- You need to capture small eccDNAs below the 100 kb size cutoff, since the filtering thresholds will exclude them
- Candidate regions have already been filtered and reconstructed by AmpliconArchitect

## Inputs

- BAM or FASTQ files from low-coverage WGS of tumor samples (e.g., SRA SRP081035, CCLE PRJNA523380)
- BWA alignments to the hg19 human reference genome
- ReadDepth copy-number alteration calls
- Blacklist region file derived from normal-sample analyses

## Outputs

- Filtered candidate region list (copy number >5, size >100 kb) suitable for AmpliconArchitect input
- Downstream eccDNA coordinate catalog (chromosome, start, end, copy count), e.g., a ~754-entry wgs_ecdna_catalog.tsv

## How to apply

Align WGS reads (FASTQ/BAM from sources such as SRA study SRP081035 or CCLE BioProject PRJNA523380) to the hg19 human reference genome with BWA, then call copy-number alterations with ReadDepth. Retain only candidate regions meeting the paper's thresholds: copy number >5 and size >100 kb, which enriches for focal amplifications likely to harbor eccDNAs. Trim blacklist regions identified from normal-sample analyses from the candidate list to remove recurrent artifacts. The filtered candidate regions are then passed to AmpliconArchitect to reconstruct focal amplifications and infer circular cycles, which are converted to eccDNA coordinate records (chromosome, start, end, copy count).

## Related tools

- **ReadDepth** (Calls copy-number alterations from BWA-aligned low-coverage WGS; its output is the candidate list filtered at copy number >5 and size >100 kb)
- **AmpliconArchitect** (Consumes the filtered candidate regions to reconstruct focal amplifications and infer circular cycles) — github.com/solo7773/eccDNAdb
- **BWA** (Aligns WGS reads to the hg19 reference genome prior to copy-number calling)

## Examples

```
bwa mem hg19.fa sample.fastq | samtools sort -o sample.bam && ReadDepth.py -b sample.bam; python filter_candidates.py --cn 5 --min-size 100000 --blacklist normal_blacklist.bed > candidates.bed
```

## Evaluation signals

- All retained regions have copy number >5 and size >100 kb (invariant of the two thresholds)
- No region overlapping the normal-sample-derived blacklist remains in the output
- Downstream AmpliconArchitect run produces a parseable TSV/BED eccDNA catalog
- Number of reconstructed eccDNA records matches the ~754 WGS-derived eccDNAs reported for the SRP081035 + CCLE PRJNA523380 data scope
- Total WGS-derived (754) plus manually curated (516) eccDNAs reconciles to the 1270 entries in eccDNAdb

## Limitations

- All eccDNAs in the database were identified by the AmpliconArchitect pipeline only; alternative eccDNA detection methods (CIRCLE-seq, CIDER-Seq) are limited by scarce sequencing data
- Thresholds (copy number >5, size >100 kb) bias toward large focal amplifications and will miss smaller or lower-amplification eccDNAs
- Results depend on hg19 alignment and blacklist quality; blacklist regions must come from normal-sample analyses to remove artifacts

## Evidence

- [methods] Call copy-number alterations with ReadDepth and retain candidate regions with copy number >5 and size >100 kb: [evidence span withheld — anchor status: not_found]
- [discussion] we systematically identified eccDNAs in human cancers by analysis of WGS data using AmpliconArchitect: "we systematically identified eccDNAs in human cancers by analysis of WGS data using AmpliconArchitect"
- [discussion] AmpliconArchitect is a robust and feasible tool for detecting eccDNA elements from WGS data: "AmpliconArchitect is a robust and feasible tool for detecting eccDNA elements from WGS data [25]."
- [readme] The source code for the identification of eccDNAs from whole genome sequencing data of human cancers: "The source code for the identification of eccDNAs from whole genome sequencing data of human cancers and for the annotation of eccDNA-derived genes."
- [discussion] Limitation: all eccDNAs in the database were identified by the AmpliconArchitect pipeline only: "will be recorded in eccDNAdb to overcome current limitation that all eccDNAs in the database were identified by the AmpliconArchitect pipeline"
