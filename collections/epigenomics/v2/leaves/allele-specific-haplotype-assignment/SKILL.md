---
name: allele-specific-haplotype-assignment
description: Use when ChIA-PET (or similar chromatin interaction) data must be resolved
  into haplotype-specific peaks and loops for a genome whose SNP phasing information
  is available, and allelic bias in binding coverage needs statistical testing.
when_to_use_negative:
- The sample has no genome-wide SNP phasing information available (e.g., a cell line
  not covered by a phased reference bundle), so reads cannot be haplotype-labeled.
- The analysis goal is total (non-allele-specific) peak and loop calling — the standard
  ChIA-PIPE pipeline without the allele-specific workflow suffices.
- Reads are multi-mapping or redundant (MAPQ < 30, non-unique alignments), since haplotype
  assignment requires uniquely aligned reads.
edam_operation: http://edamontology.org/operation_3195
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3169
tools:
- name: ChIA-PIPE
  role: 'Automated pipeline performing the allele-specific workflow: haplotype assignment
    of uniquely aligned reads and binomial testing of SNP allelic bias within the
    ChIA-PET analysis'
  repo: https://github.com/TheJacksonLaboratory/ChIA-PIPE
- name: CPU (ChIA-PET Utilities)
  role: Modularized executables (e.g., dedup) producing the unique, nonredundant PET
    alignments (MAPQ ≥ 30) that are haplotype-labeled
  repo: https://github.com/cheehongsg/CPU
- name: BWA
  role: Maps flanking genomic tags (BWA-ALN and BWA-MEM hybrid) to generate the alignments
    that overlap phased SNPs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1126/sciadv.aay2078
    title: 'ChIA-PIPE: A fully automated pipeline for comprehensive ChIA-PET data
      analysis and visualization'
  source_content_hash: sha256:bbd18747f182b7b92a779144c3216269c65d45f5902eb5925d7960e9510004f2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TheJacksonLaboratory/ChIA-PIPE
  tools:
  - BWA
  - CPU (ChIA-PET Utilities)
  - ChIA-PIPE
  edam_operation: http://edamontology.org/operation_3195
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# allele-specific-haplotype-assignment

## Summary

Assign haplotype labels to uniquely aligned ChIA-PET reads by overlapping them with phased SNPs (e.g., NA12878 hg19 Broad bundle), then use a binomial test with Benjamini-Hochberg FDR correction (Q ≤ 0.1) to detect SNPs with significant allelic bias in protein factor binding coverage, enabling allele-specific peak and loop resolution in ChIA-PIPE.

## When to use

Use when ChIA-PET (or similar chromatin interaction) data must be resolved into haplotype-specific peaks and loops for a genome whose SNP phasing information is available, and allelic bias in binding coverage needs statistical testing.

## When NOT to use

- The sample has no genome-wide SNP phasing information available (e.g., a cell line not covered by a phased reference bundle), so reads cannot be haplotype-labeled.
- The analysis goal is total (non-allele-specific) peak and loop calling — the standard ChIA-PIPE pipeline without the allele-specific workflow suffices.
- Reads are multi-mapping or redundant (MAPQ < 30, non-unique alignments), since haplotype assignment requires uniquely aligned reads.

## Inputs

- Uniquely aligned ChIA-PET reads (MAPQ ≥ 30) from ChIA-PIPE dedup, e.g., CTCF or RNAPII libraries
- Genome-wide SNP phasing information for the sample, e.g., 1000 Genome Project NA12878 phasing bundle (hg19, Broad Institute ftp bundle)
- Peak regions from ChIA-PIPE peak calling
- Loop calls from ChIA-PIPE loop calling (PET count ≥ 3, peak-supported anchors)

## Outputs

- Haplotype-labeled (phased) ChIA-PET reads
- Table of heterozygous SNPs with significant allelic bias in ChIA-PET read coverage (BH FDR Q ≤ 0.1)
- Phased allele-specific peak calls
- Phased allele-specific interaction loop table (e.g., 636 phased CTCF loops for the MiSeq test dataset)

## How to apply

After ChIA-PIPE completes linker filtering, BWA mapping of flanking genomic tags, retention of unique alignments (MAPQ ≥ 30) and nonredundant PETs, and loop calling (PET count ≥ 3 with peak support at both anchors), assign each uniquely aligned read overlapping a phased SNP a haplotype label using the genome-wide SNP phasing bundle for the sample (e.g., NA12878 hg19 from the Broad Institute bundle). For each phased SNP within peak regions, test allele-specific binding coverage bias with a binomial test, then apply Benjamini-Hochberg multiple hypothesis correction and retain SNPs with FDR Q ≤ 0.1 as significantly biased. Finally, resolve loop anchors against these significantly biased SNPs to classify loops as allele-specific phased interactions, exporting a phased loop table (e.g., 636 phased CTCF interaction loops for the GM12878 MiSeq test data). The binomial test provides the statistical basis: reads covering a heterozygous SNP should split between haplotypes roughly equally unless binding is allele-biased.

## Related tools

- **ChIA-PIPE** (Automated pipeline performing the allele-specific workflow: haplotype assignment of uniquely aligned reads and binomial testing of SNP allelic bias within the ChIA-PET analysis) — https://github.com/TheJacksonLaboratory/ChIA-PIPE
- **CPU (ChIA-PET Utilities)** (Modularized executables (e.g., dedup) producing the unique, nonredundant PET alignments (MAPQ ≥ 30) that are haplotype-labeled) — https://github.com/cheehongsg/CPU
- **BWA** (Maps flanking genomic tags (BWA-ALN and BWA-MEM hybrid) to generate the alignments that overlap phased SNPs)

## Examples

```
bash chia_pipe-master/0.chia_pipe_shell.sh -c chia_pipe-master/example_config_file.sh
```

## Evaluation signals

- Exported phased CTCF loop table matches the reported count of 636 phased-interaction loops for the GM12878 MiSeq test dataset (LDK0004-ds)
- All retained biased SNPs have Benjamini-Hochberg FDR Q ≤ 0.1; any SNP with Q > 0.1 must be excluded
- Only reads with MAPQ ≥ 30 that overlap phased SNPs receive haplotype labels; unaligned or multi-mapping reads carry no haplotype assignment
- Every allele-specific loop has both anchors resolved against significantly biased SNPs and PET count ≥ 3 with peak support
- Haplotype coverage at heterozygous SNPs without binding bias approximates a 1:1 split, consistent with the binomial null

## Limitations

- The allele-specific analysis depends on external phasing data (NA12878 hg19 Broad bundle); it cannot be applied to samples lacking genome-wide SNP phasing information.
- Only uniquely aligned reads (MAPQ ≥ 30) receive haplotype assignments, so multi-mapping reads near SNPs are lost from allelic analysis.
- Loops are classified as allele-specific only when anchors resolve against SNPs passing FDR Q ≤ 0.1; regions without informative biased SNPs cannot be phased.

## Evidence

- [discussion] Binomial test with BH correction for allelic bias: "SNPs with Benjamini-Hochberg–adjusted P values [i.e., FDR (false discovery rate) Q values] ≤0.1 (32) were considered to have significantly biased protein factor binding coverage."
- [discussion] Haplotype assignment of uniquely aligned reads: "uniquely aligned (MAPQ ≥ 30) CTCF and RNAPII ChIA-PET reads were given a haplotype assignment depending on whether they overlapped a phased SNP"
- [discussion] Phasing data availability for GM12878/NA12878: "we used the GM12878 cell line, which was derived from the 1000 Genome Project human subject NA12878, whose genome-wide SNP phasing information is available"
- [abstract] Allele-specific resolution capability: "ChIA-PIPE calls chromatin contact domains, resolves allele-specific peaks and loops, and annotates enhancer-promoter loops"
- [readme] Test data staging for the phased-loop task: "mkdir -p fastq
cp  LDK0004-ds_*.fastq.gz  fastq"
