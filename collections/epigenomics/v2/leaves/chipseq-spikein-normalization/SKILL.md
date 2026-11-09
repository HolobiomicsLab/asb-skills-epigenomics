---
name: chipseq-spikein-normalization
description: Use when comparing ChIP-seq signal across conditions (e.g. drug treatment
  series) using spike-in chromatin, and a genome-wide normalization that preserves
  background signal while correcting peak signal is needed.
when_to_use_negative:
- There is no exogenous spike-in chromatin in the experiment (spikChIP requires sample/spike
  BAM pairs and spike-in peak BEDs)
- Only a subset of pre-selected loci needs normalizing and a uniform spike-in factor
  (ChIP-Rx or tag removal) is acceptable — spikChIP is designed for genome-wide, region-class-aware
  normalization
- The spike-in material has fewer than ~1 million reads or falls well below 2%-5%
  of the experimental genome, since normalization reliability depends on sufficient
  spike-in depth
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3678
tools:
- name: spikChIP
  role: 'Main pipeline: bins both genomes, computes RAW/TRADITIONAL/CHIPRX/TAGREMOVAL/SPIKCHIP
    normalizations and classifies bins into peaks/background'
  repo: https://github.com/eblancoga/spikChIP
- name: SeqCode
  role: Counts reads inside 1000-bp bins (average and maximum) and overlaps bins with
    MACS2 peak BED files
  repo: https://github.com/eblancoga/seqcode
- name: Samtools
  role: Reports the number of reads in BAM files and performs downsampling for tag
    removal
- name: gawk
  role: Manages plain-text column files for extracting background bins and computing
    median signal differences
- name: R (affy, MASS)
  role: Runs the local regression (normalize.loess) on traditionally-normalized spike-in
    values with pseudo-count 0.1 to derive spikChIP correction factors and draws boxplots
- name: Bowtie
  role: Maps sample and spike-in reads to hg19 and dm3, discarding multi-locus reads
    (user pre-processing)
- name: MACS2
  role: Calls broad peaks on each sample/spike BAM to supply the peak BED files (user
    pre-processing)
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_006
  - task_008
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqab064
    title: 'SpikChIP: a novel computational methodology to compare multiple ChIP-seq
      using spike-in chromatin'
  source_content_hash: sha256:aab61ffe3d51d6e6cee27ca88b91578980376bd2b43e8ea03a6480f106c2bc2a
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
  - Bowtie
  - MACS2
  - R
  - R (affy, MASS)
  - Samtools
  - SeqCode
  - gawk
  - spikChIP
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3678
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.95
n_claims: 20
---

# chipseq-spikein-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

spikChIP is a Perl pipeline that normalizes multiple ChIP-seq experiments with exogenous spike-in chromatin (e.g. Drosophila DNA mixed into human samples) using a local regression (LOESS) strategy that applies the spike-in correction factor gradually from background to enriched regions, avoiding the background overcorrection seen in ChIP-Rx and tag removal.

## When to use

Use when comparing ChIP-seq signal across conditions (e.g. drug treatment series) using spike-in chromatin, and a genome-wide normalization that preserves background signal while correcting peak signal is needed.

## When NOT to use

- There is no exogenous spike-in chromatin in the experiment (spikChIP requires sample/spike BAM pairs and spike-in peak BEDs)
- Only a subset of pre-selected loci needs normalizing and a uniform spike-in factor (ChIP-Rx or tag removal) is acceptable — spikChIP is designed for genome-wide, region-class-aware normalization
- The spike-in material has fewer than ~1 million reads or falls well below 2%-5% of the experimental genome, since normalization reliability depends on sufficient spike-in depth

## Inputs

- BAM files of each sample/spike genome (e.g. human hg19 reads and fruit fly dm3 reads), mapped with Bowtie with multi-locus reads discarded
- Broad-peak BED files per sample and spike-in, called with MACS2
- Plain-text spikChIP configuration file (one line per condition: condition, bam_sample, bam_spike, peaks_sample, peaks_spike)
- ChromInfo.txt chromosome list (hg19 + dm3 with 'FLY'-suffixed spike-in chromosomes)

## Outputs

- gzip-compressed normalized genome-wide bin tables per strategy, e.g. FINAL_H3K79me2_25-75-75-25_SPIKCHIP_1000_avg_normalized_sample.txt.gz and _spike.txt.gz
- Peak and background bin classification files (*_avg_peaks.txt.gz, *_avg_bg.txt.gz) for each normalization
- Boxplot PDFs in plots/ comparing normalized distributions for peaks vs background across conditions
- Optional summary tables of background median differences or Wilcoxon equilibration P-values (e.g. ER_normalization_wilcoxon_pvalues.tsv)

## How to apply

Provide pre-mapped BAM files (Bowtie, multi-locus reads discarded) for each sample and spike-in genome, MACS2 broad-peak BED files, a plain-text configuration file listing bam_sample/bam_spike/peaks_sample/peaks_spike per condition, and a ChromInfo.txt with spike-in chromosomes marked with a 'FLY' suffix. Run spikChIP with a 1000-bp bin size (-k 1), which segments both genomes, counts reads per bin (average and maximum) with SeqCode, and computes RAW, TRADITIONAL, CHIPRX, TAGREMOVAL and SPIKCHIP normalized values (Samtools reports BAM read totals; gawk handles column files). The SPIKCHIP correction derives LOESS factors in R (affy normalize.loess, MASS library) guided by spike-in bins with pseudo-count 0.1, and applies the correction increasingly from background to peak bins. Results land in results/ as gzip bin tables (e.g. FINAL_<prefix>_SPIKCHIP_1000_avg_normalized_sample.txt.gz and _spike.txt.gz) with peak/background classified bins and boxplot PDFs in plots/.

## Related tools

- **spikChIP** (Main pipeline: bins both genomes, computes RAW/TRADITIONAL/CHIPRX/TAGREMOVAL/SPIKCHIP normalizations and classifies bins into peaks/background) — https://github.com/eblancoga/spikChIP
- **SeqCode** (Counts reads inside 1000-bp bins (average and maximum) and overlaps bins with MACS2 peak BED files) — https://github.com/eblancoga/seqcode
- **Samtools** (Reports the number of reads in BAM files and performs downsampling for tag removal)
- **gawk** (Manages plain-text column files for extracting background bins and computing median signal differences)
- **R (affy, MASS)** (Runs the local regression (normalize.loess) on traditionally-normalized spike-in values with pseudo-count 0.1 to derive spikChIP correction factors and draws boxplots)
- **Bowtie** (Maps sample and spike-in reads to hg19 and dm3, discarding multi-locus reads (user pre-processing))
- **MACS2** (Calls broad peaks on each sample/spike BAM to supply the peak BED files (user pre-processing))

## Examples

```
./spikChIP -vhcde FLY -k 1 -p 1 config/H3K79me2_25-75_config.txt config/ChromInfo.txt
```

## Evaluation signals

- Under spikChIP, the genome-wide median background signal differs by less than 5% between conditions (e.g. 25:75 vs 75:25 H3K79me2), whereas ChIP-Rx and tag removal show ~20-25% median background shift
- Spike-in equilibration Wilcoxon rank-sum P-values between conditions: spikChIP should be least significant (e.g. P=0.02066 ER dataset vs 3.446e-15 for ChIP-Rx; P=0.01547 H3K27me3 vs <2.2e-16 for ChIP-Rx)
- Peak-region normalized signal declines progressively across a titration series (0% to 100% EPZ5676) while background distributions remain flat; H3K4me3 control shows equivalent peak/background levels
- Expected output files exist in results/ (FINAL_*_SPIKCHIP_1000_avg_normalized_sample.txt.gz, _spike.txt.gz, *_avg_peaks.txt.gz, *_avg_bg.txt.gz) and boxplot PDFs appear in plots/
- Normalized tracks should reproduce the published patterns viewable in the deposited UCSC session

## Limitations

- Spike-in reads should be at least one million reads and approximately 2%-5% of the experimental genome to minimize changes in overall material used
- The final amount of spike-in reads can be influenced by the mixture ratio, antibody quality, and target abundance in the experimental condition
- Using the same antibody to capture spike-in material carries a cross-reactivity constraint; a second antibody against a fly-specific histone variant (H2Av) is recommended

## Evidence
