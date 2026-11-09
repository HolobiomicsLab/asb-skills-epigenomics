---
name: spikein-read-ratio-scaling-factor-calculation
description: Use when comparing two spike-in ChIP-seq conditions (e.g. H3K79me2 25:75
  vs 75:25 with fly chromatin) and you need a ratio-based scaling factor to equalize
  spike-in read depth before bin-level signal counting.
when_to_use_negative:
- When the goal is to avoid overcorrection of non-occupied/background regions — the
  article shows tag removal (like ChIP-Rx) applies the correction uniformly and yields
  a ~20-25% background signal difference; use spikChIP's local regression instead
- When sample and spike-in BAM files have not yet been generated with a mapping tool
  (Bowtie with multi-locus reads discarded) and peaks not yet called with MACS2
- When spike-in reads are far below the recommended minimum of ~1 million reads (2%-5%
  of the experimental genome), making M/N unstable
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0769
tools:
- name: Samtools
  role: Counts fly spike-in reads (M and N) per condition and down-samples the high-spike-in
    human BAM via samtools view -s with factor R
  repo: https://github.com/eblancoga/seqcode
- name: spikChIP
  role: Pipeline whose configuration file declares sample/spike BAM and peak files
    and which also implements TAG REMOVAL as one of its built-in normalization approaches
  repo: https://github.com/eblancoga/spikChIP
- name: SeqCode
  role: Counts reads of the down-sampled BAMs per 1000-bp bin (average and maximum)
    and overlaps bins with BED peaks to produce the TAGREMOVAL normalized tables
  repo: https://github.com/eblancoga/seqcode
- name: gawk
  role: Manipulates the plain-text bin/peak tables to classify bins as peaks or background
    and quantify the background-region signal difference
- name: MACS2
  role: Calls broad peaks (BED) used to define the enriched versus background bin
    classes
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqab064
    title: 'SpikChIP: a novel computational methodology to compare multiple ChIP-seq
      using spike-in chromatin'
  source_content_hash: sha256:76a9cde05e06396cdfa3b1264e1255b8b5a729aea99c3ae8fa9ca1f32b0098e8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/eblancoga/spikChIP
  repo_url: https://github.com/eblancoga/seqcode
  tools:
  - MACS2
  - Samtools
  - SeqCode
  - gawk
  - spikChIP
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# spikein-read-ratio-scaling-factor-calculation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes a spike-in-based scaling factor R = M/N from fly spike-in read totals of two ChIP-seq conditions and down-samples the high-spike-in sample with Samtools (the 'tag removal' normalization strategy also implemented inside spikChIP), producing comparable BAM inputs for genome-wide signal comparison.

## When to use

Use when comparing two spike-in ChIP-seq conditions (e.g. H3K79me2 25:75 vs 75:25 with fly chromatin) and you need a ratio-based scaling factor to equalize spike-in read depth before bin-level signal counting.

## When NOT to use

- When the goal is to avoid overcorrection of non-occupied/background regions — the article shows tag removal (like ChIP-Rx) applies the correction uniformly and yields a ~20-25% background signal difference; use spikChIP's local regression instead
- When sample and spike-in BAM files have not yet been generated with a mapping tool (Bowtie with multi-locus reads discarded) and peaks not yet called with MACS2
- When spike-in reads are far below the recommended minimum of ~1 million reads (2%-5% of the experimental genome), making M/N unstable

## Inputs

- spikChIP configuration plain-text file (e.g. H3K79me2_25-75_config.txt) declaring bam_sample, bam_spike and BED peak paths per condition
- Human sample BAM files and fly spike-in BAM files for both conditions (mapped with Bowtie, multi-locus reads discarded)
- ChromInfo.txt chromosome-size file for genome segmentation
- MACS2 broad-peak BED files for peak/background classification

## Outputs

- Scaling factor R = M/N computed from fly spike-in read totals
- Down-sampled human BAM from the high-spike-in condition (samtools view -s)
- TAGREMOVAL normalized 1000-bp bin count tables (e.g. FINAL_H3K79me2_25-75-75-25_TAGREMOVAL_1000_sample_avg.txt.gz and _max.txt.gz)
- Background-region versus peak-bin signal comparison between the 25:75 and 75:25 conditions

## How to apply

From the spikChIP configuration file (e.g. H3K79me2_25-75_config.txt) and ChromInfo.txt, first verify that the human sample BAM and fly spike-in BAM exist for both conditions. Count fly spike-in reads (chromosomes carrying the FLY suffix) in each condition with Samtools to obtain totals M and N, then compute the scaling factor R = M/N. Down-sample the human BAM from the condition with more spike-in reads using samtools view -s with factor R, so both conditions carry equivalent spike-in depth. The down-sampled BAMs are then binned (1000-bp bins per ChromInfo.txt) and read counts per bin are computed with SeqCode (average and maximum) to yield TAGREMOVAL normalized tables, which can be classified with gawk into MACS2 broad-peak versus background bins. Note that this strategy applies the correction uniformly, so it produces a disproportionate correction (~20-25% median difference) over non-enriched background bins — the overcorrection spikChIP's local regression was designed to avoid.

## Related tools

- **Samtools** (Counts fly spike-in reads (M and N) per condition and down-samples the high-spike-in human BAM via samtools view -s with factor R) — https://github.com/eblancoga/seqcode
- **spikChIP** (Pipeline whose configuration file declares sample/spike BAM and peak files and which also implements TAG REMOVAL as one of its built-in normalization approaches) — https://github.com/eblancoga/spikChIP
- **SeqCode** (Counts reads of the down-sampled BAMs per 1000-bp bin (average and maximum) and overlaps bins with BED peaks to produce the TAGREMOVAL normalized tables) — https://github.com/eblancoga/seqcode
- **gawk** (Manipulates the plain-text bin/peak tables to classify bins as peaks or background and quantify the background-region signal difference)
- **MACS2** (Calls broad peaks (BED) used to define the enriched versus background bin classes)

## Examples

```
samtools view -c sample_FLY.bam  # total N; samtools view -s 0.62 -b sample_HUMAN.bam > sample_HUMAN_downsampled.bam  # R = M/N
```

## Evaluation signals

- Compressed TAGREMOVAL outputs FINAL_H3K79me2_25-75-75-25_TAGREMOVAL_1000_sample_avg.txt.gz and _max.txt.gz exist in results/
- Fly spike-in read totals M and N are reported for both conditions and R = M/N is a plausible down-sampling factor (<1, applied to the high-spike-in condition)
- Median background-region (non-peak bin) signal difference between the 25:75 and 75:25 conditions falls in the ~20-25% range reported for tag removal normalization (Figure 4B and C)
- Down-sampled BAM contains approximately R times the reads of the original high-spike-in sample BAM
- Bin counts are computed on 1000-bp bins consistent with ChromInfo.txt and the -k bin size parameter

## Limitations

- TAG REMOVAL (like RAW, TRADITIONAL and CHIPRX) applies the spike-in correction uniformly, so it overcorrects non-enriched background regions rather than concentrating the correction on peak signal as spikChIP does
- Adoption of different spike-in normalization strategies can introduce inconsistencies in the final results, as each method presents its own benefits and limitations
- The final amount of spike-in reads can be influenced by the mixture ratio and by antibody quality or target abundance, affecting the stability of M and N

## Evidence

- [results] Tag removal normalization produces a disproportionate correction over non-enriched background genomic bins, yielding an approximately 20-25% difference in the median of ChIP-seq signal between the 25:75 and 75:25 conditions: [evidence span withheld — anchor status: not_found]
