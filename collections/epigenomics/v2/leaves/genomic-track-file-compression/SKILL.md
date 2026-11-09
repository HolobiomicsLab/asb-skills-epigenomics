---
name: genomic-track-file-compression
description: Use when filtered .bam/.bai alignments from a ChIPseq or RNAseq pipeline
  must be converted into genome-browser track files (wig/bigwig) or compact GRanges
  objects for visualization and downstream analysis.
when_to_use_negative:
- When the input is still raw FASTQ data — tracks must be built from filtered, aligned
  .bam/.bai files produced upstream by NEAT's 'map' and 'filter' modules
- When the analysis goal is alignment/peakcalling rather than visualization or GRanges-based
  downstream analysis — use NEAT's map, filter, or peakcalling modules instead
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: NEAT
  role: Framework providing the Bigwig, Wig, and GRanges modules that generate the
    compressed track files from filtered alignments
  repo: https://github.com/pschorderet/NEAT
- name: SAMtools
  role: Upstream tool used by the 'filter' module (view/sort/rmdup/index) to prepare
    the .bam/.bai alignments that feed the track-generation modules
- name: R
  role: Runtime for the 'GRanges' module that converts alignments into compact GRanges
    objects
- name: Bioconductor
  role: Package ecosystem supplying the GRanges infrastructure used by the GRanges
    module
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-016-0902-3
    title: 'NEAT: a framework for building fully automated NGS pipelines and analyses'
  source_content_hash: sha256:8d567828ff33617bb02d17e5401dc15b6e8d37a58b268ce53d08cb025c2bc1a3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pschorderet/NEAT
  tools:
  - Bioconductor
  - NEAT
  - R
  - SAMtools
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# genomic-track-file-compression

## Summary

NEAT's genome-track generation modules (Bigwig, Wig, GRanges) compress processed ChIPseq/RNAseq alignments into visualization-ready tracks: compressed .wig files (~4–6 Mb vs ~4–6 Gb bam), bigwig counterparts, and compact GRanges R objects for downstream analysis.

## When to use

Use when filtered .bam/.bai alignments from a ChIPseq or RNAseq pipeline must be converted into genome-browser track files (wig/bigwig) or compact GRanges objects for visualization and downstream analysis.

## When NOT to use

- When the input is still raw FASTQ data — tracks must be built from filtered, aligned .bam/.bai files produced upstream by NEAT's 'map' and 'filter' modules
- When the analysis goal is alignment/peakcalling rather than visualization or GRanges-based downstream analysis — use NEAT's map, filter, or peakcalling modules instead

## Inputs

- filtered .bam/.bai alignment files (e.g. truncated H3K4me3 ChIPseq test dataset from mouse embryonic stem cells, produced by the 'filter' module)
- AdvancedSettings.txt with bin size, duplicate-read handling and fragment-size filtering parameters

## Outputs

- compressed small-size wiggle (.wig) track files (~4–6 Mb)
- bigwig-format genome-browser tracks
- compact GRanges R/Bioconductor objects (~40–60 Mb, vs ~4–6 Gb for bam)

## How to apply

Run the track modules after NEAT's 'filter' module (which uses samtools view/sort/rmdup/index on the alignment) and 'peakcalling' module. Read the per-base-pair bin size from the project's AdvancedSettings.txt, since wiggle compression is driven by genome-wide binning. The 'Wig' module bins the alignment signal across the genome at that bin size and writes compressed small-size wiggle (.wig) track files; the 'Bigwig' module emits the bigwig-format counterpart for genome-browser use; the 'GRanges' module (R/Bioconductor) converts alignments into significantly smaller GRanges objects necessary for downstream analysis. Order of operations matters: tracks are generated from filtered, duplicate-managed bam files with user-specified fragment-size filters, not from raw alignments.

## Related tools

- **NEAT** (Framework providing the Bigwig, Wig, and GRanges modules that generate the compressed track files from filtered alignments) — https://github.com/pschorderet/NEAT
- **SAMtools** (Upstream tool used by the 'filter' module (view/sort/rmdup/index) to prepare the .bam/.bai alignments that feed the track-generation modules)
- **R** (Runtime for the 'GRanges' module that converts alignments into compact GRanges objects)
- **Bioconductor** (Package ecosystem supplying the GRanges infrastructure used by the GRanges module)

## Evaluation signals

- Produced .wig files fall in the small size range reported by the paper (~4–6 Mb, versus ~4–6 Gb for bam and ~40–60 Mb for GRanges)
- Bigwig and wig tracks are loadable in a genome browser such as IGV or UCSC and display the expected enrichment profile for the H3K4me3 ChIPseq test dataset
- GRanges objects are 'significantly smaller' than the source bam files, as stated for the 'GRanges' module
- Track generation occurs only after 'filter' and 'peakcalling' modules have run, matching the documented pipeline order (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files)
- Bin size used for genome-wide binning matches the value configured in AdvancedSettings.txt

## Limitations

- NEAT is limited to Mac OSx; no Linux/Windows support is stated
- NEAT is still under development ('Devel mode'); bugs may be encountered
- NEAT must be run on the institution's cluster with customizable storage space, job submission limits, wall time and cluster-specific parameters
- Test datasets are truncated (H3K4me3 ChIPseq or RNAseq from mouse embryonic stem cells), so size metrics may differ on full datasets

## Evidence

- [intro] Automated track generation follows filtering and peakcalling: "NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc)"
- [methods] GRanges module produces compact objects for downstream analysis: "The ‘GRanges’ module creates significantly smaller GRanges objects (compared to bam files), which are necessary for downstream analysis"
- [readme] README lists Bigwig, Wig, and GRanges as pipeline steps: "(vii)	Bigwig
    
    (viii)	Wig
    
    (viii)	GRanges"
- [methods] Filter module parameters configured in AdvancedSettings.txt: "The ‘filter’ module allows the user to specify filtering parameters (AdvancedSettings.txt) including how to manage duplicate reads, minimum and maximum size of fragments"
- [abstract] Small-sized outputs enable sharing across users and institutions: "the small-sized files produced by NEAT allow users to easily manipulate, consolidate and share datasets from different users and institutions"
