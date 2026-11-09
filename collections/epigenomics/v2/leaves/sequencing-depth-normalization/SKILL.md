---
name: sequencing-depth-normalization
description: Use when computing normalized ChIP-seq signal (buildChIPprofile, recoverChIPlevels,
  meta-plots, boxplots) from BAM files of samples sequenced at different depths, or
  when down-sampling to test depth requirements.
when_to_use_negative:
- The experiment includes spike-in material (e.g. MTF2 samples) — use the spike-in
  genome read counts as the normalization factor instead of total BAM depth
- Input data is already a normalized profile (e.g. bigWig/BedGraph) rather than a
  raw mapped BAM, so depth normalization would double-normalize the signal
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: SeqCode
  role: Performs the depth-based normalization of values in bins/regions and applies
    it in buildChIPprofile and recoverChIPlevels; spike-in factor can be manually
    introduced
  repo: https://github.com/eblancoga/seqcode
- name: SAMtools
  role: Down-samples BAM files to defined depths (e.g. 1,000,000 and 8,000,000 reads)
    prior to normalization; integrated internally to read BAM/SAM
- name: Bowtie
  role: Maps reads to the reference genome so that total mapped read counts are available
    as the normalization denominator
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_008
  source_papers:
  - doi: 10.1038/s41598-021-98889-7
    title: Productive visualization of high-throughput sequencing data using the SeqCode
      open portable platform
  source_content_hash: sha256:6ee58a9a4f2bfd406a3b0afc36ac9d664b4207bd84d9550d24d4e902558963b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eblancoga/seqcode
  tools:
  - Bowtie
  - R
  - SAMtools
  - SeqCode
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# sequencing-depth-normalization

## Summary

Normalizes ChIP-seq/NGS signal values by sequencing depth (total mapped reads in the BAM) before generating SeqCode profiles, region-level counts, and plots, ensuring comparability across samples sequenced at different depths; spike-in read counts over an exogenous genome can substitute as the normalization factor.

## When to use

Use when computing normalized ChIP-seq signal (buildChIPprofile, recoverChIPlevels, meta-plots, boxplots) from BAM files of samples sequenced at different depths, or when down-sampling to test depth requirements.

## When NOT to use

- The experiment includes spike-in material (e.g. MTF2 samples) — use the spike-in genome read counts as the normalization factor instead of total BAM depth
- Input data is already a normalized profile (e.g. bigWig/BedGraph) rather than a raw mapped BAM, so depth normalization would double-normalize the signal

## Inputs

- BAM/SAM files of mapped ChIP-seq reads with unaligned reads excluded (SeqCode internally integrates SAMtools/HTSlib to read them)
- mm9 chromInfo.txt chromosome-size file (or equivalent for the target genome assembly)
- RefSeq refGene.txt annotation file from UCSC
- Optionally, spike-in genome mapped read counts to compute a manual spike-in factor

## Outputs

- Depth-normalized compressed BedGraph genome-wide profiles from buildChIPprofile
- avg/max/total normalized read counts per gene or region from recoverChIPlevels
- Boxplots and meta-plots (R-generated PDFs) whose y-axes are comparable across samples of different depths

## How to apply

First map reads (Bowtie over mm9/mm10/hg38/dm3), exclude unaligned reads from the resulting BAM files, and record the total number of mapped reads per sample. SeqCode internally normalizes every value calculated within a finite element (regions, bins, gene bodies) by sequencing depth — the total number of reads in the BAM file — so the 1M- and 8M-read down-sampled BAMs (e.g. SAMtools view -s on H3K4me3 SRX367147 and H3K36me3 GSM1019769) yield directly comparable BedGraph profiles from buildChIPprofile (default 100 bp windows, 150 bp fragment, single-end). When recoverChIPlevels computes avg/max/total counts over gene bodies or peak sets, the same depth-based normalization applies, which is what lets expression-class boxplots (highly/moderately/silent) be compared across the nine ChIP-seq marks. Decision point: if a spike-in control was included (e.g. MTF2 samples in Fig. 7), do not use BAM depth — instead manually introduce the spike-in factor, computed as reads mapped over the spike-in genome, into the SeqCode applications. Verify the depth normalization worked by confirming that within a single mark, the 1M and 8M profiles show the same signal scale rather than an 8-fold offset.

## Related tools

- **SeqCode** (Performs the depth-based normalization of values in bins/regions and applies it in buildChIPprofile and recoverChIPlevels; spike-in factor can be manually introduced) — https://github.com/eblancoga/seqcode
- **SAMtools** (Down-samples BAM files to defined depths (e.g. 1,000,000 and 8,000,000 reads) prior to normalization; integrated internally to read BAM/SAM)
- **Bowtie** (Maps reads to the reference genome so that total mapped read counts are available as the normalization denominator)

## Examples

```
samtools view -s 0.125 H3K4me3_full.bam > H3K4me3_1M.bam; bin/buildChIPprofile -v chromInfo_mm9.txt H3K4me3_1M.bam H3K4me3_1M
```

## Evaluation signals

- Down-sampled profiles of the same mark (e.g. 1M vs 8M reads of H3K4me3) show the same signal scale after SeqCode's depth normalization, not an ~8-fold offset
- Boxplot y-axis values across expression classes are in normalized units, allowing qualitative comparison with Fig. 6 (active marks high on highly expressed genes)
- 1 million reads resolve the sharp H3K4me3 promoter pattern while 8 million are needed for broad H3K36me3 domains, matching Supplementary Figure 5
- Unaligned reads have been excluded from the BAM before normalization (SeqCode rejects/ignores unaligned BAM content)
- For spike-in experiments, the normalization factor corresponds to spike-in genome mapped reads, not total BAM read count

## Limitations

- Spike-in normalization factors must be manually computed and introduced by the user; SeqCode does not compute them automatically
- Depth-based normalization cannot compensate for broad-background differences — broad H3K36me3 domains are indistinguishable at 1M reads even after normalization
- Generation of genome-wide profiles in bigwig format is not supported and deferred to future versions

## Evidence

- [methods] SeqCode performed the normalization of values calculated within any finite element (e.g. regions, bins, etc.) for a given sequencing experiment by sequencing depth: "SeqCode performed the normalization of values calculated within any finite element (e.g. regions, bins, etc.) for a given sequencing experiment by sequencing depth"
- [methods] When spike-in material was included into the ChIP-seq experiment (e.g. MTF2 samples in Fig. 7), the number of reads mapped over the spike-in genome was instead used to normalize the data.: "When spike-in material was included into the ChIP-seq experiment (e.g. MTF2 samples in Fig. 7), the number of reads mapped over the spike-in genome was instead used to normalize the data."
- [discussion] we allow users of SeqCode to correct the normalization of sequencing samples by introducing the spike-in factor in our applications: "we allow users of SeqCode to correct the normalization of sequencing samples by introducing the spike-in factor in our applications"
