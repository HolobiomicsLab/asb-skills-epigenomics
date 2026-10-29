---
name: chromatin-loop-anchor-resolution
description: Use when ChIA-PET reads must be resolved to allele-specific loops for
  a sample with genome-wide SNP phasing (e.g., NA12878), and you need statistically
  biased SNPs at loop anchors.
when_to_use_negative:
- Input data lacks genome-wide SNP phasing information for the sample — allele-specific
  resolution cannot be performed
- Data type is not chromatin-mapping data (ChIA-PET, HiChIP, or PLAC-seq); the loop-calling
  and anchor-resolution logic is specific to these PET-based assays
- The goal is bulk loop calling without allele specificity — plain loop calling (PET
  count ≥ 3 with peak support) suffices and the SNP bias testing is unnecessary
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ChIA-PIPE
  role: Automated pipeline executing linker filtering, BWA mapping, dedup (MAPQ ≥
    30), peak calling, loop calling, and haplotype assignment for allele-specific
    peaks and loops
  repo: https://github.com/TheJacksonLaboratory/ChIA-PIPE
- name: CPU
  role: ChIA-PET Utilities modules (e.g., dedup) invoked by ChIA-PIPE to retain unique
    alignments and nonredundant PETs used for loop detection
  repo: https://github.com/cheehongsg/CPU
- name: BWA
  role: Hybrid BWA-ALN/BWA-MEM aligner used to map flanking genomic tags of PET reads
    to the reference genome before haplotype assignment
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1126/sciadv.aay2078
    title: 'ChIA-PIPE: A fully automated pipeline for comprehensive ChIA-PET data
      analysis and visualization'
  source_content_hash: sha256:9b777dd5b64ec0eb3adc4fe25c1b4896fe4fc2c0b6daf20720456612d2e784c0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TheJacksonLaboratory/ChIA-PIPE
  tools:
  - BWA
  - CPU
  - ChIA-PIPE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# chromatin-loop-anchor-resolution

## Summary

Resolves haplotype-specific (phased) chromatin interaction loops from ChIA-PET data by assigning haplotype labels to uniquely aligned reads overlapping phased SNPs, testing allelic coverage bias per SNP, and classifying loops as allele-specific phased interactions. This is how ChIA-PIPE derived the 636 phased CTCF loops from GM12878 (NA12878) data.

## When to use

Use when ChIA-PET reads must be resolved to allele-specific loops for a sample with genome-wide SNP phasing (e.g., NA12878), and you need statistically biased SNPs at loop anchors.

## When NOT to use

- Input data lacks genome-wide SNP phasing information for the sample — allele-specific resolution cannot be performed
- Data type is not chromatin-mapping data (ChIA-PET, HiChIP, or PLAC-seq); the loop-calling and anchor-resolution logic is specific to these PET-based assays
- The goal is bulk loop calling without allele specificity — plain loop calling (PET count ≥ 3 with peak support) suffices and the SNP bias testing is unnecessary

## Inputs

- ChIA-PET paired-end FASTQ reads (e.g., LDK0004-ds_*.fastq.gz CTCF MiSeq data)
- Phased SNP bundle (NA12878 hg19 Broad Institute bundle from ftp://gsapubftp-anonymous@ftp.broadinstitute.org/bundle/hg19/)
- Reference genome for BWA alignment
- ChIA-PIPE configuration file (e.g., example_config_file.sh)

## Outputs

- Phased (allele-specific) CTCF interaction loop table (reported: 636 phased-interaction loops)
- Per-SNP allelic bias statistics (binomial-test P values with Benjamini-Hochberg FDR Q values)
- Haplotype-labeled uniquely aligned ChIA-PET reads

## How to apply

Run the ChIA-PIPE pipeline (linker filtering, BWA mapping of flanking genomic tags, dedup retaining unique MAPQ ≥ 30 alignments and nonredundant PETs, peak calling, and loop calling with PET count ≥ 3 and peak support at both anchors). Assign each uniquely aligned read overlapping a phased SNP from the NA12878 hg19 Broad bundle a haplotype label. For each phased SNP within peak regions, test allele-specific binding coverage bias with a binomial test and retain SNPs with Benjamini-Hochberg FDR Q ≤ 0.1 as significantly biased. Finally, resolve loop anchors against these significantly biased SNPs to classify loops as allele-specific phased interactions and export the phased loop table (expected: 636 phased-interaction loops for the CTCF test case).

## Related tools

- **ChIA-PIPE** (Automated pipeline executing linker filtering, BWA mapping, dedup (MAPQ ≥ 30), peak calling, loop calling, and haplotype assignment for allele-specific peaks and loops) — https://github.com/TheJacksonLaboratory/ChIA-PIPE
- **CPU** (ChIA-PET Utilities modules (e.g., dedup) invoked by ChIA-PIPE to retain unique alignments and nonredundant PETs used for loop detection) — https://github.com/cheehongsg/CPU
- **BWA** (Hybrid BWA-ALN/BWA-MEM aligner used to map flanking genomic tags of PET reads to the reference genome before haplotype assignment)

## Examples

```
bash chia_pipe-master/0.chia_pipe_shell.sh -c chia_pipe-master/example_config_file.sh
```

## Evaluation signals

- Phased loop table count matches the reported 636 phased-interaction loops for the CTCF GM12878 MiSeq test data
- All retained SNP bias calls have Benjamini-Hochberg FDR Q ≤ 0.1
- Haplotype labels are assigned only to uniquely aligned reads (MAPQ ≥ 30) overlapping phased SNPs
- Called loops satisfy both filters: PET count ≥ 3 and peak support at both anchors
- Pipeline stages complete in order (linker filtering → BWA mapping → dedup → peak calling → loop calling) without QC failures reported by the automated quality assessment

## Limitations

- Allele-specific resolution requires genome-wide SNP phasing information for the sample; it is only demonstrated for GM12878/NA12878, which was derived from a 1000 Genomes Project subject
- No accession IDs are provided for the CTCF and RNAPII ChIA-PET sequencing data used in the allele-specific analysis, complicating exact reproduction
- Loop classification is restricted to PETs with linker detected and both ends having genomic tags; other read-pair categories are excluded from loop detection

## Evidence
