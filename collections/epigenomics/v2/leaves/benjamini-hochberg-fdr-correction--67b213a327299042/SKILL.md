---
name: benjamini-hochberg-fdr-correction
description: Use when testing thousands of phased heterozygous SNPs for allelic imbalance
  in ChIA-PET read coverage and needing multiple-testing-corrected significance calls
  before classifying allele-specific loops.
when_to_use_negative:
- Testing a single pre-specified hypothesis where no multiple-testing correction is
  needed
- Input data lack phased heterozygous SNP annotations, so no per-SNP allelic binomial
  tests can be computed
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3169
tools:
- name: ChIA-PIPE
  role: 'Performs allele-specific analysis: haplotype assignment of reads, binomial
    testing of SNP coverage bias, and Benjamini-Hochberg FDR correction'
  repo: https://github.com/TheJacksonLaboratory/ChIA-PIPE
- name: CPU
  role: Provides the dedup module that produces the unique (MAPQ >= 30), nonredundant
    PET alignments feeding the SNP coverage counts
  repo: https://github.com/cheehongsg/CPU
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1126/sciadv.aay2078
    title: 'ChIA-PIPE: A fully automated pipeline for comprehensive ChIA-PET data
      analysis and visualization'
  source_content_hash: sha256:91dc34f66a834e187efae63b1c8325491237645eb2cef591c3c145c4010eaf0c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TheJacksonLaboratory/ChIA-PIPE
  tools:
  - CPU
  - ChIA-PIPE
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# benjamini-hochberg-fdr-correction

## Summary

Apply Benjamini-Hochberg false discovery rate correction to per-SNP binomial test P values when detecting allelic bias in ChIA-PET read coverage, retaining SNPs with FDR Q <= 0.1 as significantly biased for resolving allele-specific phased peaks and loops.

## When to use

Use when testing thousands of phased heterozygous SNPs for allelic imbalance in ChIA-PET read coverage and needing multiple-testing-corrected significance calls before classifying allele-specific loops.

## When NOT to use

- Testing a single pre-specified hypothesis where no multiple-testing correction is needed
- Input data lack phased heterozygous SNP annotations, so no per-SNP allelic binomial tests can be computed

## Inputs

- Phased heterozygous SNPs (NA12878 hg19 Broad bundle) overlapping called peak regions
- Haplotype-assigned unique ChIA-PET read alignments (MAPQ >= 30) per SNP
- Per-SNP binomial test P values for allelic coverage bias

## Outputs

- Benjamini-Hochberg FDR Q value per phased SNP
- Set of significantly biased SNPs at FDR Q <= 0.1
- Allele-specific phased loop classifications derived from significant SNPs (e.g., 636 CTCF phased loops)

## How to apply

Within the ChIA-PIPE allele-specific workflow, first assign each uniquely aligned read (MAPQ >= 30) a haplotype label by overlap with phased SNPs from the NA12878 hg19 Broad bundle. For each phased SNP within peak regions, compute a binomial test P value on the bias in ChIA-PET read coverage between the two haplotypes. Adjust all SNP-level P values with the Benjamini-Hochberg procedure and retain only SNPs with FDR Q <= 0.1 as having significantly biased protein factor binding coverage. These significant SNPs then serve as anchors for classifying loops as allele-specific phased interactions, as in the GM12878 CTCF analysis yielding 636 phased loops.

## Related tools

- **ChIA-PIPE** (Performs allele-specific analysis: haplotype assignment of reads, binomial testing of SNP coverage bias, and Benjamini-Hochberg FDR correction) — https://github.com/TheJacksonLaboratory/ChIA-PIPE
- **CPU** (Provides the dedup module that produces the unique (MAPQ >= 30), nonredundant PET alignments feeding the SNP coverage counts) — https://github.com/cheehongsg/CPU

## Examples

```
bash chia_pipe-master/0.chia_pipe_shell.sh -c chia_pipe-master/example_config_file.sh
```

## Evaluation signals

- Every phased SNP receives a finite FDR Q value and its adjusted value is monotone-consistent with its raw binomial P value rank
- Retained SNPs satisfy the Benjamini-Hochberg FDR Q <= 0.1 threshold exactly as specified
- Downstream phased loop count matches the reported expectation of 636 phased CTCF interaction loops for the GM12878 MiSeq test data
- Haplotype assignment covered all uniquely aligned reads overlapping phased SNPs (no unassigned reads in peak regions)
- Corrected P values are computed over the full family of tested SNPs within peak regions, not per-SNP or per-chromosome subsets

## Limitations

- Requires genome-wide SNP phasing information for the sample's genome (available for GM12878/NA12878 via the Broad hg19 bundle); samples without phasing cannot use this procedure
- No accession IDs are provided in the article for the CTCF and RNAPII ChIA-PET sequencing data used in the allele-specific analysis, complicating exact replication
- The article does not specify the reference genome version in the allele-specific section beyond the hg19 Broad bundle context

## Evidence
