---
name: bam-downsampling-normalization
description: Use when comparing two spike-in ChIP-seq conditions (e.g. H3K79me2 25:75
  vs 75:25 mixed Jurkat cells) with unequal fly spike-in read totals, and you need
  to equalize spike-in depth by downsampling the human BAM, or you need a baseline
  uniform-scaling normalization to benchmark against spikChIP.
when_to_use_negative:
- When the goal is to minimize overcorrection of non-occupied/background regions —
  Tag removal applies the spike-in correction uniformly and overcorrects background
  bins; spikChIP's local regression, which applies the correction gradually from background
  to enriched regions, is the article's remedy.
- When spike-in sequencing depth is below the recommended ~1 million reads (2%–5%
  of the experimental genome) — the article recommends the -d option only for processing
  BAMs < 1 Million reads and warns shallow spike-in undermines normalization.
- When the experiments lack an exogenous spike-in control (no fly/DNA reference),
  since R = M/N cannot be computed.
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: spikChIP
  role: Pipeline that computes TAG REMOVAL alongside RAW, TRADITIONAL, CHIPRX and
    SPIKCHIP normalizations from the config file and ChromInfo; reads the sample/spike
    BAMs and peak BEDs.
  repo: https://github.com/eblancoga/spikChIP
- name: Samtools
  role: Counts fly spike-in reads per condition (M and N from FLY-suffix chromosomes)
    and performs the downsampling of the human BAM with samtools view -s using factor
    R.
- name: SeqCode
  role: Counts reads per 1000-bp bin (average and maximum) over the down-sampled BAMs
    to produce the TAGREMOVAL normalized tables.
  repo: https://github.com/eblancoga/seqcode
- name: gawk
  role: Manages the plain-text normalized tables and MACS2 broad-peak BED files to
    classify bins into peaks versus background and quantify the background signal
    difference.
- name: MACS2
  role: Upstream peak caller producing the broad-peak BED files used to separate enriched
    bins from background.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqab064
    title: 'SpikChIP: a novel computational methodology to compare multiple ChIP-seq
      using spike-in chromatin'
  source_content_hash: sha256:abde8f87c94ba78a2c9ea0d7ada4daa9e7ef298eca24ddb2468d0db897d4e2ab
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/eblancoga/spikChIP
  repo_url: https://github.com/eblancoga/spikChIP
  tools:
  - MACS2
  - Samtools
  - SeqCode
  - gawk
  - spikChIP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# bam-downsampling-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implements the Tag removal spike-in normalization: scale ChIP-seq BAM files by the ratio R = M/N of fly (dm3) spike-in read totals between two conditions using samtools downsampling, so that both samples carry equal spike-in depth before downstream signal comparison. It is one of the normalization approaches (RAW, TRADITIONAL, CHIPRX, TAG REMOVAL) computed by spikChIP, and is used here to quantify how a uniform global correction affects background genomic regions.

## When to use

Use when comparing two spike-in ChIP-seq conditions (e.g. H3K79me2 25:75 vs 75:25 mixed Jurkat cells) with unequal fly spike-in read totals, and you need to equalize spike-in depth by downsampling the human BAM, or you need a baseline uniform-scaling normalization to benchmark against spikChIP.

## When NOT to use

- When the goal is to minimize overcorrection of non-occupied/background regions — Tag removal applies the spike-in correction uniformly and overcorrects background bins; spikChIP's local regression, which applies the correction gradually from background to enriched regions, is the article's remedy.
- When spike-in sequencing depth is below the recommended ~1 million reads (2%–5% of the experimental genome) — the article recommends the -d option only for processing BAMs < 1 Million reads and warns shallow spike-in undermines normalization.
- When the experiments lack an exogenous spike-in control (no fly/DNA reference), since R = M/N cannot be computed.

## Inputs

- spikChIP configuration plain-text file (sample, bam_sample, bam_spike, peaks_sample, peaks_spike per condition, e.g. H3K79me2_25-75_config.txt)
- Human (hg19) sample BAM files for both conditions (mapped with Bowtie, multi-locus reads discarded)
- Fly (dm3) spike-in BAM files for both conditions (chromosomes with the FLY suffix)
- ChromInfo.txt chromosome size file for genome segmentation
- MACS2 broad-peak BED files for the sample genome (to classify bins as peaks vs background)

## Outputs

- Tag removal scaling factor R = M/N (fly spike-in read totals between conditions)
- Down-sampled human BAM file from the high-spike-in condition (samtools view -s)
- TAGREMOVAL normalized read-count tables per 1000-bp bin, average and maximum (e.g. FINAL_H3K79me2_25-75-75-25_TAGREMOVAL_1000_sample_avg.txt.gz and _max.txt.gz)
- Median percentage difference in ChIP-seq signal over non-enriched background bins between the 25:75 and 75:25 conditions (~20–25% for the H3K79me2 dataset)

## How to apply

Read the spikChIP configuration file (e.g. H3K79me2_25-75_config.txt) and ChromInfo.txt, and verify that human and fly BAM files exist for both conditions. Count fly spike-in reads on FLY-suffixed chromosomes in each condition with samtools to obtain totals M and N, and compute the scaling factor R = M/N. Down-sample the human BAM from the high-spike-in condition using samtools view -s with factor R, so both conditions carry equivalent spike-in coverage. Segment both genomes into 1000-bp bins according to ChromInfo.txt and count reads per bin (average and maximum) with SeqCode to generate the TAGREMOVAL normalized tables (e.g. FINAL_H3K79me2_25-75-75-25_TAGREMOVAL_1000_sample_avg.txt.gz and _max.txt.gz). Use gawk on MACS2 broad-peak BED files to classify bins as peaks versus background and quantify the background-region signal difference between the 25:75 and 75:25 conditions; for this dataset, expect a disproportionate correction yielding an approximately 20–25% difference in the median background signal, comparable to ChIP-Rx overcorrection.

## Related tools

- **spikChIP** (Pipeline that computes TAG REMOVAL alongside RAW, TRADITIONAL, CHIPRX and SPIKCHIP normalizations from the config file and ChromInfo; reads the sample/spike BAMs and peak BEDs.) — https://github.com/eblancoga/spikChIP
- **Samtools** (Counts fly spike-in reads per condition (M and N from FLY-suffix chromosomes) and performs the downsampling of the human BAM with samtools view -s using factor R.)
- **SeqCode** (Counts reads per 1000-bp bin (average and maximum) over the down-sampled BAMs to produce the TAGREMOVAL normalized tables.) — https://github.com/eblancoga/seqcode
- **gawk** (Manages the plain-text normalized tables and MACS2 broad-peak BED files to classify bins into peaks versus background and quantify the background signal difference.)
- **MACS2** (Upstream peak caller producing the broad-peak BED files used to separate enriched bins from background.)

## Examples

```
samtools view -s $R -b H3K79me2_75_sample.bam > H3K79me2_75_sample_TAGREMOVAL_downsampled.bam
```

## Evaluation signals

- The scaling factor R equals the ratio of fly spike-in (FLY-suffix chromosome) read totals M/N between the two conditions, verified with samtools counts.
- Down-sampling equalizes spike-in depth: after samtools view -s, the high-spike-in condition's fly read count approximates the low-spike-in condition's total.
- Compressed TAGREMOVAL output files exist in results/, specifically FINAL_H3K79me2_25-75-75-25_TAGREMOVAL_1000_sample_avg.txt.gz and the matching _max.txt.gz.
- Both the average and maximum per-bin count tables are produced for every condition, and bin coordinates tile the genome in 1000-bp bins per ChromInfo.txt.
- The median background-region (non-peak bin) signal difference between 25:75 and 75:25 conditions falls in the ~20–25% range reported for Tag removal normalization, indicating the expected disproportionate correction over non-enriched bins.

## Limitations

- Tag removal applies the spike-in correction uniformly across all signal, producing a disproportionate correction over non-enriched background bins (~20–25% median difference between 25:75 and 75:25), comparable to the overcorrection seen with ChIP-Rx.
- Different spike-in normalization strategies can introduce inconsistencies to the final results, as each presents its own benefits and limitations; results from Tag removal should not be mixed with spikChIP-corrected tracks.
- BAM files with fewer than 1 million reads require the -d option, and spike-in depth below ~1 million reads (2%–5% of the experimental genome) compromises the normalization.

## Evidence

- [methods] Samtools (to inform about number of reads of BAM files and for downsampling): "Samtools (to inform about number of reads of BAM files and for downsampling)"
- [methods] TAG REMOVAL is one of several normalization approaches computed by the program: "(RAW, TRADITIONAL, CHIPRX and TAG REMOVAL are other normalization approaches also calculated by the program)"
- [readme] README confirms spikChIP implements tag removal among its normalization strategies: "spikChIP adopts a wide range of available normalization strategies (raw, traditional, ChIP-Rx, tag removal)."
- [discussion] Mixing normalization strategies can introduce inconsistencies: "the adoption of different strategies can introduce inconsistencies to the final results, as each method presents its own benefits and limitations"
