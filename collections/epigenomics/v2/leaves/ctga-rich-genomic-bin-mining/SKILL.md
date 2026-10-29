---
name: ctga-rich-genomic-bin-mining
description: Use when you need to extract CT/GA-rich regions from a reference genome
  (e.g., hg19) as in silico read-restriction sets for a bisulfite-free 4-letter alignment
  benchmark, or to test promoter enrichment of such regions against genomic background.
when_to_use_negative:
- Input is sequencing reads (FASTQ) to be aligned or methylation-called — use Msuite's
  3-letter or 4-letter alignment mode instead of genome-wide bin mining
- The reference genome or species is not hg19/mm9/mm10/hg38 and no RefSeq annotation
  has been downloaded from the UCSC genome browser, since the TSS/promoter definition
  depends on RefSeq annotation files
- The goal is per-read 4-letter alignment accuracy measurement rather than characterizing
  where CT/GA-rich regions occur genome-wide
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Msuite
  role: Provides the hg19/mm9/mm10/hg38 RefSeq annotation files and the repository
    scripts for mined CT/GA-rich regions used to define the benchmark restriction
    criterion
  repo: https://github.com/hellosunking/Msuite/
- name: R
  role: Used in the Msuite project for data analysis and visualization of the enrichment
    results
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.patter.2020.100127
    title: 'Msuite: A High-Performance and Versatile DNA Methylation Data-Analysis
      Toolkit'
  source_content_hash: sha256:91ef83ba7f557b0e01c062c83853891f689a3e407015d0f38d14791a5b13b2b1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/hellosunking/Msuite
  repo_url: https://github.com/hellosunking/Msuite
  tools:
  - Msuite
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# CT/GA-rich genomic bin mining and promoter enrichment testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

This skill mines the human hg19 genome for CT/GA-rich regions — 500-bp non-overlapping bins with CT proportion ≥80% or GA proportion ≥80% — the same restriction criterion used to build Msuite's in silico TAPS reads for the 4-letter alignment benchmark, and then tests whether these bins are enriched in promoters versus the genomic background.

## When to use

Use when you need to extract CT/GA-rich regions from a reference genome (e.g., hg19) as in silico read-restriction sets for a bisulfite-free 4-letter alignment benchmark, or to test promoter enrichment of such regions against genomic background.

## When NOT to use

- Input is sequencing reads (FASTQ) to be aligned or methylation-called — use Msuite's 3-letter or 4-letter alignment mode instead of genome-wide bin mining
- The reference genome or species is not hg19/mm9/mm10/hg38 and no RefSeq annotation has been downloaded from the UCSC genome browser, since the TSS/promoter definition depends on RefSeq annotation files
- The goal is per-read 4-letter alignment accuracy measurement rather than characterizing where CT/GA-rich regions occur genome-wide

## Inputs

- hg19 genome sequence as a single multi-fasta file (or a directory of per-chromosome sequences)
- RefSeq annotation file for hg19 (included with the Msuite package for mm9/mm10/hg19/hg38)
- Msuite package scripts for mined CT/GA-rich regions (github.com/hellosunking/Msuite)

## Outputs

- Set of 500-bp genomic bins with CT proportion >=80% or GA proportion >=80% (CT/GA-rich regions)
- Promoter window intervals (-2000 bp to +1000 bp around each RefSeq TSS)
- Plain-text enrichment table reporting promoter-overlap fraction of CT/GA-rich bins versus whole-genome background fraction

## How to apply

Parse the RefSeq annotation file for hg19 that ships inside the Msuite package (build.index files cover mm9/mm10/hg19/hg38) and extract each gene's transcription start site (TSS), then define a promoter window spanning 2,000 bp upstream to 1,000 bp downstream of each TSS. Scan the hg19 genome multi-fasta sequence in 500-bp non-overlapping bins and retain bins in which the CT proportion is >=80% or the GA proportion is >=80%, mirroring the criterion used to restrict the in silico TAPS reads in Msuite's benchmark design. Compute the fraction of CT/GA-rich bins falling inside promoter windows and the corresponding fraction across the whole genome as background, and write both numbers to a plain-text enrichment table. The expected result is a marked excess of promoter overlap (~4.5% of CT/GA-rich bins in promoters vs ~2.9% genomic background), with CT/GA-rich regions covering roughly 0.31% of the human genome — the rationale for benchmarking Msuite's 4-letter mode on CT/GA-restricted reads.

## Related tools

- **Msuite** (Provides the hg19/mm9/mm10/hg38 RefSeq annotation files and the repository scripts for mined CT/GA-rich regions used to define the benchmark restriction criterion) — https://github.com/hellosunking/Msuite/
- **R** (Used in the Msuite project for data analysis and visualization of the enrichment results)

## Evaluation signals

- CT/GA-rich regions cover approximately 0.31% of the human genome in total bin length
- The promoter-overlap fraction of CT/GA-rich bins is markedly higher than the whole-genome background fraction (reported ~4.5% vs ~2.9%)
- Every retained bin satisfies the criterion: CT proportion >=80% or GA proportion >=80% in a 500-bp non-overlapping window
- Promoter windows are exactly 2,000 bp upstream to 1,000 bp downstream of each RefSeq TSS, one window per extracted TSS
- The output enrichment table is a plain-text file containing both the promoter fraction and the genomic background fraction

## Limitations

- The CT/GA-rich restriction criterion is specific to the 4-letter TAPS/5hmC-CATCH benchmark design (low non-CpG methylation assays) and may not generalize to other protocols or species
- RefSeq TSS definitions are only directly available in the package for mm9/mm10/hg19/hg38; other genome builds require downloading annotations from the UCSC genome browser
- Genome indices and annotation must be prepared first via build.index.sh before any reference-genome analysis with Msuite

## Evidence
