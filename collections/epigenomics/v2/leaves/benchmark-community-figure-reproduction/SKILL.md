---
name: benchmark-community-figure-reproduction
description: Use when per-species mapped read counts from BWA-mem-mapped RIMS-seq,
  DNA-seq, and bisulfite-seq libraries are available and the goal is to test whether
  a sequencing protocol biases species relative abundances in a defined synthetic
  community.
when_to_use_negative:
- Input is raw FASTQ reads rather than mapped per-species counts — map with BWA-mem
  first (e.g. mapping.pl) before abundance normalization
- The goal is methylase specificity/motif discovery (the RIMS-seq.pl / MosDi pipeline)
  rather than comparing species abundance bias across protocols
- The library preparation is tagmentation-based; RIMS-seq only works with adapter-ligation
  libraries, so abundance comparisons with RIMS-seq libraries are invalid
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: mapping.pl
  role: Maps sequencing reads to reference genomes using BWA-mem to produce the per-species
    mapped read counts needed for abundance normalization
  repo: https://github.com/Ettwiller/RIMS-seq/
- name: BWA
  role: Underlying aligner (BWA-mem) used by mapping.pl to map RIMS-seq, DNA-seq and
    bisulfite-seq reads to species reference genomes
  repo: https://github.com/lh3/bwa
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkab705
    title: Rapid identification of methylase specificity (RIMS-seq) jointly identifies
      methylated motifs and generates shotgun sequencing of bacterial genomes
  source_content_hash: sha256:71a8ff691c27ed1c1d4e56fb009e990ec0b313e376a82a1631d9fc24793bfd66
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ettwiller/RIMS-seq
  tools:
  - BWA
  - mapping.pl
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# benchmark-community-figure-reproduction

## Summary

Reproduces the Figure 5A benchmark comparing normalized species relative abundances across RIMS-seq, DNA-seq(+3H), and bisulfite-seq libraries from ATCC MSA-1005/1006 synthetic gut and skin microbiome communities, quantifying bisulfite bias on AT-rich species.

## When to use

Use when per-species mapped read counts from BWA-mem-mapped RIMS-seq, DNA-seq, and bisulfite-seq libraries are available and the goal is to test whether a sequencing protocol biases species relative abundances in a defined synthetic community.

## When NOT to use

- Input is raw FASTQ reads rather than mapped per-species counts — map with BWA-mem first (e.g. mapping.pl) before abundance normalization
- The goal is methylase specificity/motif discovery (the RIMS-seq.pl / MosDi pipeline) rather than comparing species abundance bias across protocols
- The library preparation is tagmentation-based; RIMS-seq only works with adapter-ligation libraries, so abundance comparisons with RIMS-seq libraries are invalid

## Inputs

- Per-species mapped read count table for ATCC MSA-1005/1006 synthetic gut and skin microbiome libraries (RIMS-seq, DNA-seq, bisulfite-seq)
- BAM files or mapped read counts produced by BWA-mem against the reference genomes of community members
- Per-library total mapped read counts for normalization

## Outputs

- abundance_comparison.csv containing normalized per-species relative abundances and the RIMS-seq/DNA-seq and bisulfite-seq/DNA-seq ratio columns
- Reproduced Figure 5A-style abundance comparison (RIMS-seq vs DNA-seq vs bisulfite-seq)

## How to apply

Load the per-species mapped read count table for the ATCC MSA-1005/1006 synthetic communities sequenced with RIMS-seq, DNA-seq(+3H), and bisulfite-seq, all mapped to reference genomes with BWA-mem (e.g. via the mapping.pl script from the RIMS-seq repository, which requires adapters to be removed prior to mapping). Using pandas, normalize each species' mapped read count by the total mapped reads in the same library to obtain relative abundances. Compute two ratio columns per species: RIMS-seq/DNA-seq and bisulfite-seq/DNA-seq abundance ratios. Judge success against the figure's acceptance criteria: all RIMS-seq/DNA-seq ratios fall within 0.8-1.2, while bisulfite-seq overestimates the AT-rich species Clostridioides difficile (~2.65x) and Staphylococcus epidermidis (~3.9x). Write the normalized abundances plus both ratio columns to abundance_comparison.csv.

## Related tools

- **mapping.pl** (Maps sequencing reads to reference genomes using BWA-mem to produce the per-species mapped read counts needed for abundance normalization) — https://github.com/Ettwiller/RIMS-seq/
- **BWA** (Underlying aligner (BWA-mem) used by mapping.pl to map RIMS-seq, DNA-seq and bisulfite-seq reads to species reference genomes) — https://github.com/lh3/bwa

## Examples

```
import pandas as pd; df = pd.read_csv('mapped_counts.csv'); ab = df.set_index('species').div(df.set_index('species').sum()); ab.to_csv('abundance_comparison.csv')
```

## Evaluation signals

- All RIMS-seq/DNA-seq abundance ratios fall within the 0.8-1.2 interval for every species in the community
- bisulfite-seq/DNA-seq ratio for Clostridioides difficile is approximately 2.65
- bisulfite-seq/DNA-seq ratio for Staphylococcus epidermidis is approximately 3.9
- Relative abundances are computed within-library (each species count divided by the library's total mapped reads), so each library's abundances sum to 1
- abundance_comparison.csv contains, per species, the normalized abundances plus both ratio columns

## Limitations

- Bisulfite sequencing itself biases abundance estimation of AT-rich species (over-estimating C. difficile and S. epidermidis), so bisulfite-seq should not be used for abundance profiling
- In natural microbial communities, uneven species representation can cause RIMS-seq to identify motifs only in the most abundant species, and SNPs may increase background noise for C-to-T deamination detection
- RIMS-seq library prep must be adapter-ligation based; the method does not work with tagmentation-based protocols

## Evidence

- [results] Species abundances are similar between DNA-seq(+3H) and RIMS-seq (abundance ratios between 0.8 and 1.2): [evidence span withheld — anchor status: not_found]
- [results] Bisulfite overestimates AT-rich species: [evidence span withheld — anchor status: not_found]
- [methods] Mapping with BWA-mem: "This script uses BWA-mem to map the reads to a reference genome."
- [readme] Library prep requirement: "RIMS-seq does not work if the library preparation protocol is based on tagmentation or other technologies"
