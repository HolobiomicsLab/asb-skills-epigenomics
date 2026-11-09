---
name: cycle-to-ecdna-coordinate-conversion
description: Use when AmpliconArchitect has produced reconstructed cycles from WGS
  focal amplifications and you need tabular eccDNA coordinate records for a database
  or BED/TSV catalog.
when_to_use_negative:
- Input eccDNAs come from manual literature curation rather than WGS analysis, since
  eccDNAdb combines both sources (754 WGS-derived + 516 curated) and only the WGS
  branch uses this conversion
- Sequencing data is from CIRCLE-seq or CIDER-Seq, which the paper notes have few
  available datasets and are not covered by the AmpliconArchitect pipeline
- Candidate regions have not been filtered for copy number >5 and size >100 kb, since
  running cycle conversion on unfiltered calls would not match the paper's catalog
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: AmpliconArchitect
  role: Reconstructs focal amplifications and infers circular cycles from WGS data,
    whose cycles are then converted to eccDNA coordinate records
  repo: https://github.com/solo7773/eccDNAdb
- name: eccDNAdb scripts
  role: Numbered scripts in solo7773/eccDNAdb that order the analysis steps and produce/annotate
    the eccDNA catalog
  repo: https://github.com/solo7773/eccDNAdb
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41388-022-02286-x
    title: 'eccDNAdb: a database of extrachromosomal circular DNA profiles in human
      cancers'
  source_content_hash: sha256:39bcc27c7f2ad7a241e11dfdc4afc99c87da8f6790ec94c6cb43920cc3de8afe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/solo7773/eccDNAdb
  tools:
  - AmpliconArchitect
  - eccDNAdb scripts
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cycle-to-ecdna-coordinate-conversion

## Summary

Converts focal amplification cycles reconstructed by AmpliconArchitect into eccDNA coordinate records (chromosome, start, end, copy count), producing a catalog such as the ~754-entry WGS-derived eccDNA set in eccDNAdb.

## When to use

Use when AmpliconArchitect has produced reconstructed cycles from WGS focal amplifications and you need tabular eccDNA coordinate records for a database or BED/TSV catalog.

## When NOT to use

- Input eccDNAs come from manual literature curation rather than WGS analysis, since eccDNAdb combines both sources (754 WGS-derived + 516 curated) and only the WGS branch uses this conversion
- Sequencing data is from CIRCLE-seq or CIDER-Seq, which the paper notes have few available datasets and are not covered by the AmpliconArchitect pipeline
- Candidate regions have not been filtered for copy number >5 and size >100 kb, since running cycle conversion on unfiltered calls would not match the paper's catalog

## Inputs

- Reconstructed AmpliconArchitect cycle output for filtered focal amplification candidate regions (copy number >5, size >100 kb, blacklist-trimmed)
- Low-coverage WGS BAM/FASTQ aligned to hg19 (SRA SRP081035, CCLE PRJNA523380)

## Outputs

- wgs_ecdna_catalog.tsv — a parseable TSV/BED-style catalog of eccDNA records with fields: chromosome, start, end, copy count (~754 entries)

## How to apply

First align low-coverage WGS reads (FASTQ/BAM from SRA study SRP081035 and CCLE BioProject PRJNA523380) to hg19 with BWA, then call copy-number alterations with ReadDepth, retaining candidate regions with copy number >5 and size >100 kb. Trim blacklist regions derived from normal-sample analyses from the candidate list, and run AmpliconArchitect on the filtered candidates to reconstruct focal amplifications and infer circular cycles. Finally, convert each reconstructed cycle to an eccDNA coordinate record containing chromosome, start, end, and copy count, and write the records to a TSV (e.g., wgs_ecdna_catalog.tsv). Rationale: the paper's WGS pipeline used AmpliconArchitect as the sole eccDNA detection tool, and this conversion step turns its cycle output into the database's coordinate catalog, distinct from the 516 literature-curated eccDNAs (754 + 516 = 1270 total in eccDNAdb).

## Related tools

- **AmpliconArchitect** (Reconstructs focal amplifications and infers circular cycles from WGS data, whose cycles are then converted to eccDNA coordinate records) — https://github.com/solo7773/eccDNAdb
- **eccDNAdb scripts** (Numbered scripts in solo7773/eccDNAdb that order the analysis steps and produce/annotate the eccDNA catalog) — https://github.com/solo7773/eccDNAdb

## Examples

```
python 01_convert_cycles_to_ecdna.py --cycles aa_cycles/ --ref hg19 --copy-num 5 --min-size 100000 --blacklist blacklist.bed --out wgs_ecdna_catalog.tsv
```

## Evaluation signals

- The output catalog is a parseable TSV/BED file with chromosome, start, end, and copy count columns
- The number of eccDNA records in the catalog matches the ~754 WGS-derived eccDNAs reported for the SRP081035 + CCLE data scope
- Total eccDNA count combined with the 516 literature-curated eccDNAs reconciles to the 1270 eccDNAs in eccDNAdb
- All catalog records derive from candidate regions that passed copy number >5, size >100 kb, and blacklist trimming

## Limitations

- All eccDNAs in eccDNAdb were identified by the AmpliconArchitect pipeline only, so eccDNAs detectable by other methods (e.g., CIRCLE-seq, CIDER-Seq) may be missed
- Detection depends on low-coverage WGS read depth, limiting sensitivity for low-copy or small circular elements
- Blacklist regions derived from normal-sample analyses are trimmed, potentially discarding true eccDNAs in those loci

## Evidence

- [discussion] AmpliconArchitect is a robust and feasible tool for detecting eccDNA elements from WGS data: "AmpliconArchitect is a robust and feasible tool for detecting eccDNA elements from WGS data [25]."
- [discussion] we systematically identified eccDNAs in human cancers by analysis of WGS data using AmpliconArchitect: "we systematically identified eccDNAs in human cancers by analysis of WGS data using AmpliconArchitect"
- [abstract] eccDNAdb currently includes 1270 eccDNAs, which were identified in 480 samples (of 42 cancers): "eccDNAdb currently includes 1270 eccDNAs, which were identified in 480 samples (of 42 cancers)"
- [readme] The source code for the identification of eccDNAs from whole genome sequencing data of human cancers: "The source code for the identification of eccDNAs from whole genome sequencing data of human cancers and for the annotation of eccDNA-derived genes."
- [discussion] will be recorded in eccDNAdb to overcome current limitation that all eccDNAs in the database were identified by the AmpliconArchitect pipeline: "will be recorded in eccDNAdb to overcome current limitation that all eccDNAs in the database were identified by the AmpliconArchitect pipeline"
