---
name: ct-ga-rich-region-sequence-selection
description: Use when benchmarking DNA methylation aligners on reads from CT/GA-rich
  regions, or when stratifying human genome bins by CT/GA proportion (≥70%/≥80%) to
  compare 4-letter vs 3-letter mapping efficiency and accuracy.
when_to_use_negative:
- Data generated with a BS-seq protocol — the README states you MUST use the 3-letter
  mode with '-m BS'; 4-letter mode ONLY supports TAPS/5hmC-CATCH data where non-CpG
  methylation is very low
- Samples with high non-CpG methylation (e.g., not most human somatic tissues), where
  the 4-letter mode's assumption does not hold
- The goal is methylation calling or visualization on real data rather than benchmarking
  aligner mapping performance on CT/GA-enriched regions
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Msuite
  role: Aligns TAPS reads in 4-letter ('-4 -m TAPS') and 3-letter ('-3 -m TAPS') modes;
    indices built with build.index/build.index.sh; results finalized by running 'make'
    in each output directory
  repo: https://github.com/hellosunking/Msuite/
- name: SHERMAN script
  role: Generates the in silico paired-end 36-bp benchmark reads from CT/GA-rich 500-bp
    bins under the TAPS conversion model
  repo: https://github.com/hellosunking/Msuite/
- name: Bismark
  role: 3-letter comparator aligner (bowtie2-based cytosine-to-thymine conversion)
    run under matched settings on the same read sets
- name: BWA-meth
  role: 3-letter comparator aligner (bowtie2/BWA-based conversion alignment) run under
    matched settings on the same read sets
- name: bowtie2
  role: Underline aligner used by both Msuite and Bismark for the conversion-based
    mapping step
  repo: https://github.com/BenLangmead/bowtie2
- name: samtools
  role: Converts SAM alignment output into BAM format
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.patter.2020.100127
    title: 'Msuite: A High-Performance and Versatile DNA Methylation Data-Analysis
      Toolkit'
  source_content_hash: sha256:b128c8959454fad233b5a29dc8401d1f3915f9366a1d1f12ab0a4b62fc260e87
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
  - BWA-meth
  - Bismark
  - Msuite
  - SHERMAN script
  - bowtie2
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# ct-ga-rich-region-sequence-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill that stratifies human-genome 500-bp bins by CT/GA proportion (≥70% and ≥80%) and uses in silico TAPS-converted paired-end 36-bp reads from those bins to demonstrate that Msuite's unique 4-letter alignment mode outperforms 3-letter mode (Msuite, Bismark, BWA-meth) in mapping efficiency and accuracy.

## When to use

Use when benchmarking DNA methylation aligners on reads from CT/GA-rich regions, or when stratifying human genome bins by CT/GA proportion (≥70%/≥80%) to compare 4-letter vs 3-letter mapping efficiency and accuracy.

## When NOT to use

- Data generated with a BS-seq protocol — the README states you MUST use the 3-letter mode with '-m BS'; 4-letter mode ONLY supports TAPS/5hmC-CATCH data where non-CpG methylation is very low
- Samples with high non-CpG methylation (e.g., not most human somatic tissues), where the 4-letter mode's assumption does not hold
- The goal is methylation calling or visualization on real data rather than benchmarking aligner mapping performance on CT/GA-enriched regions

## Inputs

- Human reference genome FASTA (e.g., hg19/hg38) plus RefSeq annotation for index building
- SHERMAN-simulated paired-end 36-bp FASTQ reads from human genomic bins with CT/GA proportion ≥70% and ≥80% (TAPS protocol: 20% CpG / 0.5% CpH conversion, 0.1% error rate)
- Msuite genome indices (4-letter and 3-letter) and matching Bismark and BWA-meth indices

## Outputs

- Per-aligner mapping efficiency and accuracy stratified by CT/GA region class (e.g., 4-letter ME 79.67% / accuracy 99.78% vs 3-letter 67.14% / 99.03%, Bismark 68.39% / 98.94%, BWA-meth 76.31% / 92.06% for ≥70% bins)
- table2_ct_ga_mapping_efficiency.csv with the CT/GA rows of Table 2
- Aligned SAM/BAM files per aligner per region class (SAM converted to BAM with samtools)

## How to apply

Retrieve the benchmark-generation scripts from the Msuite GitHub deposit and simulate 1,000,000 paired-end 36-bp reads with the SHERMAN script following the TAPS protocol (C->T conversion 20% at CpG sites, 0.5% at CpH sites, 0.1% error rate), sampling 500-bp human-genome bins stratified by CT/GA proportion ≥70% and ≥80%. Build Msuite indices with build.index/build.index.sh (e.g., hg19/hg38 with RefSeq annotation) plus matching Bismark and BWA-meth indices. Align each read set with Msuite in 4-letter mode ('-4 -m TAPS') and 3-letter mode ('-3 -m TAPS'), running 'make' in each output directory, then align the same reads with Bismark and BWA-meth, both of which perform 3-letter bowtie2-based conversion alignment, under matched settings. Compute mapping efficiency and accuracy per aligner per CT/GA region class, keeping only reads with unique best hits (multi-mapping reads with equal best scores are discarded), and assemble the CT/GA rows of Table 2.

## Related tools

- **Msuite** (Aligns TAPS reads in 4-letter ('-4 -m TAPS') and 3-letter ('-3 -m TAPS') modes; indices built with build.index/build.index.sh; results finalized by running 'make' in each output directory) — https://github.com/hellosunking/Msuite/
- **SHERMAN script** (Generates the in silico paired-end 36-bp benchmark reads from CT/GA-rich 500-bp bins under the TAPS conversion model) — https://github.com/hellosunking/Msuite/
- **Bismark** (3-letter comparator aligner (bowtie2-based cytosine-to-thymine conversion) run under matched settings on the same read sets)
- **BWA-meth** (3-letter comparator aligner (bowtie2/BWA-based conversion alignment) run under matched settings on the same read sets)
- **bowtie2** (Underline aligner used by both Msuite and Bismark for the conversion-based mapping step) — https://github.com/BenLangmead/bowtie2
- **samtools** (Converts SAM alignment output into BAM format)

## Examples

```
msuite -4 -m TAPS -x hg19 -1 reads_R1.fq -2 reads_R2.fq -o out.4letter && cd out.4letter && make
```

## Evaluation signals

- Reproduced mapping efficiency for ≥70% CT/GA bins matches the reported benchmark: Msuite 4-letter 79.67% vs 3-letter 67.14%, Bismark 68.39%, BWA-meth 76.31%
- Reproduced accuracy values match: 4-letter 99.78%, 3-letter 99.03%, Bismark 98.94%, BWA-meth 92.06% (BWA-meth markedly less accurate on these regions)
- The 4-letter over 3-letter advantage increases for bins with CT/GA ≥80% relative to ≥70%
- Mapping-efficiency counts include only reads with unique best hits; multi-mapping reads with equal best scores are discarded consistently across all four aligners
- Output table2_ct_ga_mapping_efficiency.csv contains rows for every aligner × region-class combination with matching column schema to Table 2

## Limitations

- No strong evidence that the 4-letter mode is more accurate on inconsistent CpG sites; the authors call for further validation
- Accuracy of the 4- and 3-letter modes and the limitations of current sequencing-based protocols should be validated with additional methods such as microarrays
- The real TAPS dataset had imperfect chemical treatment, so benchmark results from in silico reads may not transfer exactly
- 4-letter mode is only appropriate for TAPS/5hmC-CATCH data with very low non-CpG methylation; BS-seq data must use 3-letter mode with '-m BS'

## Evidence

- [intro] most of the current mainstream data-analysis tools only support 3-letter alignment: [evidence span withheld — anchor status: not_found]
