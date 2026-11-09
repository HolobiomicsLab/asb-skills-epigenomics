---
name: bisulfite-read-to-reference-alignment-offset-mapping
description: Use when computing methylation levels from trimmed BSP sequencing reads
  (.ab1 chromatograms) requires converting each read cytosine position into the corresponding
  CpG coordinate on the reference .fasta template.
when_to_use_negative:
- Input data are not bisulfite sequencing PCR products (e.g. whole-genome bisulfite
  sequencing FASTQ alignments), for which a read-mapper rather than ABSP's template-offset
  approach is appropriate.
- The goal is only genomic visualization of methylation plots; the reference genome
  is only needed for plots, not for computing methylation percentages.
- The .ab1 file has been trimmed so short that no CpG sites are covered — the analysis
  must instead be stopped and trimming parameters adjusted (e.g. th_quality_error
  lowered from 0.001).
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_3674
tools:
- name: ABSP
  role: R-based tool performing the local pair-wise alignment of trimmed reads to
    the reference template and applying the Sstart/Pstart offset to map cytosine positions
    onto CpG coordinates
  repo: https://github.com/ABSP-methylation-tool/ABSP
- name: R
  role: Runtime environment in which ABSP and its individual-analysis workflow execute
  repo: https://www.r-project.org/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad008
    title: 'ABSP: an automated R tool to efficiently analyze region-specific CpG methylation
      from bisulfite sequencing PCR'
  source_content_hash: sha256:98c89ed1920365851df83549809528e06589384fcf0833df565c644167269d78
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ABSP-methylation-tool/ABSP
  tools:
  - ABSP
  - R
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_3674
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# bisulfite-read-to-reference-alignment-offset-mapping

## Summary

Maps cytosine positions from bisulfite sequencing reads back onto CpG coordinates of the reference DNA using a local pair-wise alignment offset, as implemented in the ABSP R tool for direct-BSP and cloning-BSP .ab1 chromatogram data.

## When to use

Use when computing methylation levels from trimmed BSP sequencing reads (.ab1 chromatograms) requires converting each read cytosine position into the corresponding CpG coordinate on the reference .fasta template.

## When NOT to use

- Input data are not bisulfite sequencing PCR products (e.g. whole-genome bisulfite sequencing FASTQ alignments), for which a read-mapper rather than ABSP's template-offset approach is appropriate.
- The goal is only genomic visualization of methylation plots; the reference genome is only needed for plots, not for computing methylation percentages.
- The .ab1 file has been trimmed so short that no CpG sites are covered — the analysis must instead be stopped and trimming parameters adjusted (e.g. th_quality_error lowered from 0.001).

## Inputs

- Reference DNA sequence in .fasta format (with coordinates in the header, e.g. chr1:1-200)
- Raw sequencing trace files (.ab1 chromatograms) from bisulfite sequencing PCR
- Trimmed read sequences from ABSP's quality-score and primary peak-ratio trimming step

## Outputs

- Methylation table with per-CpG genomic coordinates and methylated/unmethylated counts, written to the 'results' folder
- CpG methylation levels per sample feeding the individual analysis .html report

## How to apply

Load the reference sequence from the CDH1-style .fasta file and the raw chromatograms from the .ab1 files, then scan the reference to enumerate all CpG dinucleotide positions. Align each trimmed read to the reference template with a local pair-wise alignment and select the sense or antisense template that yields the longest alignment. Convert each read cytosine position to the template coordinate using the offset derived from the reference start/end coordinates: cytosine position = cytosine position on template − Sstart + Pstart (e.g. 100 − 30 + 22 = 92 on the forward strand). Tally methylated versus unmethylated cytosines at each covered CpG to compute methylation percentages and write the coordinate-annotated methylation table to the results folder.

## Related tools

- **ABSP** (R-based tool performing the local pair-wise alignment of trimmed reads to the reference template and applying the Sstart/Pstart offset to map cytosine positions onto CpG coordinates) — https://github.com/ABSP-methylation-tool/ABSP
- **R** (Runtime environment in which ABSP and its individual-analysis workflow execute) — https://www.r-project.org/

## Examples

```
ABSP::individual_analysis(reference_fasta = 'CDH1.fasta', sequencing_files = c('CDH1_sample.ab1'), th_quality_error = 0.001)
```

## Evaluation signals

- Spot-check the offset arithmetic: a read cytosine at position 100 with Sstart=30 and Pstart=22 must map to template coordinate 92 on the forward strand.
- The generated methylation table must contain at least one covered CpG site; if no CpG is covered, the run should abort with 'Analysis has been stopped as no CpG sites were found covered by sequencing results.'
- For antisense-template alignments, the longest of the sense/antisense local alignments must be the one selected before computing offsets.
- Mapped CpG coordinates must fall within the reference interval declared in the .fasta header (e.g. chr1:1-200 for a 200 bp sequence).
- Each read cytosine must map to a reference position that is part of a CpG dinucleotide in the reference scan.

## Limitations

- With an arbitrary reference genome, displayed coordinates in tables and plots are arbitrary and must not be taken into consideration; use 'CpG numbers' or 'None' position labels in Grouped Analysis.
- If trimming yields a very short final sequence, only a few CpG sites may be covered, or the mapping stage aborts with the 'no CpG sites were found covered' error.
- Over-aggressive Phred-quality trimming (default th_quality_error = 0.001, Phred 30) can remove regions containing CpGs before mapping.

## Evidence

- [methods] ABSP maps cytosine positions from the template DNA onto the sequencing read using the local pair-wise alignment offset: cytosine position = cytosine position on template − SstartF + PstartF (e.g. 100 − 30 + 22 = 92 on the forward strand).: [evidence span withheld — anchor status: not_found]
- [other] Align each trimmed read to the reference template with a local pair-wise alignment and choose between sense and antisense templates based on the longest alignment.: [evidence span withheld — anchor status: not_found]
- [readme] Tally methylated versus unmethylated cytosines at each CpG to compute methylation levels and write the coordinate-annotated methylation table to the results folder.: "Proceed to run the analysis to compute the CpG methylation levels of your sample"
- [readme] the analysis may be aborted with the error message: "Error: Analysis has been stopped as no CpG sites were found covered by sequencing results.": "Error: Analysis has been stopped as no CpG sites were found covered by sequencing results."
- [readme] By setting your coordinates to "chr1:1-XXX" (where "XXX" represents the length of your sequence), the sequence will surely correspond to a series of N nucleotides, which will be displayed in grey in genomic plots.: "By setting your coordinates to "chr1:1-XXX" (where "XXX" represents the length of your sequence), the sequence will surely correspond to a series of N nucleotides"
