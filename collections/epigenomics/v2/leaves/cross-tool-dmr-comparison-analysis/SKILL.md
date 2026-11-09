---
name: cross-tool-dmr-comparison-analysis
description: Use when benchmarking DMR-calling tools for false-positive rates (e.g.,
  contrasting replicates of the same sample should produce zero DMRs) or reproducing
  the DMRfinder vs. DSS/bsseq comparison.
when_to_use_negative:
- The input is raw methylation counts needing only a single-tool DMR call rather than
  a cross-tool benchmark (use DMRfinder directly instead).
- The analysis requires modeling sets of regions, quantitative traits, or batch effects
  — DMRfinder explicitly does not support these.
- Only one sample/replicate per condition is available, so no same-sample replicate
  contrast (the key false-positive control) can be run.
edam_operation: http://edamontology.org/operation_3207
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
tools:
- name: DMRfinder
  role: 'Primary pipeline: extract_CpG_data.py, combine_CpG_sites.py, and findDMRs.r
    perform post-alignment methylation extraction, region clustering, and beta-binomial
    DMR testing'
  repo: https://github.com/jsh58/DMRfinder
- name: Bismark
  role: Aligns MethylC-seq reads to the reference genome via in silico bisulfite conversion
    and Bowtie2, producing SAM output consumed by extract_CpG_data.py
  repo: https://github.com/FelixKrueger/Bismark
- name: DSS
  role: Statistical framework (beta-binomial hierarchical modeling) used by DMRfinder
    and also run standalone as a comparison package
  repo: http://www.bioconductor.org/packages/release/bioc/html/DSS.html
- name: bsseq
  role: Alternative R/Bioconductor DMR-calling package run on the same aligned data
    to quantify false positives on replicate contrasts
- name: Bowtie2
  role: Aligner invoked by Bismark for read mapping
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-017-1909-0
    title: 'DMRfinder: efficiently identifying differentially methylated regions from
      MethylC-seq data'
  source_content_hash: sha256:69234b8ae5a613bd2080f3cfd197a8686efee3511200715b51c5f8514f93d5f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jsh58/DMRfinder
  tools:
  - Bismark
  - Bowtie2
  - DMRfinder
  - DSS
  - Python
  - R
  - bsseq
  - combine_CpG_sites.py
  - extract_CpG_data.py
  - findDMRs.r
  edam_operation: http://edamontology.org/operation_3207
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9565217391304348
n_claims: 23
---

# cross-tool-dmr-comparison-analysis

## Summary

Reproduce the DMRfinder benchmark against bsseq, BiSeq, and DSS on Lister et al. IMR90/H1 MethylC-seq data, including same-sample replicate contrasts that should yield no DMRs, to quantify false positives and statistical differences between tools.

## When to use

Use when benchmarking DMR-calling tools for false-positive rates (e.g., contrasting replicates of the same sample should produce zero DMRs) or reproducing the DMRfinder vs. DSS/bsseq comparison.

## When NOT to use

- The input is raw methylation counts needing only a single-tool DMR call rather than a cross-tool benchmark (use DMRfinder directly instead).
- The analysis requires modeling sets of regions, quantitative traits, or batch effects — DMRfinder explicitly does not support these.
- Only one sample/replicate per condition is available, so no same-sample replicate contrast (the key false-positive control) can be run.

## Inputs

- Lister et al. IMR90 and H1 MethylC-seq FASTQ reads (4.8 billion reads)
- SAM/BAM alignment files from Bismark with 'XM' methylation strings and 'XG' genome designations
- Reference genome for Bismark/Bowtie2 alignment
- Merged CpG count table (combined.csv) from combine_CpG_sites.py

## Outputs

- Sorted per-sample CpG count tables (*.cov files)
- combined.csv of clustered genomic regions
- results.csv of DMR test results per region
- dmr_comparison_results.csv tabulating DMR counts and false positives per tool

## How to apply

Align Lister et al. IMR90 and H1 MethylC-seq reads with Bismark v0.15.0 (Bowtie2), since a bug related to ambiguous read alignments affected Bismark 0.16.0. Extract merged CpG methylation counts per sample using extract_CpG_data.py to produce sorted count tables (*.cov files), then cluster CpGs into regions with combine_CpG_sites.py using default parameters (-r 3, -s 1, -d 100, -c 3, -x 500, -m 20). Test regions for differential methylation with findDMRs.r, which applies DSS beta-binomial hierarchical modeling and Wald tests with thresholds -d 0.10 (min methylation difference) and -p 0.05 (max p-value). Run the comparison packages bsseq, BiSeq, and DSS on the same aligned data, including a contrast of two replicates of the same sample (IMR90 replicates) to quantify false positives, and tabulate DMR counts and the call differences between DMRfinder and DSS (which share the same statistical basis). The key validation is that the same-sample replicate contrast yields minimal regions with DMRfinder while alternative packages report a substantial number of false positives.

## Related tools

- **DMRfinder** (Primary pipeline: extract_CpG_data.py, combine_CpG_sites.py, and findDMRs.r perform post-alignment methylation extraction, region clustering, and beta-binomial DMR testing) — https://github.com/jsh58/DMRfinder
- **Bismark** (Aligns MethylC-seq reads to the reference genome via in silico bisulfite conversion and Bowtie2, producing SAM output consumed by extract_CpG_data.py) — https://github.com/FelixKrueger/Bismark
- **DSS** (Statistical framework (beta-binomial hierarchical modeling) used by DMRfinder and also run standalone as a comparison package) — http://www.bioconductor.org/packages/release/bioc/html/DSS.html
- **bsseq** (Alternative R/Bioconductor DMR-calling package run on the same aligned data to quantify false positives on replicate contrasts)
- **Bowtie2** (Aligner invoked by Bismark for read mapping) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml

## Examples

```
python combine_CpG_sites.py -v -o combined.csv C1.cov C2.cov E1.cov E2.cov && Rscript findDMRs.r -i combined.csv -o results.csv -v -n Control,Exptl C1,C2 E1,E2
```

## Evaluation signals

- The IMR90 replicate-vs-replicate contrast yields zero DMRs with DSS and two DMRs with DMRfinder (minimal false positives), while other packages report a substantial number of false positives.
- extract_CpG_data.py output *.cov files are sorted by chromosome and position, with methylation counts merged across both strands of each CpG.
- Region clustering uses the documented defaults: min 3 counts per CpG (-r 3), max 100 bp CpG spacing (-d 100), min 3 CpGs per region (-c 3), max 500 bp region length (-x 500), min 20 total counts (-m 20).
- DMR calling thresholds are respected: minimum methylation difference 0.10 (-d 0.10) and maximum p-value 0.05 (-p 0.05).
- Biological-sample contrasts reveal the reported fundamental call differences between DMRfinder and DSS despite their identical underlying statistical basis (DSS).

## Limitations

- DMRfinder identifies individual differentially methylated regions but does not consider sets of regions or quantitative traits.
- DMRfinder cannot account for batch effects.
- Bismark version 0.16.0 contained a bug related to ambiguous read alignments; version 0.15.0 is recommended until fixed.
- Bismark's deduplication script is not recommended — it keeps the first read at a position regardless of sequence or methylation information.

## Evidence

- [abstract] Contrasting two replicates of the same sample yielded minimal genomic regions with DMRfinder, whereas two alternative software packages reported a substantial number of false positives.: "Contrasting two replicates of the same sample yielded minimal genomic regions with DMRfinder, whereas two alternative software packages reported a substantial number of false positives."
- [abstract] DMRfinder vs DSS same statistical basis: "Further analyses of biological samples revealed fundamental differences between DMRfinder and another software package, despite the fact that they utilize the same underlying statistical basis."
- [readme] Bismark version recommendation: [evidence span withheld — anchor status: not_found]
- [methods] findDMRs.r thresholds: "-d <float>    Min. methylation difference between sample groups
                    ([0-1]; def. 0.10)"
- [methods] Region length threshold: "The DMRfinder script limits this effect by splitting any region whose length exceeds a specified threshold (by default, 500 bp)"
