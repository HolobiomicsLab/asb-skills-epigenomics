---
name: genomic-region-promoter-intergenic-intragenic-assignment
description: Use when BED peak lists (e.g. MACS-called H3K4me3, H3K27me3, or bivalent
  peaks) must be classified into promoter, intergenic, or intragenic features using
  a RefSeq refGene annotation for a genome assembly such as mouse mm9.
when_to_use_negative:
- The goal is peak-level signal quantification or read counting over regions rather
  than feature-category annotation (use SeqCode recoverChIPlevels/counting instead).
- Only a genome browser-style track profile (e.g. bigwig) is needed — SeqCode defers
  bigwig generation to future versions.
- The peak coordinates and the refGene annotation correspond to different assemblies
  (e.g. mm9 peaks vs mm10 refGene), which would corrupt promoter assignments.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: SeqCode
  role: genomeDistribution command classifies BED peak intervals into promoter/intergenic/intragenic
    features and emits R scripts for distribution charts
  repo: https://github.com/eblancoga/seqcode
- name: MACS
  role: upstream peak caller producing the BED peak lists that genomeDistribution
    annotates
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-021-98889-7
    title: Productive visualization of high-throughput sequencing data using the SeqCode
      open portable platform
  source_content_hash: sha256:2e22088375f99ed4c335ac6b11c8191caeff284892c2bc7adcbdd8cc0f52e0ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eblancoga/seqcode
  tools:
  - MACS
  - SeqCode
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# genomic-region-promoter-intergenic-intragenic-assignment

## Summary

Assigns every interval in a BED-formatted peak list (e.g. MACS-called ChIP-seq peaks) to a genomic feature category — promoter, intergenic, or intragenic — using the SeqCode genomeDistribution command against a RefSeq refGene annotation and a chromosome-size file for a specific genome assembly, enabling feature-composition charts such as the promoter enrichment of bivalent H3K4me3+H3K27me3 peaks.

## When to use

Use when BED peak lists (e.g. MACS-called H3K4me3, H3K27me3, or bivalent peaks) must be classified into promoter, intergenic, or intragenic features using a RefSeq refGene annotation for a genome assembly such as mouse mm9.

## When NOT to use

- The goal is peak-level signal quantification or read counting over regions rather than feature-category annotation (use SeqCode recoverChIPlevels/counting instead).
- Only a genome browser-style track profile (e.g. bigwig) is needed — SeqCode defers bigwig generation to future versions.
- The peak coordinates and the refGene annotation correspond to different assemblies (e.g. mm9 peaks vs mm10 refGene), which would corrupt promoter assignments.

## Inputs

- BED-formatted peak lists (e.g. bivalent H3K4me3+H3K27me3, H3K4me3-only, H3K27me3-only peaks from MACS peak calling)
- Chromosome-size file (chromInfo.txt) for the target genome assembly (e.g. mouse mm9 from UCSC)
- RefSeq gene transcript annotation file (refGene.txt) for the same assembly

## Outputs

- Per-peak assignment to promoter, intergenic, or intragenic genomic features
- Aggregated per-class feature-count files
- Pie-chart plots of genome-feature distribution plus the R scripts that generate them, written into an output folder

## How to apply

Load the BED-formatted peak classes produced upstream by MACS peak calling, then run the SeqCode genomeDistribution command once per peak list, supplying the chromInfo.txt (chromosome sizes) and refGene.txt (RefSeq transcript annotations) files retrieved for the target assembly (e.g. mouse mm9 from UCSC). genomeDistribution uses the RefSeq annotations to classify every peak as promoter, intergenic, or intragenic and emits an output folder containing the per-feature counts and the R scripts that generate the pie-chart plots of the distributions. Aggregate the per-class counts across the bivalent, H3K4me3-only, and H3K27me3-only peak lists to compare feature composition. Rationale: the unit of information in SeqCode is the RefSeq annotation file, so the assignment quality depends directly on the refGene file matching the assembly used for peak coordinates. Validate by confirming the charts reproduce the expected biological pattern (bivalent peaks preferentially in gene promoters, minor fraction intergenic, as in Fig. 5b of the SeqCode paper).

## Related tools

- **SeqCode** (genomeDistribution command classifies BED peak intervals into promoter/intergenic/intragenic features and emits R scripts for distribution charts) — https://github.com/eblancoga/seqcode
- **MACS** (upstream peak caller producing the BED peak lists that genomeDistribution annotates)

## Examples

```
bin/genomeDistribution ChromInfo.txt refGene.txt bivalent_peaks.bed bivalent_dist; ./tests/test_genomeDistribution.pl -v
```

## Evaluation signals

- Every input peak is assigned to exactly one of the three feature categories (promoter, intergenic, intragenic) and per-category counts sum to the input peak count.
- The output folder contains the distribution data files, the pie-chart plot(s), and the R scripts that generated them.
- The annotated distributions reproduce the Fig. 5b pattern: bivalent (H3K4me3+H3K27me3) peaks are preferentially found in gene promoters relative to single-mark peak classes, with only a minor intergenic fraction.
- The chromInfo.txt and refGene.txt files correspond to the same assembly (e.g. mm9) as the peak coordinates; the SeqCode tests/ folder (test_genomeDistribution.pl) terminates with 'Successful termination: [DONE]'.

## Limitations

- Assignment quality depends entirely on the RefSeq annotation file; SeqCode relies on RefSeq and handles only assemblies available from that consortium.
- The SeqCode web interface is intended as a teaching resource with input size restrictions; unconstrained analysis requires the local command-line build.
- Feature categories are limited to the promoter/intergenic/intragenic scheme emitted by genomeDistribution; finer categorizations are not produced by this command.

## Evidence

- [readme] Calculate the distribution of genome features from a set of genomic coordinates: "Calculate the distribution of genome features from a set of genomic coordinates"
- [readme] Two required reference files per genome: "Most programs require the user to provide two files for the particular genome that is being analyzed: the list of chromosomes (and their sizes) and the RefSeq gene transcript annotations."
- [readme] R scripts stored with outputs: "The output of the program is usually stored into an output folder in which the user will find the resulting plots, the scripts to generate the plots in R (if any), and the data files."
- [methods] RefSeq annotation as basis: "SeqCode relies on the RefSeq reference annotations and is able to deal with the genome and assembly release of every organism that is available from this consortium."
