---
name: histone-mark-correlation-analysis
description: Use when you have mESC (or analogous) ChIP-seq BAM files for chromatin
  marks plus ranked RNA-seq expression data and need to test whether mark signal correlates
  with gene expression groups (high/moderate/silent).
when_to_use_negative:
- The input is already a feature/coverage table or processed bigWig profiles — SeqCode
  expects BAM/SAM mapped reads plus chromInfo/refGene annotations, and bigwig generation
  is explicitly not supported.
- The analysis needs genome-browser visualization tracks or spike-in-corrected peak
  overlap statistics rather than signal-versus-expression-group comparison via boxplots.
- The ChIP-seq experiment includes spike-in material that requires manual spike-in
  normalization and your comparison design does not allow supplying that factor.
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3170
tools:
- name: SeqCode
  role: recoverChIPlevels computes avg/max/total normalized read counts over gene
    bodies of each expression group; R scripts bundled with SeqCode render the per-group
    boxplots
  repo: https://github.com/eblancoga/seqcode
- name: Bowtie
  role: Maps ChIP-seq reads to the mouse mm9 genome
- name: SAMtools
  role: Excludes unaligned reads from the Bowtie BAM files before SeqCode processing
- name: MACS
  role: MACS12 performs peak calling on the ChIP-seq experiments
- name: R
  role: Internally invoked by SeqCode to generate the boxplot graphics
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-021-98889-7
    title: Productive visualization of high-throughput sequencing data using the SeqCode
      open portable platform
  source_content_hash: sha256:2e0af68de1493c103f8985652ba5cc17d6d3dd1e38758926091c723ba4572e2a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eblancoga/seqcode
  tools:
  - Bowtie
  - MACS
  - R
  - SAMtools
  - SeqCode
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# histone-mark-correlation-analysis

## Summary

Reproduces the correlation between normalized ChIP-seq signal of active (H3K4me3, H3K36me3, Ser5P, Ser2P) and repressive (H3K27me3, Suz12, Jarid2, Ring1b, H2Aub) chromatin marks and RNA-seq-defined expression classes of RefSeq genes, as demonstrated in the SeqCode article's Fig. 6 workflow using recoverChIPlevels and R-based boxplots.

## When to use

Use when you have mESC (or analogous) ChIP-seq BAM files for chromatin marks plus ranked RNA-seq expression data and need to test whether mark signal correlates with gene expression groups (high/moderate/silent).

## When NOT to use

- The input is already a feature/coverage table or processed bigWig profiles — SeqCode expects BAM/SAM mapped reads plus chromInfo/refGene annotations, and bigwig generation is explicitly not supported.
- The analysis needs genome-browser visualization tracks or spike-in-corrected peak overlap statistics rather than signal-versus-expression-group comparison via boxplots.
- The ChIP-seq experiment includes spike-in material that requires manual spike-in normalization and your comparison design does not allow supplying that factor.

## Inputs

- Raw mESC ChIP-seq FASTQ files for the nine chromatin marks (NCBI GEO accessions from Table 4)
- Published mESC RNA-seq data (GSM1562339, Jacinto et al.) for expression ranking
- Bowtie-aligned BAM files for each ChIP-seq experiment (unaligned reads excluded)
- mm9 chromInfo (chromosome sizes) file
- mm9 refGene RefSeq transcript annotation file
- Gene lists for the three expression groups (100 highly, 100 moderately, 100 silently expressed genes)

## Outputs

- Normalized ChIP-seq read count tables (avg/max/total) over gene bodies per expression group per mark (recoverChIPlevels output folder)
- Boxplots of per-group ChIP levels for each mark (SeqCode R-generated PDF figures, Fig. 6 style)

## How to apply

Retrieve raw ChIP-seq FASTQ files per Table 4 (H3K4me3 SRX367147, Jarid2 SRX336228, H3K36me3 GSM1019769, H3K27me3 GSM1019772, Suz12 GSM1019771, H2Aub GSM850471, Ring1b GSM1041372, Ser5P GSM850467, Ser2P GSM850470) and the mESC RNA-seq data (GSM1562339, Jacinto et al.). Rank RefSeq genes by expression and select three groups of 100 genes: highly, moderately, and silently expressed. Map reads to the mouse mm9 genome with Bowtie, exclude unaligned reads from the BAM files with SAMtools, and call peaks with MACS12. Run SeqCode's recoverChIPlevels with the mm9 chromInfo and refGene annotation files to compute avg/max/total normalized read counts (normalized by total reads in the BAM, i.e. sequencing depth) over the gene bodies of each expression group for every mark, then generate per-group boxplots with SeqCode's R-based plotting to obtain the Fig. 6-style figure.

## Related tools

- **SeqCode** (recoverChIPlevels computes avg/max/total normalized read counts over gene bodies of each expression group; R scripts bundled with SeqCode render the per-group boxplots) — https://github.com/eblancoga/seqcode
- **Bowtie** (Maps ChIP-seq reads to the mouse mm9 genome)
- **SAMtools** (Excludes unaligned reads from the Bowtie BAM files before SeqCode processing)
- **MACS** (MACS12 performs peak calling on the ChIP-seq experiments)
- **R** (Internally invoked by SeqCode to generate the boxplot graphics)

## Examples

```
bin/recoverChIPlevels mm9.chromInfo.txt mm9.refGene.txt H3K4me3_mESC.bam genes_high_100.txt genes_moderate_100.txt genes_silent_100.txt H3K4me3_levels
```

## Evaluation signals

- The boxplot figure qualitatively reproduces Fig. 6: active marks (H3K4me3, H3K36me3, Ser5P, Ser2P) high on highly expressed genes and decreasing with lower expression
- Polycomb/repressive marks (H3K27me3, Suz12, Jarid2, Ring1b, H2Aub) show the inverse pattern: highest on silent genes and lowest on highly expressed genes
- Each expression group contains exactly 100 RefSeq genes selected by ranking the RNA-seq data (GSM1562339)
- recoverChIPlevels values are normalized by total reads in the BAM file (sequencing depth), so counts are depth-adjusted
- SeqCode run terminates successfully with an output folder containing the count data, R scripts, and PDF plots (no numeric threshold is reported; concordance of correlation directions is the acceptance criterion)

## Limitations

- No numeric threshold is reported in the article, so validation rests on qualitative concordance of correlation directions with Fig. 6 rather than a statistical cutoff
- Generation of genome-wide profiles in bigwig format is not supported and deferred to future versions
- Spike-in normalization factors must be manually introduced by users when spike-in material was included in the ChIP-seq experiment
- Most commands require user-supplied chromInfo and refGene annotation files for the exact genome assembly (mm9 here) retrieved from UCSC

## Evidence

- [results] ChIP-seq signal levels across the three expression-based gene groups show that gene expression positively correlated with H3K4me3, H3K36me3, Ser5P, and Ser2P, and negatively correlated with H3K27me3: [evidence span withheld — anchor status: not_found]
- [methods] Bowtie maps reads to mm9: "Bowtie7 was used to map the reads over the genome of mouse (mm9 and mm10), human (hg38), and the fruit fly (dm3)."
- [methods] SeqCode performed the normalization of values calculated within any finite element (e.g. regions, bins, etc.) for a given sequencing experiment by sequencing depth: "SeqCode performed the normalization of values calculated within any finite element (e.g. regions, bins, etc.) for a given sequencing experiment by sequencing depth"
- [methods] Unaligned reads must be excluded from the resulting BAM files that will be processed by SeqCode commands.: "Unaligned reads must be excluded from the resulting BAM files that will be processed by SeqCode commands."
- [readme] SeqCode commands require chromInfo and RefSeq annotations: "Most programs require the user to provide two files for the particular genome that is being analyzed: the list of chromosomes (and their sizes) and the RefSeq gene transcript annotations."
