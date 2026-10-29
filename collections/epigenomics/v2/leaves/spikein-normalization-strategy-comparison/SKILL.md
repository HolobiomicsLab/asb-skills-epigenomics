---
name: spikein-normalization-strategy-comparison
description: Use when you have human+fly spike-in BAM files (e.g. Jurkat H3K79me2
  25:75 vs 75:25 mixtures, GEO GSM1465005/GSM1465007) and need to quantify how a spike-in
  normalization strategy alters background vs peak-region ChIP signal.
when_to_use_negative:
- The goal is to avoid overcorrection of non-occupied/background regions — use the
  spikChIP local regression normalization instead of Tag removal or ChIP-Rx
- The BAM files contain no fly spike-in alignments (no FLY-suffix chromosomes), so
  the spike-in scaling factor R = M/N cannot be computed
- Only one experimental condition needs normalization (no cross-condition comparison),
  so spike-in strategy comparison has no contrast to quantify
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: spikChIP
  role: Pipeline that reads the configuration file, checks BAM/BED existence, and
    calculates RAW, TRADITIONAL, CHIPRX, TAG REMOVAL and SPIKCHIP normalization approaches
    over 1000-bp bins
  repo: https://github.com/eblancoga/spikChIP
- name: Samtools
  role: Counts fly spike-in reads (M and N) and performs downsampling of the human
    BAM with samtools view -s using factor R
  repo: https://github.com/eblancoga/seqcode
- name: SeqCode
  role: Counts reads per 1000-bp bin (average and maximum values) over bins and peaks
    defined in BED files to build the normalized tables
  repo: https://github.com/eblancoga/seqcode
- name: gawk
  role: Manages the plain-text column files to classify bins into peaks versus background
    and quantify the background-region signal difference
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqab064
    title: 'SpikChIP: a novel computational methodology to compare multiple ChIP-seq
      using spike-in chromatin'
  source_content_hash: sha256:b0b39f712ad688096088e7da1a4166faa5fe6e19ecce4f1b3a803dc9a23b831c
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
  - Samtools
  - SeqCode
  - gawk
  - spikChIP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# spikein-normalization-strategy-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce and compare spike-in ChIP-seq normalization strategies (RAW, TRADITIONAL, CHIPRX, TAG REMOVAL, SPIKCHIP) as calculated by the spikChIP pipeline, quantifying how each strategy corrects signal at non-enriched background regions versus peaks — e.g. the ~20–25% Tag removal background difference between 25:75 and 75:25 H3K79me2 conditions.

## When to use

Use when you have human+fly spike-in BAM files (e.g. Jurkat H3K79me2 25:75 vs 75:25 mixtures, GEO GSM1465005/GSM1465007) and need to quantify how a spike-in normalization strategy alters background vs peak-region ChIP signal.

## When NOT to use

- The goal is to avoid overcorrection of non-occupied/background regions — use the spikChIP local regression normalization instead of Tag removal or ChIP-Rx
- The BAM files contain no fly spike-in alignments (no FLY-suffix chromosomes), so the spike-in scaling factor R = M/N cannot be computed
- Only one experimental condition needs normalization (no cross-condition comparison), so spike-in strategy comparison has no contrast to quantify

## Inputs

- spikChIP plain-text configuration file (e.g. H3K79me2_25-75_config.txt) listing sample BAM, spike-in BAM, and peak BED files per condition
- ChromInfo.txt chromosome-size file
- Mapped human and fruit fly (dm3) BAM files for each condition (e.g. Jurkat K79 25% and 75%)
- MACS2 broad-peak BED files for the human samples

## Outputs

- TAGREMOVAL-normalized 1000-bp bin tables (e.g. FINAL_H3K79me2_25-75-75-25_TAGREMOVAL_1000_sample_avg.txt.gz and _max.txt.gz)
- Tag removal scaling factor R = M/N from fly spike-in read totals
- Median background-region (non-peak bin) signal difference between the 25:75 and 75:25 conditions (expected ~20–25% for Tag removal)

## How to apply

Load the spikChIP configuration file (e.g. H3K79me2_25-75_config.txt) with the ChromInfo.txt file and verify the existence of human and fly BAM files and MACS2 broad-peak BED files for each condition. For the Tag removal strategy, count fly spike-in reads (chromosomes carrying the FLY suffix) in each condition with Samtools to obtain totals M and N, compute the scaling factor R = M/N, and down-sample the human BAM from the high-spike-in condition using samtools view -s with factor R. Segment the genome into 1000-bp bins according to ChromInfo.txt and count reads per bin (average and maximum) with SeqCode to produce TAGREMOVAL-normalized tables, one of the multiple approaches (RAW, TRADITIONAL, CHIPRX, TAG REMOVAL, SPIKCHIP) calculated by spikChIP. Use gawk on the BED peak files to classify bins into peaks versus background, then compute the median background-region signal difference between the 25:75 and 75:25 conditions; for Tag removal this is expected to be ~20–25%, an overcorrection comparable to ChIP-Rx, whereas spikChIP's local regression minimizes overcorrection of non-occupied regions.

## Related tools

- **spikChIP** (Pipeline that reads the configuration file, checks BAM/BED existence, and calculates RAW, TRADITIONAL, CHIPRX, TAG REMOVAL and SPIKCHIP normalization approaches over 1000-bp bins) — https://github.com/eblancoga/spikChIP
- **Samtools** (Counts fly spike-in reads (M and N) and performs downsampling of the human BAM with samtools view -s using factor R) — https://github.com/eblancoga/seqcode
- **SeqCode** (Counts reads per 1000-bp bin (average and maximum values) over bins and peaks defined in BED files to build the normalized tables) — https://github.com/eblancoga/seqcode
- **gawk** (Manages the plain-text column files to classify bins into peaks versus background and quantify the background-region signal difference)

## Examples

```
./spikChIP -vhcde FLY -k 1 -p 0 H3K79me2_25-75_config.txt ChromInfo.txt
```

## Evaluation signals

- The Tag removal scaling factor R = M/N is reproducible from samtools counts of FLY-suffix chromosomes in each condition's BAM
- Compressed TAGREMOVAL output files (FINAL_H3K79me2_25-75-75-25_TAGREMOVAL_1000_sample_avg.txt.gz and _max.txt.gz) exist in results/
- The median background-region difference between 25:75 and 75:25 conditions falls in the ~20–25% range reported for Tag removal normalization
- Tag removal's background overcorrection is comparable to that seen with ChIP-Rx, while SPIKCHIP normalization minimizes overcorrection of non-occupied genomic regions
- Bin tables cover the full genome segmented into 1000-bp bins consistent with ChromInfo.txt and the -k bin_size_kbp option

## Limitations

- Tag removal (like ChIP-Rx) applies the correction uniformly, producing disproportionate corrections over non-enriched background genomic bins; the adoption of different strategies can introduce inconsistencies to the final results, as each method presents its own benefits and limitations
- spikChIP requires spikChIP-reachable external inputs: BAM files generated with a mapping tool (Bowtie, multi-locus reads discarded) and BED peaks from MACS2 broad peak calling before running
- spikChIP by default refuses BAM files of < 1 million reads unless the -d option is passed

## Evidence
