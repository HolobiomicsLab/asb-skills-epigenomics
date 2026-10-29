---
name: chipseq-tss-meta-plot-generation
description: Use when you have a mapped ChIP-seq BAM (e.g. H3K4me3 in mESC), RefSeq
  refGene and chromosome-size files for the assembly, and need standardized TSS-centered
  meta-plots or heatmaps to visualize enrichment around transcription start sites.
when_to_use_negative:
- Input BAM still contains unaligned reads — SeqCode requires these to be excluded
  first
- Analysis targets genome-wide bigwig profile export — generation of genome-wide profiles
  in bigwig format is not supported and deferred to future versions
- Goal is peak calling or read counting rather than TSS-centered signal visualization
  — use other SeqCode applications or tools like MACS
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0092
tools:
- name: SeqCode
  role: Produces the TSS meta-plot (produceTSSplots) and heatmap (produceTSSmaps)
    from BAM reads, annotations, and gene lists; internally renders plots with R
  repo: https://github.com/eblancoga/seqcode
- name: SAMtools
  role: Integrated C libraries used internally by SeqCode to read BAM/SAM files
  repo: https://github.com/eblancoga/seqcode
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-021-98889-7
    title: Productive visualization of high-throughput sequencing data using the SeqCode
      open portable platform
  source_content_hash: sha256:0df5bd5bffcd96d72a67c320e1fa0181efd5f3600ad8210e230b022a3bb10cbb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eblancoga/seqcode
  tools:
  - SAMtools
  - SeqCode
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# chipseq-tss-meta-plot-generation

## Summary

Generate averaged TSS meta-plots and TSS heatmaps from ChIP-seq BAM files using SeqCode's produceTSSplots and produceTSSmaps, relying on RefSeq annotations and UCSC chromosome-size files, with the -w window option tuning graphical resolution without altering biological results.

## When to use

Use when you have a mapped ChIP-seq BAM (e.g. H3K4me3 in mESC), RefSeq refGene and chromosome-size files for the assembly, and need standardized TSS-centered meta-plots or heatmaps to visualize enrichment around transcription start sites.

## When NOT to use

- Input BAM still contains unaligned reads — SeqCode requires these to be excluded first
- Analysis targets genome-wide bigwig profile export — generation of genome-wide profiles in bigwig format is not supported and deferred to future versions
- Goal is peak calling or read counting rather than TSS-centered signal visualization — use other SeqCode applications or tools like MACS

## Inputs

- BAM file of mapped ChIP-seq reads with unaligned reads excluded (e.g. H3K4me3_chr10.bam)
- UCSC chromosome-size file (ChromInfo.txt)
- RefSeq refGene.txt transcript annotation file for the assembly
- Target gene list (e.g. genes_chr10.txt)

## Outputs

- Averaged TSS meta-plot (PDF) from produceTSSplots over a 5000 bp TSS window
- TSS heatmap (PlotHEATmap PDF, e.g. PlotHEATmap_test_1_5000.pdf) from produceTSSmaps
- Output folder containing R script (Rscript_produceTSSmaps.txt) and data files

## How to apply

Retrieve the raw ChIP-seq data (e.g. SRX367147 from NCBI GEO), map reads to the mouse genome (mm9/mm10) with Bowtie, and exclude unaligned reads from the BAM since SeqCode commands require cleaned BAM input. Prepare a UCSC chromInfo.txt chromosome-size file, the RefSeq refGene.txt transcript annotations, and a gene list (e.g. genes_chr10.txt) for the chosen assembly. Run bin/produceTSSplots with these inputs over a 5000 bp window around the TSS to produce the averaged TSS meta-plot, and bin/produceTSSmaps for the heatmap; use -w 1000 to lower heatmap resolution when memory is constrained. Execute the R script stored in the output folder (R CMD BATCH Rscript_produceTSSmaps.txt) to render the final PDF plots. The -w option affects only graphical quality and memory requirements, not the biological signal, so coarse windows remain biologically interpretable.

## Related tools

- **SeqCode** (Produces the TSS meta-plot (produceTSSplots) and heatmap (produceTSSmaps) from BAM reads, annotations, and gene lists; internally renders plots with R) — https://github.com/eblancoga/seqcode
- **SAMtools** (Integrated C libraries used internally by SeqCode to read BAM/SAM files) — https://github.com/eblancoga/seqcode

## Examples

```
bin/produceTSSmaps tests/inputs/ChromInfo.txt tests/inputs/refGene.txt tests/inputs/H3K4me3_chr10.bam tests/inputs/genes_chr10.txt test_1 5000
```

## Evaluation signals

- A PlotHEATmap PDF is produced for every window setting (e.g. PlotHEATmap_test_1_5000.pdf and PlotHEATmap_test_2_5000.pdf)
- Signal patterns across default (-w default) and lower-resolution (-w 1000) heatmaps remain biologically consistent, as in Supp. Fig. 6
- Coarser windows show only reduced graphical quality, not altered TSS enrichment shape
- The R script Rscript_produceTSSmaps.txt executes in the output folder and completes R CMD BATCH successfully
- tests/test_produceTSSmaps.pl terminates with 'Successful termination: [DONE]' within reported runtime (~46 s)

## Limitations

- Lower -w resolutions reduce graphical quality of the heatmap (though biology stays consistent)
- Generation of genome-wide profiles in bigwig format is not supported and deferred to future versions
- The SeqCode web version is intended as a teaching resource; the command-line release has no size limits but requires local UNIX (Linux/macOS) execution

## Evidence

- [other] produceTSSplots with the chromosome file, refGene file, BAM, and gene list over a 5000 bp window around the TSS to produce the averaged TSS meta-plot: [evidence span withheld — anchor status: not_found]
- [methods] Unaligned reads must be excluded from BAM files: "Unaligned reads must be excluded from the resulting BAM files that will be processed by SeqCode commands."
- [results] Outputs remained biologically consistent across window resolutions: [evidence span withheld — anchor status: not_found]
- [readme] Graphical outputs are produced by running R scripts stored in the resulting folder: "Graphical outputs are produced by running R scripts that will be stored in the resulting folder as well."
- [readme] Checking output file 1: tests/outputs/produceTSSmaps/test_1_TSSmap_5000/PlotHEATmap_test_1_5000.pdf: "%%%% Checking output file 1: tests/outputs/produceTSSmaps/test_1_TSSmap_5000/PlotHEATmap_test_1_5000.pdf	[OK]"
