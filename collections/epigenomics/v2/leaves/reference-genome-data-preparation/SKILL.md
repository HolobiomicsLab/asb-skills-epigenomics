---
name: reference-genome-data-preparation
description: Use when setting up ChiLin (or a bwa-based ChIP-seq QC pipeline) for
  a new species/build and the dependent data in db/ plus a mapped-ready genome index
  are not yet installed.
when_to_use_negative:
- The db/ dependent data for the target species/build is already extracted and md5-verified
  — skip to pipeline execution.
- Your mapper is not bwa/bowtie/star-compatible with the reference build (e.g. long-read
  ONN/PacBio data where BWA-MEM settings or minimap2 would be needed).
- The task concerns raw read QC (FastQC) or peak calling rather than preparing reference
  assets — reference prep alone does not produce QC metrics.
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0091
tools:
- name: ChiLin
  role: Pipeline whose dependent reference data (db/ archives) and installation this
    skill prepares; provides hg19/hg38/mm9/mm10 downloads and demo data
  repo: https://github.com/cfce/chilin
- name: BWA
  role: Default mapper in ChiLin step 2; its genome index is built from the reference
    fasta during this preparation
  repo: https://github.com/lh3/bwa
- name: seqtk
  role: Lightweight FASTA/FASTQ utility available in the prepared environment (e.g.
    for reference/read sequence manipulation)
  repo: https://github.com/lh3/seqtk
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-016-1274-4
    title: 'ChiLin: a comprehensive ChIP-seq and DNase-seq quality control and analysis
      pipeline'
  source_content_hash: sha256:2b4fba66599a4e63b327fab80b63811708cb000831fc4a26511e532dde491875
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfce/chilin
  tools:
  - BWA
  - ChiLin
  - bedtools
  - samtools
  - seqtk
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# reference-genome-data-preparation

## Summary

Prepare species-specific reference data (hg19/hg38/mm9/mm10 dependent-data archives, Mycoplasma fasta index) and the BWA genome index required before running the ChiLin ChIP-seq/DNase-seq QC pipeline, with checksum verification of each download.

## When to use

Use when setting up ChiLin (or a bwa-based ChIP-seq QC pipeline) for a new species/build and the dependent data in db/ plus a mapped-ready genome index are not yet installed.

## When NOT to use

- The db/ dependent data for the target species/build is already extracted and md5-verified — skip to pipeline execution.
- Your mapper is not bwa/bowtie/star-compatible with the reference build (e.g. long-read ONN/PacBio data where BWA-MEM settings or minimap2 would be needed).
- The task concerns raw read QC (FastQC) or peak calling rather than preparing reference assets — reference prep alone does not produce QC metrics.

## Inputs

- Species/build identifier (hg19, hg38, mm9, mm10)
- Dependent-data archive from cistrome.org (e.g. hg19.tgz) with its md5 checksum
- Mycoplasma genome fasta (for contamination screening)
- Reference genome fasta (e.g. ref.fa) for BWA indexing

## Outputs

- Extracted db/ directory of species-dependent data (QC references, PhastCons bigwiggles, motif databases)
- md5sum-verified archive integrity report
- BWA genome index files (ref.fa.* index set)
- Indexed Mycoplasma genome for contamination detection

## How to apply

Clone the ChiLin repository and install (e.g. 'python setup.py install -f'), then download the species-specific dependent-data archive matching your reference build (hg19.tgz, hg38.tgz, mm9.tgz or mm10.tgz from http://cistrome.org/chilin/_downloads/) into a db/ directory. Verify each archive's integrity with 'md5sum -c' before extraction, since corrupt reference data silently breaks downstream FRiP, meta-region, and conservation steps. Also download and index the Mycoplasma fasta so library contamination can be screened. Finally build the BWA genome index ('bwa index ref.fa'), because step 2 of the pipeline uses bwa (default) or bowtie/star mapping and cannot be skipped — the reference index must exist before the FASTQ treatment/control replicates are mapped.

## Related tools

- **ChiLin** (Pipeline whose dependent reference data (db/ archives) and installation this skill prepares; provides hg19/hg38/mm9/mm10 downloads and demo data) — https://github.com/cfce/chilin
- **BWA** (Default mapper in ChiLin step 2; its genome index is built from the reference fasta during this preparation) — https://github.com/lh3/bwa
- **seqtk** (Lightweight FASTA/FASTQ utility available in the prepared environment (e.g. for reference/read sequence manipulation)) — https://github.com/lh3/seqtk

## Examples

```
git clone https://github.com/cfce/chilin; cd chilin; wget -c http://cistrome.org/chilin/_downloads/hg19.tgz -P db/; cd db && md5sum -c hg19.tgz.md5 && tar xzf hg19.tgz; ./bwa index ref.fa
```

## Evaluation signals

- 'md5sum -c' reports OK for every downloaded archive before extraction
- db/ directory contains the extracted species-specific data and 'chilin -h' runs after installation
- BWA index files (e.g. .amb, .ann, .bwt, .pac, .sa) exist alongside ref.fa
- Demo run (demo/foxa1 or 'bash demo/docker_test.sh') completes mapping without missing-reference errors
- Mycoplasma fasta is indexed so step 1 contamination screening can execute

## Limitations

- ChiLin is Python 2-based and pins old tool versions (bwa 0.7.13, samtools 0.1.19, bedtools 2.17.0, macs2 2.1.0); a conda env (chilin_env) with those exact versions is expected
- Archives are hosted on cistrome.org; slow or broken network links may require the Docker image (qq06/chilin:1.0) as an alternative
- No software version for ChiLin itself is stated, and the paper notes reproducibility claims without concrete environment specification

## Evidence

- [methods] wget -c http://cistrome.org/chilin/_downloads/hg19.tgz: "wget -c http://cistrome.org/chilin/_downloads/hg19.tgz"
- [methods] Mycoplasma genome download: "download the Mycoplasma fasta for indexing, data is in the link of the `mycoplasma genome`_"
- [methods] bwa mapping step: "step 2: bwa(default), bowtie or star mapping, this step cannot be skipped, because this provided necessary BAM files."
- [readme] BWA indexing: "git clone https://github.com/lh3/bwa.git
	cd bwa; make
	./bwa index ref.fa"
- [readme] conda environment pinning: "conda install bwa=0.7.13 samtools=0.1.19 bedtools=2.17.0 seqtk ucsc-bedclip ucsc-bedgraphtobigwig ucsc-wigcorrelate ucsc-wigtobigwig fastqc numpy macs2=2.1.0 bioconductor-seqlogo"
