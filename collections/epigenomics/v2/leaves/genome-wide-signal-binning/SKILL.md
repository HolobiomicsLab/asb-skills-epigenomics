---
name: genome-wide-signal-binning
description: Use when filtered .bam/.bai alignments from a ChIPseq/RNAseq pipeline
  must be converted into compressed genome-browser tracks (.wig/.bigwig) or compact
  GRanges objects for visualization and downstream analysis.
when_to_use_negative:
- Input is raw FASTQ or unaligned reads — binning requires aligned, filtered .bam/.bai
  output from NEAT's 'map' and 'filter' modules.
- The goal is peak calling or ChIP-rx normalization rather than visualization tracks
  — use the 'peakcalling' or 'chiprx' modules instead.
- Input is already a browser track (.wig/.bigwig) or a GRanges object — re-binning
  would duplicate existing outputs.
edam_operation: http://edamontology.org/operation_3208
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3168
tools:
- name: NEAT
  role: Hosts the Wig, Bigwig and GRanges pipeline modules that automate genome-wide
    signal binning of filtered alignments after peakcalling
  repo: https://github.com/pschorderet/NEAT
- name: SAMtools
  role: Used by NEAT's 'filter' module (view/sort/rmdup/index) to produce the filtered
    .bam/.ai inputs that the binning modules consume
- name: R
  role: Environment in which NEAT's GRanges module converts alignments into compact
    GRanges objects
- name: Bioconductor
  role: Consortium providing the GRanges infrastructure used by NEAT's GRanges module
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-016-0902-3
    title: 'NEAT: a framework for building fully automated NGS pipelines and analyses'
  source_content_hash: sha256:11b37dcc1ef4b329b23505b8532ddf36440b8c5dab343cd48d634ce7fa762746
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pschorderet/NEAT
  tools:
  - Bioconductor
  - NEAT
  - R
  - SAMtools
  edam_operation: http://edamontology.org/operation_3208
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# genome-wide-signal-binning

## Summary

NEAT's genome-track generation step bins filtered NGS alignments genome-wide into compressed small-size wiggle (.wig) and bigwig browser tracks plus compact R/Bioconductor GRanges objects, replacing multi-GB .bam files with easily shared small files.

## When to use

Use when filtered .bam/.bai alignments from a ChIPseq/RNAseq pipeline must be converted into compressed genome-browser tracks (.wig/.bigwig) or compact GRanges objects for visualization and downstream analysis.

## When NOT to use

- Input is raw FASTQ or unaligned reads — binning requires aligned, filtered .bam/.bai output from NEAT's 'map' and 'filter' modules.
- The goal is peak calling or ChIP-rx normalization rather than visualization tracks — use the 'peakcalling' or 'chiprx' modules instead.
- Input is already a browser track (.wig/.bigwig) or a GRanges object — re-binning would duplicate existing outputs.

## Inputs

- Filtered .bam/.bai alignment files produced by NEAT's 'filter' module (e.g., truncated H3K4me3 ChIPseq data from mouse embryonic stem cells)
- AdvancedSettings.txt project configuration file specifying the per-base-pair bin size and filtering parameters
- NEAT project folder on the institution's cluster (qsub/bsub)

## Outputs

- Compressed small-size wiggle (.wig) genome track files
- Bigwig-format genome-browser tracks
- Compact GRanges R/Bioconductor objects for downstream analysis

## How to apply

Run after NEAT's 'filter' module (samtools view/sort/rmdup/index) has produced filtered .bam/.bai alignments for each sample, e.g., the truncated H3K4me3 ChIPseq test dataset from mouse embryonic stem cells. Read the per-base-pair bin size from AdvancedSettings.txt, since wiggle compression is driven by genome-wide binning at that resolution. Implement the Wig module to bin alignment signal across the genome at the configured bin size and write compressed small-size wiggle (.wig) files, then the Bigwig module to emit the bigwig counterpart for genome-browser loading (e.g., IGV). In parallel, the GRanges module in R/Bioconductor converts the alignments into significantly smaller GRanges objects that downstream analysis requires. Decision point: deliver .wig/.bigwig when visualization tracks are the goal and GRanges when R-based downstream analysis will consume the signal; in the README pipeline Bigwig, Wig and GRanges run as automated steps after peakcalling. Expect order-of-magnitude shrinkage (~4-6 Mb .wig versus ~4-6 Gb .bam, ~40-60 Mb GRanges), which is what makes NEAT outputs easy to manipulate, consolidate and share across institutions.

## Related tools

- **NEAT** (Hosts the Wig, Bigwig and GRanges pipeline modules that automate genome-wide signal binning of filtered alignments after peakcalling) — https://github.com/pschorderet/NEAT
- **SAMtools** (Used by NEAT's 'filter' module (view/sort/rmdup/index) to produce the filtered .bam/.ai inputs that the binning modules consume)
- **R** (Environment in which NEAT's GRanges module converts alignments into compact GRanges objects)
- **Bioconductor** (Consortium providing the GRanges infrastructure used by NEAT's GRanges module)

## Evaluation signals

- .wig output sizes land in the small-size range reported for NEAT (~4-6 Mb versus ~4-6 Gb for the input .bam)
- Bigwig tracks load in a genome browser such as IGV and show signal consistent with the filtered H3K4me3 ChIPseq alignment
- GRanges objects (~40-60 Mb, far smaller than the .bam) load in R/Bioconductor and are usable for downstream analysis
- Binning resolution of the .wig output matches the bin size configured in AdvancedSettings.txt
- Wig/Bigwig/GRanges modules run as automated pipeline steps after 'peakcalling' without manual file manipulation

## Limitations

- NEAT was still under development at publication (README 'Devel mode': 'If you encounter any bug, please contact us').
- Operating system listed as Mac OSx only; no Linux/Windows support stated in the discussion.
- Track content depends on upstream 'filter' parameters in AdvancedSettings.txt (duplicate-read handling, minimum and maximum fragment sizes), so misconfigured filtering propagates into the tracks.
- Test datasets are truncated, so absolute file-size expectations differ from full-scale runs.

## Evidence

- [intro] NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc): "NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc)"
- [methods] The 'GRanges' module creates significantly smaller GRanges objects (compared to bam files), which are necessary for downstream analysis: [evidence span withheld — anchor status: not_found]
- [abstract] the small-sized files produced by NEAT allow users to easily manipulate, consolidate and share datasets from different users and institutions: "the small-sized files produced by NEAT allow users to easily manipulate, consolidate and share datasets from different users and institutions"
- [readme] Novice users can follow through the entire analysis of a test dataset provided as part of the package.: "Novice users can follow through the entire analysis of a test dataset provided as part of the package."
