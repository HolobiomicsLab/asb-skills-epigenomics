---
name: chipseq-duplicate-read-filtering
description: Use when processing mapped ChIPseq reads (BAM) downstream of the 'map'
  module and before peakcalling, and duplicate-read handling and fragment-size bounds
  must be applied reproducibly across a project.
when_to_use_negative:
- The input is raw FASTQ that has not yet been aligned — filtering runs after the
  'map' module, not before it
- The project is RNAseq differential expression analysis (NEAT's QC/DEG track) rather
  than ChIPseq peak calling
- No duplicate-read or fragment-size handling is desired and default unfiltered alignments
  should be passed straight to peakcalling
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3163
tools:
- name: NEAT
  role: Framework hosting the 'filter' module; parameters are centralized in the project
    file and AdvancedSettings.txt
  repo: https://github.com/pschorderet/NEAT
- name: SAMtools
  role: Implements view/sort/rmdup/index operations for duplicate-read handling and
    fragment-size filtering on BAM files
- name: bwa
  role: Upstream aligner in the 'map' module whose output BAM is the filter step's
    input; same parameters must be used for spike-in
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-016-0902-3
    title: 'NEAT: a framework for building fully automated NGS pipelines and analyses'
  source_content_hash: sha256:565aee455e68f19c9194036b701415f5683ad257fafa094674c60d1f4ca249aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pschorderet/NEAT
  tools:
  - NEAT
  - R
  - SAMtools
  - bwa
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3163
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# chipseq-duplicate-read-filtering

## Summary

A ChIPseq pipeline step from NEAT that filters aligned reads using user-specified parameters — notably how to manage duplicate reads plus minimum and maximum fragment sizes — applied identically to standard and, when present, spike-in alignments before peakcalling.

## When to use

Use when processing mapped ChIPseq reads (BAM) downstream of the 'map' module and before peakcalling, and duplicate-read handling and fragment-size bounds must be applied reproducibly across a project.

## When NOT to use

- The input is raw FASTQ that has not yet been aligned — filtering runs after the 'map' module, not before it
- The project is RNAseq differential expression analysis (NEAT's QC/DEG track) rather than ChIPseq peak calling
- No duplicate-read or fragment-size handling is desired and default unfiltered alignments should be passed straight to peakcalling

## Inputs

- Aligned ChIPseq BAM files produced by the 'map' module (bwa or bowtie)
- Aligned spike-in (exogenous epigenome) BAM files for ChIP-rx workflows
- AdvancedSettings.txt template specifying duplicate-read handling and minimum/maximum fragment sizes

## Outputs

- Filtered, duplicate-processed, sorted and indexed BAM/BAI files
- Alignment files consistent between standard and spike-in samples, ready for peakcalling and ChIP-rx scaling

## How to apply

In NEAT's pipeline, the 'filter' module sits between 'map' (bwa or bowtie alignment) and 'peakcalling', with all settings read from the project's centralized file and the AdvancedSettings.txt template. The user specifies how duplicate reads are managed and sets minimum and maximum fragment-size bounds; the module then processes the aligned .bam files (with samtools view/sort/rmdup/index in the reconstructed workflow). When performing ChIP-rx normalization, apply exactly the same filtering parameters to the spike-in (exogenous epigenome) alignment, because ChIP-RX requires consistent treatment of the two alignments before genome-wide scaling of the signal. Filtered alignments feed the 'peakcalling' module and downstream wig/bigwig file creation for genome browsers such as IGV.

## Related tools

- **NEAT** (Framework hosting the 'filter' module; parameters are centralized in the project file and AdvancedSettings.txt) — https://github.com/pschorderet/NEAT
- **SAMtools** (Implements view/sort/rmdup/index operations for duplicate-read handling and fragment-size filtering on BAM files)
- **bwa** (Upstream aligner in the 'map' module whose output BAM is the filter step's input; same parameters must be used for spike-in)

## Examples

```
samtools rmdup sample.bam sample.rmdup.bam && samtools index sample.rmdup.bam
```

## Evaluation signals

- The filter module runs end-to-end on the truncated H3K4me3 ChIPseq test dataset shipped with the NEAT repository without errors
- Filtered BAM files carry no reads flagged as duplicates under the chosen duplicate-read management setting
- All retained fragments fall within the minimum and maximum sizes specified in AdvancedSettings.txt
- Standard and spike-in alignments have been filtered with identical parameters (required for valid ChIP-rx scaling)
- Downstream peakcalling and wig-file creation produce non-empty outputs from the filtered alignments

## Limitations

- NEAT is developed for Mac OSx; no Linux/Windows support is stated in the discussion section
- The paper presents a pipeline, so tangible results are in the form of outputs (Fig. 3) rather than quantitative benchmarks
- No concrete thresholds for duplicate handling are stated in the article — they are user-specified via AdvancedSettings.txt

## Evidence

- [methods] The ‘filter’ module allows the user to specify filtering parameters (AdvancedSettings.txt) including how to manage duplicate reads, minimum and maximum size of fragments: "The ‘filter’ module allows the user to specify filtering parameters (AdvancedSettings.txt) including how to manage duplicate reads, minimum and maximum size of fragments"
- [intro] NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc): "NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc)"
- [readme] Novice users can follow through the entire analysis of a test dataset provided as part of the package.: "Novice users can follow through the entire analysis of a test dataset provided as part of the package."
- [methods] ChIPrx [17] is a cutting edge normalization method for ChIPseq that performs genome-wide quantitative comparisons: "ChIPrx [17] is a cutting edge normalization method for ChIPseq that performs genome-wide quantitative comparisons"
