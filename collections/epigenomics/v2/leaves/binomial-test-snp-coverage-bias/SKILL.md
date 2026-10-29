---
name: binomial-test-snp-coverage-bias
description: Use when allele-specific (haplotype-resolved) binding or looping from
  ChIA-PET data is needed and phased SNP genotypes (e.g., NA12878 hg19 Broad bundle)
  are available to assign haplotype labels to uniquely aligned reads.
when_to_use_negative:
- No phased heterozygous SNP information is available for the sample (e.g., cell line
  without 1000 Genomes phasing like the NA12878 bundle)
- The goal is bulk (allele-agnostic) peak or loop calling rather than haplotype-resolved
  interaction analysis
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0625
tools:
- name: ChIA-PIPE
  role: 'Executes the allele-specific workflow: haplotype assignment of uniquely aligned
    reads, per-SNP binomial testing of coverage bias with BH FDR correction, and phased
    loop resolution'
  repo: https://github.com/TheJacksonLaboratory/ChIA-PIPE
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1126/sciadv.aay2078
    title: 'ChIA-PIPE: A fully automated pipeline for comprehensive ChIA-PET data
      analysis and visualization'
  source_content_hash: sha256:307cf4a9383975d51a00006df9c2eb7efb68451f08399fabfaca4028de5c1bd5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TheJacksonLaboratory/ChIA-PIPE
  tools:
  - ChIA-PIPE
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# binomial-test-snp-coverage-bias

## Summary

Detect heterozygous SNPs with significant allelic bias in ChIA-PET read coverage using a binomial test with Benjamini-Hochberg FDR correction (Q <= 0.1), providing the statistical basis for ChIA-PIPE to resolve allele-specific phased peaks and loops (e.g., the 636 phased CTCF interaction loops in GM12878).

## When to use

Use when allele-specific (haplotype-resolved) binding or looping from ChIA-PET data is needed and phased SNP genotypes (e.g., NA12878 hg19 Broad bundle) are available to assign haplotype labels to uniquely aligned reads.

## When NOT to use

- No phased heterozygous SNP information is available for the sample (e.g., cell line without 1000 Genomes phasing like the NA12878 bundle)
- The goal is bulk (allele-agnostic) peak or loop calling rather than haplotype-resolved interaction analysis

## Inputs

- ChIA-PET reads with haplotype assignments (uniquely aligned, MAPQ >= 30, overlapping phased SNPs)
- Phased SNP calls with genome-wide phasing information (e.g., 1000 Genomes NA12878 hg19 Broad Institute bundle)
- Peak regions called by ChIA-PIPE (CTCF or RNAPII ChIA-PET peaks)

## Outputs

- List of phased SNPs with binomial-test P values and Benjamini-Hochberg FDR Q values (significant bias at Q <= 0.1)
- Allele-specific phased peak calls
- Phased interaction loop table (e.g., 636 allele-specific phased CTCF loops)

## How to apply

In ChIA-PIPE's allele-specific workflow, first assign each uniquely aligned read (MAPQ >= 30) a haplotype label depending on whether it overlaps a phased SNP from the NA12878 hg19 Broad bundle. For each phased SNP within peak regions, compare ChIA-PET read coverage between the two alleles with a binomial test, treating equal allelic coverage as the null. Apply Benjamini-Hochberg multiple-testing correction and retain SNPs with FDR Q values <= 0.1 as having significantly biased protein factor binding coverage. Finally, resolve loop anchors against these significantly biased SNPs to classify loops as allele-specific phased interactions, then export the phased loop table (e.g., confirm 636 phased CTCF loops from GM12878 MiSeq data).

## Related tools

- **ChIA-PIPE** (Executes the allele-specific workflow: haplotype assignment of uniquely aligned reads, per-SNP binomial testing of coverage bias with BH FDR correction, and phased loop resolution) — https://github.com/TheJacksonLaboratory/ChIA-PIPE

## Examples

```
bash chia_pipe-master/0.chia_pipe_shell.sh -c chia_pipe-master/example_config_file.sh
```

## Evaluation signals

- Only SNPs within peak regions with Benjamini-Hochberg FDR Q <= 0.1 are retained as significantly biased
- Haplotype labels are assigned only to reads with MAPQ >= 30 overlapping phased SNPs
- The exported phased CTCF loop table count matches the reported 636 phased-interaction loops for the GM12878 MiSeq CTCF ChIA-PET test data (LDK0004-ds)
- Phased loops have peak support at both anchors (PET count >= 3) and anchors resolved against significantly biased SNPs

## Limitations

- Requires external genome-wide SNP phasing information for the sample (only available for e.g. NA12878/GM12878 via the Broad hg19 bundle)
- No accession IDs are provided for the CTCF and RNAPII ChIA-PET sequencing data used in the allele-specific analysis, complicating exact reproduction
- The reference genome version used for alignment is not specified in the discussion section (BWA-ALN/BWA-MEM hybrid mapping)

## Evidence
