---
name: local-regression-signal-correction
description: Use when comparing ChIP-seq conditions (e.g. GSK126-treated H3K27me3,
  fulvestrant ER, EPZ5676 H3K79me2 series) where spike-in ChIP-Rx or tag removal normalization
  overcorrects background signal between conditions.
when_to_use_negative:
- ChIP-seq data without exogenous spike-in chromatin (no fly/human H2Av spike-in reads
  available) — traditional or spike-in-free methods must be used instead
- Single-sample analysis with no condition to compare — spikChIP normalizes across
  multiple conditions via the LOESS regression between samples
- A normalization that must be applied uniformly across all genomic signal is required
  (e.g. you want ChIP-Rx-style global scaling, which spikChIP deliberately avoids
  to protect background regions)
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: spikChIP
  role: Perl pipeline that bins both genomes, counts reads, computes RAW/TRADITIONAL/CHIPRX/TAGREMOVAL/SPIKCHIP
    values and orchestrates the LOESS normalization
  repo: https://github.com/eblancoga/spikChIP
- name: SeqCode
  role: Counts reads inside 1000-bp bins of the BAM files (average and maximum values)
    and overlaps bins with peak BED files
  repo: https://github.com/eblancoga/seqcode
- name: Samtools
  role: Reports number of reads in BAM files and performs downsampling
- name: gawk
  role: Plain-text column handling of bin count files
- name: R (affy, MASS)
  role: Runs normalize.loess on traditionally-normalized values guided by spike-in
    bins (pseudo-count 0.1) to derive the SPIKCHIP correction factors and generates
    PDF boxplots
- name: Bowtie
  role: Maps sample and spike-in reads separately, discarding multi-locus reads, to
    produce the BAM inputs
  repo: http://bowtie-bio.sourceforge.net
- name: MACS2
  role: Calls broad peaks used to classify bins into peaks vs background regions
  repo: https://pypi.org/project/MACS2
provenance:
  source_task_ids:
  - task_007
  - task_008
  - task_006
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqab064
    title: 'SpikChIP: a novel computational methodology to compare multiple ChIP-seq
      using spike-in chromatin'
  source_content_hash: sha256:8471eb20b67d0e6412ccd011b754a7725909018ebbf6c9961a9de7677ec8a44c
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# local-regression-signal-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Apply spikChIP's LOESS (local regression) normalization to ChIP-seq experiments with exogenous Drosophila spike-in chromatin, gradually applying the spike-in-derived correction factor from background to enriched bins so that non-occupied regions are not overcorrected, unlike ChIP-Rx or tag removal.

## When to use

Use when comparing ChIP-seq conditions (e.g. GSK126-treated H3K27me3, fulvestrant ER, EPZ5676 H3K79me2 series) where spike-in ChIP-Rx or tag removal normalization overcorrects background signal between conditions.

## When NOT to use

- ChIP-seq data without exogenous spike-in chromatin (no fly/human H2Av spike-in reads available) — traditional or spike-in-free methods must be used instead
- Single-sample analysis with no condition to compare — spikChIP normalizes across multiple conditions via the LOESS regression between samples
- A normalization that must be applied uniformly across all genomic signal is required (e.g. you want ChIP-Rx-style global scaling, which spikChIP deliberately avoids to protect background regions)

## Inputs

- BAM files of sample genome reads per condition (e.g. human hg19, mapped with Bowtie, multi-locus reads discarded)
- BAM files of spike-in genome reads per condition (e.g. Drosophila dm3)
- MACS2 broad-peak BED files for each sample and spike-in genome
- Plain-text configuration file listing condition name, bam_sample, bam_spike, peaks_sample, peaks_spike per line
- ChromInfo.txt listing chromosome sizes for sample and spike-in genomes (e.g. hg19 plus 'FLY'-suffixed dm3 chromosomes)

## Outputs

- SPIKCHIP-normalized genome-wide bin files (e.g. FINAL_H3K79me2_25-75-75-25_SPIKCHIP_1000_avg_normalized_sample.txt.gz and _spike.txt.gz)
- RAW, TRADITIONAL, CHIPRX and TAGREMOVAL normalized bin values for comparison
- Peaks/background class files (*_avg_peaks.txt.gz and *_avg_bg.txt.gz) per normalization strategy
- PDF boxplots in plots/ comparing normalized peak vs background distributions for sample and spike-in genomes
- Wilcoxon rank-sum P-values quantifying spike-in equilibration and peak vs background signal reduction

## How to apply

Segment both the sample genome (e.g. hg19) and the spike-in genome (dm3 with 'FLY'-suffixed chromosomes) into 1000-bp bins, count reads per bin (average and maximum) with SeqCode and Samtools, and classify bins as peaks or background by overlapping with user-provided MACS2 broad-peak BED files. After computing RAW, TRADITIONAL, CHIPRX and TAGREMOVAL values, run the local regression step in R using the affy normalize.loess function on the traditionally-normalized values, guided by the spike-in (fly) bins with pseudo-count 0.1, to derive per-condition correction factors. Apply the SPIKCHIP correction increasingly and gradually from background to enriched bins rather than uniformly over all ChIP signal, so background reads — which dominate total reads — minimally influence the correction factor. Outputs (FINAL_*_SPIKCHIP_1000_avg_normalized_sample.txt.gz and _spike.txt.gz, plus peak/background class files and PDF boxplots) are written to results/ and plots/, with logging to logs/. Validate by confirming background median signal differs <5% between conditions and spike-in distributions equilibrate (Wilcoxon rank-sum test) while peak signal still reflects the biological change.

## Related tools

- **spikChIP** (Perl pipeline that bins both genomes, counts reads, computes RAW/TRADITIONAL/CHIPRX/TAGREMOVAL/SPIKCHIP values and orchestrates the LOESS normalization) — https://github.com/eblancoga/spikChIP
- **SeqCode** (Counts reads inside 1000-bp bins of the BAM files (average and maximum values) and overlaps bins with peak BED files) — https://github.com/eblancoga/seqcode
- **Samtools** (Reports number of reads in BAM files and performs downsampling)
- **gawk** (Plain-text column handling of bin count files)
- **R (affy, MASS)** (Runs normalize.loess on traditionally-normalized values guided by spike-in bins (pseudo-count 0.1) to derive the SPIKCHIP correction factors and generates PDF boxplots)
- **Bowtie** (Maps sample and spike-in reads separately, discarding multi-locus reads, to produce the BAM inputs) — http://bowtie-bio.sourceforge.net
- **MACS2** (Calls broad peaks used to classify bins into peaks vs background regions) — https://pypi.org/project/MACS2

## Examples

```
./spikChIP.pl -vcp 3 config/H3K27me3_control-EZH2inh_config.txt config/ChromInfo.txt
```

## Evaluation signals

- Median background signal differs by less than 5% between conditions after SPIKCHIP normalization, whereas ChIP-Rx and tag removal shift background bins by ~20-25% median difference (H3K79me2 25:75 vs 75:25 benchmark)
- Spike-in (fly) bin distributions equilibrate between conditions after normalization: Wilcoxon rank-sum P-values such as spikChIP 0.02066 vs ChIP-Rx 3.446e-15 and tag removal <2.2e-16 (ER fulvestrant dataset)
- Peak and background signal respond differently: on GSE64243 H3K27me3, spikChIP shows ~69% reduction over peaks vs ~23% over background, while ChIP-Rx and tag removal show equivalent ~81-85% reduction in both classes (overcorrection signature)
- For multi-sample series (GSE60104 Jurkat H3K79me2 0/25/50/75/100% EPZ5676), peak-region normalized signal declines progressively across the five conditions while background distributions remain flat
- H3K4me3 control shows equivalent peak and background signal levels across all samples, confirming the normalization does not manufacture a spurious trend

## Limitations

- The correction is gradual and region-class-dependent, so results differ from uniform spike-in scaling methods and can introduce inconsistencies when compared across studies using different strategies
- Requires external peak caller (MACS2 broad peaks) classifications supplied by the user; bins rely on user-provided BED files for peaks/background classes
- spike-in reads should be at least one million reads, approximately 2%-5% of the experimental genome, and BAM files of <1 Million reads need the -d option to be processed

## Evidence
