---
name: pipeline-wall-clock-benchmarking
description: Use when comparing runtime efficiency of post-alignment bisulfite methylation
  pipelines, or when a user needs per-step wall-clock timings for extracting, clustering,
  and testing CpG methylation counts on large MethylC-seq datasets.
when_to_use_negative:
- When the goal is detecting batch effects or modeling sets of regions/quantitative
  traits — DMRfinder explicitly does not support these
- When reads are not yet aligned (benchmark covers post-alignment steps only; alignment
  with Bismark precedes the timed steps)
- When input is not Bismark-produced SAM with 'XM' methylation strings and 'XG' genome
  designations
edam_operation: http://edamontology.org/operation_0004
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0622
tools:
- name: DMRfinder
  role: 'Pipeline under benchmark: extracts methylation counts, clusters CpG sites,
    and tests DMRs'
  repo: https://github.com/jsh58/DMRfinder
- name: extract_CpG_data.py
  role: Converts Bismark aligner output into a merged, sorted table of methylated/unmethylated
    CpG counts; timed step 1
  repo: https://github.com/jsh58/DMRfinder
- name: combine_CpG_sites.py
  role: Clusters CpG sites into genomic regions with defaults (-r 3, -s 1, -d 100,
    -c 3, -x 500, -m 20); timed step 2
  repo: https://github.com/jsh58/DMRfinder
- name: findDMRs.r
  role: Performs pairwise DMR testing using DSS beta-binomial hierarchical modeling
    and Wald tests; timed step 3
  repo: https://github.com/jsh58/DMRfinder
- name: Bismark
  role: Upstream aligner producing the SAM/BAM inputs and the comparison benchmark
    for extraction runtime
  repo: https://github.com/FelixKrueger/Bismark
- name: DSS
  role: Statistical framework (Bioconductor package) used by findDMRs.r for beta-binomial
    hierarchical modeling
  repo: http://www.bioconductor.org/packages/release/bioc/html/DSS.html
- name: Samtools
  role: Converts BAM alignments to SAM via `samtools view -h` for piping into extract_CpG_data.py
  repo: http://www.htslib.org/
- name: Python
  role: Runtime for extract_CpG_data.py and combine_CpG_sites.py (tested with versions
    2.7.12 and 3.5.2)
  repo: https://www.python.org/downloads/
- name: R
  role: Runtime for findDMRs.r (tested with Rscript version 3.3.1 and DSS 2.14.0)
  repo: https://www.r-project.org/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-017-1909-0
    title: 'DMRfinder: efficiently identifying differentially methylated regions from
      MethylC-seq data'
  source_content_hash: sha256:e851fa4bb166a76d9a7d30aceed4f6666e3d022d7636ab4986d0ee21006c4d2e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jsh58/DMRfinder
  tools:
  - Bismark
  - DMRfinder
  - DSS
  - Python
  - R
  - Samtools
  - combine_CpG_sites.py
  - extract_CpG_data.py
  - findDMRs.r
  edam_operation: http://edamontology.org/operation_0004
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 18
---

# pipeline-wall-clock-benchmarking

## Summary

Benchmarking the per-step wall-clock runtime (and resource use) of DMRfinder against alternative MethylC-seq DMR-calling software (Bismark, bsseq, BiSeq, DSS) on the large Lister et al. IMR90/H1 MethylC-seq datasets, demonstrating DMRfinder completes each step in a fraction of the time.

## When to use

Use when comparing runtime efficiency of post-alignment bisulfite methylation pipelines, or when a user needs per-step wall-clock timings for extracting, clustering, and testing CpG methylation counts on large MethylC-seq datasets.

## When NOT to use

- When the goal is detecting batch effects or modeling sets of regions/quantitative traits — DMRfinder explicitly does not support these
- When reads are not yet aligned (benchmark covers post-alignment steps only; alignment with Bismark precedes the timed steps)
- When input is not Bismark-produced SAM with 'XM' methylation strings and 'XG' genome designations

## Inputs

- Bismark-aligned SAM or BAM alignment files from Lister et al. MethylC-seq data (IMR90 and H1 human cell lines)
- BAM-to-SAM conversion via `samtools view -h` when BAM input is supplied

## Outputs

- Per-sample merged, sorted per-CpG methylation count tables (.cov)
- Clustered region table (combined.csv) from default clustering parameters
- DMR results table (results.csv) from DSS beta-binomial testing
- runtime_comparison.csv: per-step wall-clock times comparable against other software

## How to apply

Obtain Bismark-aligned SAM/BAM alignments of the Lister et al. IMR90 and H1 MethylC-seq reads (converting BAM to SAM with `samtools view -h` where required). Time step 1: run `extract_CpG_data.py` on each alignment to produce merged, sorted per-CpG methylated/unmethylated count tables. Time step 2: run `combine_CpG_sites.py` with default clustering parameters (-r 3, -s 1, -d 100, -c 3, -x 500, -m 20) to cluster CpG sites into genomic regions. Time step 3: run `findDMRs.r` to perform pairwise DMR testing with DSS beta-binomial hierarchical modeling and Wald tests. Record wall-clock seconds per step, compile into a runtime comparison table, and evaluate against reported competitor runtimes (e.g., extraction of IMR90 replicate 1a taking 9,378 sec with DMRfinder vs 22,514 sec with Bismark, with 193 times less disk space).

## Related tools

- **DMRfinder** (Pipeline under benchmark: extracts methylation counts, clusters CpG sites, and tests DMRs) — https://github.com/jsh58/DMRfinder
- **extract_CpG_data.py** (Converts Bismark aligner output into a merged, sorted table of methylated/unmethylated CpG counts; timed step 1) — https://github.com/jsh58/DMRfinder
- **combine_CpG_sites.py** (Clusters CpG sites into genomic regions with defaults (-r 3, -s 1, -d 100, -c 3, -x 500, -m 20); timed step 2) — https://github.com/jsh58/DMRfinder
- **findDMRs.r** (Performs pairwise DMR testing using DSS beta-binomial hierarchical modeling and Wald tests; timed step 3) — https://github.com/jsh58/DMRfinder
- **Bismark** (Upstream aligner producing the SAM/BAM inputs and the comparison benchmark for extraction runtime) — https://github.com/FelixKrueger/Bismark
- **DSS** (Statistical framework (Bioconductor package) used by findDMRs.r for beta-binomial hierarchical modeling) — http://www.bioconductor.org/packages/release/bioc/html/DSS.html
- **Samtools** (Converts BAM alignments to SAM via `samtools view -h` for piping into extract_CpG_data.py) — http://www.htslib.org/
- **Python** (Runtime for extract_CpG_data.py and combine_CpG_sites.py (tested with versions 2.7.12 and 3.5.2)) — https://www.python.org/downloads/
- **R** (Runtime for findDMRs.r (tested with Rscript version 3.3.1 and DSS 2.14.0)) — https://www.r-project.org/

## Examples

```
samtools view -h C1_bismark_bt2.bam | python extract_CpG_data.py -i - -o C1.cov -v && time python combine_CpG_sites.py -v -o combined.csv C1.cov C2.cov E1.cov E2.cov
```

## Evaluation signals

- Recorded extraction runtime for IMR90 replicate 1a should be on the order of 9,378 sec for DMRfinder versus 22,514 sec reported for Bismark's approach
- Extraction step should consume substantially less disk space (~193 times less than Bismark)
- Each .cov output must be sorted by chromosome (per reference header order) and by 1-based cytosine coordinate, with counts merged across both CpG strands
- combined.csv and results.csv must be produced with the default parameter set (-r 3, -s 1, -d 100, -c 3, -x 500, -m 20) for reproducible timing
- Per-step timings must show DMRfinder completing each analysis step in a fraction of the time required by alternative software

## Limitations

- Benchmark reflects the Lister et al. IMR90/H1 datasets (4.8 billion reads); timings are hardware- and dataset-dependent
- DMRfinder cannot account for batch effects and does not model sets of regions or quantitative traits
- Bismark version 0.16.0 had a bug related to ambiguous read alignments; version 0.15.0 is recommended until fixed
- DMRfinder scripts were tested with Rscript 3.3.1, DSS 2.14.0, and Python 2.7.12/3.5.2; other versions untested in this benchmark

## Evidence

- [abstract] For each step, DMRfinder completed the analysis in a fraction of the time required by other software.: "For each step, DMRfinder completed the analysis in a fraction of the time required by other software."
- [results] extraction runtime result: [evidence span withheld — anchor status: not_found]
- [methods] DMRfinder efficiently identifies genomic regions with differentially methylated CpG sites from high-throughput MethylC-seq datasets: "DMRfinder efficiently identifies genomic regions with differentially methylated CpG sites from high-throughput MethylC-seq datasets"
- [readme] samtools piping invocation: "$ samtools view -h C1_bismark_bt2.bam  |  python extract_CpG_data.py  -i -  -o C1.cov  -v"
- [methods] clustering parameters from task workflow: "-x <int>    Max. length of a region (def. 500)"
- [readme] statistical framework: "It then performs pairwise comparisons to identify differentially methylated regions (DMRs) using the statistical framework of the Bioconductor package DSS"
- [readme] input requirements for extraction: "SAM alignment file produced by Bismark (must have
                    a header, 'XM' methylation strings, and 'XG'
                    genome designations; can use '-' for stdin)"
