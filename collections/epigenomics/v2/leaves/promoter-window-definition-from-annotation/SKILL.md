---
name: promoter-window-definition-from-annotation
description: Use when testing whether sequence- or feature-defined genomic bins (e.g.,
  CT/GA-rich regions from the hg19 multi-fasta) are enriched in promoters, and only
  a gene-annotation file (RefSeq) is available for locating TSSs.
when_to_use_negative:
- The annotation is not a RefSeq-style gene table or the genome assembly is not among
  mm9/mm10/hg19/hg38, since the Msuite package only includes RefSeq files for those
  versions and other versions must be fetched from the UCSC genome browser
- The candidate bins were defined for bisulfite (3-letter BS-seq) analysis rather
  than the bisulfite-free TAPS context, where the CT/GA >=80% bin criterion is meaningful
- Promoter coordinates are already provided as ready-made intervals (e.g., a promoter
  BED), making TSS re-extraction redundant
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0654
tools:
- name: Msuite
  role: Provides the bundled hg19 RefSeq annotation and the build.index.sh utility
    that consumes the genome FASTA plus RefSeq annotation, anchoring the TSS-based
    coordinate system used for promoter windows
  repo: https://github.com/hellosunking/Msuite/
- name: R
  role: Used for post-processing and tabulation of the promoter-overlap versus background
    enrichment fractions
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.patter.2020.100127
    title: 'Msuite: A High-Performance and Versatile DNA Methylation Data-Analysis
      Toolkit'
  source_content_hash: sha256:8fa11759964bd8e408a7d9ee953465ffcfb2876ad29a02c74392b49fca5b78f2
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# promoter-window-definition-from-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Derive promoter intervals by extracting transcription start sites (TSSs) from a RefSeq annotation file and spanning a fixed window (2,000 bp upstream to 1,000 bp downstream) around each TSS, as done in the Msuite hg19 workflow to test whether CT/GA-rich 500-bp bins are enriched near promoters relative to the ~2.9% whole-genome background.

## When to use

Use when testing whether sequence- or feature-defined genomic bins (e.g., CT/GA-rich regions from the hg19 multi-fasta) are enriched in promoters, and only a gene-annotation file (RefSeq) is available for locating TSSs.

## When NOT to use

- The annotation is not a RefSeq-style gene table or the genome assembly is not among mm9/mm10/hg19/hg38, since the Msuite package only includes RefSeq files for those versions and other versions must be fetched from the UCSC genome browser
- The candidate bins were defined for bisulfite (3-letter BS-seq) analysis rather than the bisulfite-free TAPS context, where the CT/GA >=80% bin criterion is meaningful
- Promoter coordinates are already provided as ready-made intervals (e.g., a promoter BED), making TSS re-extraction redundant

## Inputs

- RefSeq annotation file for hg19 (REFSEQ.txt; .gz/.bz2 accepted, as bundled in the Msuite package)
- hg19 genome sequence as a single multi-fasta file or a directory of per-chromosome FASTA files
- Optional: BED/interval list of candidate regions (e.g., CT/GA-rich 500-bp bins with >=80% CT or GA proportion)

## Outputs

- Promoter window interval list (TSS ± [−2,000 bp, +1,000 bp])
- Plain-text enrichment table reporting promoter-overlap fraction of CT/GA-rich bins (~4.5%) versus whole-genome background (~2.9%)

## How to apply

Parse the RefSeq annotation file for the target assembly (Msuite ships hg19 files; gzipped REFSEQ.txt.gz is accepted) and extract the TSS coordinate per gene, using strand to identify the transcript start. For every TSS, define the promoter window as 2,000 bp upstream to 1,000 bp downstream, then intersect it with candidate regions — here, 500-bp non-overlapping bins of the hg19 multi-fasta retained when CT or GA proportion >= 80% (the same criterion used to restrict in silico TAPS reads in Msuite's 4-letter benchmark). Compute the fraction of retained bins overlapping any promoter window and compare with the fraction of all bins overlapping promoters (genomic background). The rationale is that the Msuite paper itself extracted TSSs from RefSeq genes for TSS-centered analysis, so this window definition reproduces their promoter coordinate system. Write both fractions to a plain-text enrichment table so the enrichment ratio is directly auditable.

## Related tools

- **Msuite** (Provides the bundled hg19 RefSeq annotation and the build.index.sh utility that consumes the genome FASTA plus RefSeq annotation, anchoring the TSS-based coordinate system used for promoter windows) — https://github.com/hellosunking/Msuite/
- **R** (Used for post-processing and tabulation of the promoter-overlap versus background enrichment fractions)

## Examples

```
build.index/build.index.sh hg19.fa REFSEQ.txt.gz hg19
```

## Evaluation signals

- The promoter-overlap fraction of CT/GA-rich bins (~4.5%) must exceed the whole-genome background fraction (~2.9%), reproducing the reported enrichment
- CT/GA-rich bins should cover ~0.31% of the human genome, sanity-checking the >=80% CT/GA bin filter
- Every promoter window center must coincide with a RefSeq TSS coordinate, and strand must orient the −2,000/+1,000 bp offsets correctly
- The enrichment table should contain exactly two comparable fractions computed on the same 500-bp binning grid

## Limitations

- Msuite only ships RefSeq annotations for mm9/mm10/hg19/hg38; other assemblies require downloading annotations from the UCSC genome browser
- The 4-letter/CT-GA analysis context applies only to TAPS/5hmC-CATCH data where non-CpG methylation is very low, not BS-seq data
- A fixed ±window definition ignores gene-specific promoter architecture; overlapping gene TSSs may double-count promoter regions

## Evidence

- [other] CT/GA-rich region enrichment finding motivating the window analysis: [evidence span withheld — anchor status: not_found]
