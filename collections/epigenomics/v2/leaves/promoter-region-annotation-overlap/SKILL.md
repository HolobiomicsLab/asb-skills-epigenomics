---
name: promoter-region-annotation-overlap
description: Use when methylation callsets (e.g. Bismark .cov output) need unique
  region identifiers per promoter or CGI before running DXM, or when restricting deconvolution
  to specific genomic compartments like TSS +/-5kb promoters.
when_to_use_negative:
- Regions of interest already carry unique region names in the methylation file, making
  re-annotation redundant
- The analysis targets whole-genome or chromosome-wide methylation profiles rather
  than user-defined regions such as promoters or CGIs — DXM deconvolves over user-defined
  regions, and a global profile across an entire chromosome would be difficult given
  assembly gaps and repetitive sequences
- The annotation genome build does not match the methylation data's genome version
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
tools:
- name: bedtools
  role: Performs the intersect -wo overlap between the methylation bed and promoter/CGI
    annotation, with awk used to project the regionName column into the DXM input
    format
- name: DXM
  role: 'Downstream consumer: requires the regionName-annotated six-column BED-like
    file so it can compute across all CpGs of each uniquely named region'
  repo: https://github.com/edwardslab-wustl/dxm
- name: Bismark
  role: Source of the .cov methylation callsets (chr pos1 pos2 methylation meth_cov
    unmeth_cov) that are converted to BED-like format before overlapping with annotations
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkab516
    title: Determining subpopulation methylation profiles from bisulfite sequencing
      data of heterogeneous samples using DXM
  source_content_hash: sha256:f7e289c6e6fb861e5467b7106c415ebb42cf0e6639470ba86dbc21f9661525f2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edwardslab-wustl/dxm
  tools:
  - Bismark
  - DXM
  - bedtools
  - numpy
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# promoter-region-annotation-overlap

## Summary

Annotate per-CpG methylation calls with unique region names by overlapping them against promoter coordinates (e.g. TSS +/-5kb for refSeq hg19 genes or CGIs) using bedtools intersect, producing the regionName column required by DXM input files.

## When to use

Use when methylation callsets (e.g. Bismark .cov output) need unique region identifiers per promoter or CGI before running DXM, or when restricting deconvolution to specific genomic compartments like TSS +/-5kb promoters.

## When NOT to use

- Regions of interest already carry unique region names in the methylation file, making re-annotation redundant
- The analysis targets whole-genome or chromosome-wide methylation profiles rather than user-defined regions such as promoters or CGIs — DXM deconvolves over user-defined regions, and a global profile across an entire chromosome would be difficult given assembly gaps and repetitive sequences
- The annotation genome build does not match the methylation data's genome version

## Inputs

- Tab-delimited methylation BED-like file (chromosome, position1, position2, methylation_level, coverage), e.g. converted from Bismark .cov
- Promoter/CGI annotation bed file (e.g. refGene_hg19_10kb_fixed.txt.gz for TSS +/-5kb regions on hg19, or cgi.bed from UCSC Table Browser)

## Outputs

- DXM-format input file with six columns: chromosome, position1, position2, regionName, fractionalMethylation, coverage (e.g. dxm_in.bed)

## How to apply

Prepare the methylation data as a tab-delimited BED-like file (chr, position1, position2, fractionalMethylation, coverage), collapsing data across strands and filtering coverage to the maximum expected sequencing coverage. Obtain a regions bed file — either a CGI bed downloaded from the UCSC Genome Browser Table Browser, or refGene_hg19_10kb_fixed.txt.gz from the DXM example_data directory for TSS +/-5kb promoter windows on hg19. Run bedtools intersect with the -wo flag between methylation.bed and the regions file, then use awk to print the columns in DXM's six-column input order (chr, position1, position2, regionName, fractionalMethylation, coverage), ensuring every promoter receives a unique region name. The awk column indices may need adjustment depending on which annotation bed is used.

## Related tools

- **bedtools** (Performs the intersect -wo overlap between the methylation bed and promoter/CGI annotation, with awk used to project the regionName column into the DXM input format)
- **DXM** (Downstream consumer: requires the regionName-annotated six-column BED-like file so it can compute across all CpGs of each uniquely named region) — https://github.com/edwardslab-wustl/dxm
- **Bismark** (Source of the .cov methylation callsets (chr pos1 pos2 methylation meth_cov unmeth_cov) that are converted to BED-like format before overlapping with annotations)

## Examples

```
bedtools intersect -wo -a methylation.bed -b refGene_hg19_10kb_fixed.txt | awk '{print $1"\t"$2"\t"$3"\t"$9"\t"$4"\t"$5;}' > dxm_in.bed
```

## Evaluation signals

- Output file has exactly six tab-delimited columns in DXM order: chr, position1, position2, regionName, fractionalMethylation, coverage
- Every distinct promoter/CGI in the annotation that overlaps a CpG appears with a unique regionName; duplicated region names across different promoters would break DXM's per-region computation
- fractionalMethylation values lie between 0 and 1 (divide by 100 if the aligner outputs percentages)
- Coverage values are integers below the maximum expected sequencing coverage that will be passed via the -c flag of dxm_solveMethylation
- bedtools intersect -wo output rows correspond to CpG–region overlaps; CpGs falling outside all annotated regions should be absent from the DXM input

## Limitations

- As regions get larger and include more CpGs, the number of CpGs that must change in methylation to be solved as a distinct profile increases — region definition affects DXM sensitivity
- The awk column-printing statement must be adjusted when swapping in different annotation bed files, since column layouts differ
- DXM does not explicitly use the chromosome or position2 columns (it computes relative coordinates internally), but they are still required in the file

## Evidence

- [readme] Overlap the data and filter the correct columns: bedtools intersect -wo -a methylation.bed -b cgi.bed | awk '{print $1"\t"$2"\t"$3"\t"$9"\t"$4"\t"$5;}' > dxm_in.bed: "Overlap the data and filter the correct columns: 

    bedtools intersect -wo -a methylation.bed -b cgi.bed | awk '{print $1"\t"$2"\t"$3"\t"$9"\t"$4"\t"$5;}' > dxm_in.bed"
- [readme] DXM computes across all CpGs of a given region, and thus, unique region names should be generated for each region of interest.: "DXM computes across all CpGs of a given region, and thus, unique region names should be generated for each region of interest."
- [readme] if you want to use the regions +/- 5kb around the TSS for refSeq annotated genes (hg19) you can download and unzip the file refGene_hg19_10kb_fixed.txt.gz from the example_data directory.: "if you want to use the regions +/- 5kb around the TSS for refSeq annotated genes (hg19) you can download and unzip the file refGene_hg19_10kb_fixed.txt.gz from the example_data directory."
- [discussion] DXM deconvolves methylation profiles over user-defined regions.: "DXM deconvolves methylation profiles over user-defined regions."
- [methods] bedtools intersect -wo -a methylation.bed -b cgi.bed: "bedtools intersect -wo -a methylation.bed -b cgi.bed"
