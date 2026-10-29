---
name: wilcoxon-spike-in-equilibration-testing
description: Use when you have spikChIP output bin files (FINAL_*_spike and FINAL_*_sample
  peak/background classes) for ≥2 conditions and need to rank normalization strategies
  by spike-in equilibration quality and detect background overcorrection.
when_to_use_negative:
- The ChIP-seq experiment has no exogenous spike-in chromatin (no fly/H2Av or other
  exogenous control), so no spike-in equilibration can be measured
- Spike-in sequencing depth is far below the recommended ~1 million reads (~2%–5%
  of the experimental genome), making the spike-in distribution too noisy for reliable
  rank-sum testing (the spikChIP -d flag is even needed for BAM files < 1M reads)
- Only a single condition/sample is available — the Wilcoxon test requires two condition-specific
  normalized distributions to compare
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: spikChIP
  role: Generates the per-bin normalized spike-in and sample values (RAW, TRADITIONAL,
    CHIPRX, TAGREMOVAL, SPIKCHIP) and peak/background class files that the Wilcoxon
    tests are run on
  repo: https://github.com/eblancoga/spikChIP
- name: R
  role: Runs the Wilcoxon rank-sum tests; spikChIP itself uses R (affy and MASS libraries)
    for the local regression underlying the SPIKCHIP strategy
- name: MACS2
  role: Produces the broad-peak BED files used to classify bins into peaks vs background
    before testing
- name: SeqCode
  role: Counts reads of BAM files and overlaps bins and peaks of BED files inside
    the spikChIP pipeline
  repo: https://github.com/eblancoga/seqcode
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1093/nargab/lqab064
    title: 'SpikChIP: a novel computational methodology to compare multiple ChIP-seq
      using spike-in chromatin'
  source_content_hash: sha256:009df572875c8e15d8fdc6db84152529b15538ed197b2ec5b07514a8edf6d1dd
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
  - Samtools
  - SeqCode
  - gawk
  - spikChIP
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# wilcoxon-spike-in-equilibration-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how well each spike-in ChIP-seq normalization strategy (ChIP-Rx, Tag removal, spikChIP) equilibrated the fly spike-in signal between experimental conditions by running Wilcoxon rank-sum tests on the normalized spike-in read distributions over genome bins, then comparing peak vs background signal reduction. It is the acceptance test that distinguishes overcorrection of non-enriched background from genuine locus-specific signal loss.

## When to use

Use when you have spikChIP output bin files (FINAL_*_spike and FINAL_*_sample peak/background classes) for ≥2 conditions and need to rank normalization strategies by spike-in equilibration quality and detect background overcorrection.

## When NOT to use

- The ChIP-seq experiment has no exogenous spike-in chromatin (no fly/H2Av or other exogenous control), so no spike-in equilibration can be measured
- Spike-in sequencing depth is far below the recommended ~1 million reads (~2%–5% of the experimental genome), making the spike-in distribution too noisy for reliable rank-sum testing (the spikChIP -d flag is even needed for BAM files < 1M reads)
- Only a single condition/sample is available — the Wilcoxon test requires two condition-specific normalized distributions to compare

## Inputs

- FINAL_*_spike and FINAL_*_sample peak/background bin files produced by spikChIP for each normalization strategy and condition
- *_avg_peaks.txt.gz and *_avg_bg.txt.gz class files from the spikChIP results/ folder
- 1000-bp binned normalized values for RAW, TRADITIONAL, CHIPRX, TAGREMOVAL and SPIKCHIP strategies
- Two-condition configuration file (e.g. config/H3K27me3_control-EZH2inh_config.txt) and ChromInfo.txt with 'FLY'-suffixed spike-in chromosomes

## Outputs

- Wilcoxon rank-sum P-values per normalization strategy comparing spike-in (and sample) bin distributions between conditions (e.g. ChIP-Rx <2.2e-16, Tag removal 9.611e-14, spikChIP 0.01547 for H3K27me3)
- Percentage of ChIP signal reduction in peaks vs background regions relative to the control condition (e.g. spikChIP 69% peaks vs 23% background)
- Summary TSV (e.g. GSE64243_normalization_metrics.tsv or ER_normalization_wilcoxon_pvalues.tsv)

## How to apply

After running spikChIP (which segments both genomes into 1000-bp bins and computes RAW, TRADITIONAL, CHIPRX, TAGREMOVAL and SPIKCHIP normalized values), classify bins into peaks and background using the user-supplied BED peak files (e.g. MACS2 broad peaks) and extract the *_avg_peaks.txt.gz and *_avg_bg.txt.gz class files from results/. For each normalization strategy, take the normalized spike-in bin distributions (from the FINAL_*_spike peak and background files) for each condition and run a Wilcoxon rank-sum test between conditions; apply the same test to the sample (experimental genome) peak and background bins. Interpretation: a low spike-in P-value means the spike-in signal remains uneven between conditions, indicating a poorly equilibrating strategy; the strategy with the highest (least significant) P-value, typically spikChIP (0.01547 for GSE64243 H3K27me3, 0.02066 for GSE102882 ER), equilibrated best. Finally, compute percentage signal reduction in peaks vs background relative to the control condition — ChIP-Rx and Tag removal reduce signal ~81–85% in both classes (overcorrection), whereas spikChIP shows distinct reduction (69% peaks vs 23% background for H3K27me3 GSK126), and write the metrics to a TSV.

## Related tools

- **spikChIP** (Generates the per-bin normalized spike-in and sample values (RAW, TRADITIONAL, CHIPRX, TAGREMOVAL, SPIKCHIP) and peak/background class files that the Wilcoxon tests are run on) — https://github.com/eblancoga/spikChIP
- **R** (Runs the Wilcoxon rank-sum tests; spikChIP itself uses R (affy and MASS libraries) for the local regression underlying the SPIKCHIP strategy)
- **MACS2** (Produces the broad-peak BED files used to classify bins into peaks vs background before testing)
- **SeqCode** (Counts reads of BAM files and overlaps bins and peaks of BED files inside the spikChIP pipeline) — https://github.com/eblancoga/seqcode

## Examples

```
wilcox.test(read.table('FINAL_CHIPRX_control_spike.txt')[,4], read.table('FINAL_CHIPRX_treated_spike.txt')[,4])$p.value
```

## Evaluation signals

- Spike-in Wilcoxon P-value for spikChIP should be the least significant among strategies (target values: 0.01547 for the H3K27me3 GSK126 dataset, 0.02066 for the ER fulvestrant dataset)
- ChIP-Rx and Tag removal spike-in P-values should be strongly significant (e.g. <2.2e-16 and 9.611e-14 for H3K27me3; 3.446e-15 and <2.2e-16 for ER), reflecting poor equilibration
- Signal-reduction percentages should show ChIP-Rx and Tag removal reducing signal ~81–85% in BOTH peaks and background, while spikChIP separates the two classes (69% peaks vs 23% background for H3K27me3)
- The spikChIP strategy must show signal loss concentrated at enriched loci, not uniform background reduction between conditions
- The summary TSV must contain P-values and peak/background reduction percentages for all compared strategies (e.g. GSE64243_normalization_metrics.tsv)

## Limitations

- The number of spike-in reads in the final sequencing step should be at least one million reads, approximately 2%–5% of the experimental genome; the final amount can be influenced by the mixture ratio, antibody quality, and target abundance
- Spike-in sequencing noise can distort normalization; spikChIP specifically aims to reduce its influence, but background bins still dominate total read counts
- spikChIP requires prior peak calling (external peak caller, e.g. MACS2 broad peaks) and pre-built BAM files from a mapper such as Bowtie; the correction factor is applied gradually from background to enriched bins, so results depend on peak-call quality

## Evidence
