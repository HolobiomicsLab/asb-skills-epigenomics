---
name: mnase-seq-occupancy-modeling
description: Use when paired-end MNase-seq (or ATAC-seq) data are available and you
  need genome-wide, single-nucleotide-resolution probabilistic occupancy scores for
  nucleosomes and DNA-binding factors, e.g. ORC at annotated ACS replication origins.
when_to_use_negative:
- The goal is simple peak calling / footprinting rather than joint probabilistic occupancy
  — the paper compares RoboCOP against MACS2 peak/footprint identification followed
  by FIMO TF-labeling and shows it outperforms that pipeline, so use RoboCOP, not
  that alternative, when occupancy probabilities are needed.
- Only single-end accessibility data or no nucleotide sequence are available — RoboCOP
  integrates paired-end MNase-seq/ATAC-seq data with nucleotide sequence as inputs.
- Analysis targets organisms or datasets outside the demonstrated yeast (S. cerevisiae)
  MNase-seq/ATAC-seq setting without recalibration.
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_0622
tools:
- name: RoboCOP
  role: multivariate HMM that jointly computes genome-wide probabilistic nucleosome
    and DBF occupancy from MNase-seq data plus nucleotide sequence; used once per
    condition (wild-type and orc1-161)
  repo: https://github.com/HarteminkLab/RoboCOP
- name: COMPETE
  role: earlier probabilistic DBF occupancy method from the same lab; RoboCOP increases
    TF binding site prediction accuracy over COMPETE
- name: NucleoATAC
  role: comparison method; RoboCOP (ATAC-seq mode) positions many more nucleosomes
    than NucleoATAC on the same data
- name: MACS2
  role: alternative peak/footprint identification baseline used for comparison, followed
    by FIMO TF-labeling
- name: FIMO
  role: TF-labeling step in the alternative MACS2-based baseline against which RoboCOP
    is benchmarked
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkab553
    title: 'RoboCOP: jointly computing chromatin occupancy profiles for numerous factors
      from chromatin accessibility data'
  source_content_hash: sha256:92a727795a564f20c7cc3ebf444245adf2741e52e5275caef7de89f17610e2f9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HarteminkLab/RoboCOP
  tools:
  - COMPETE
  - FIMO
  - MACS2
  - NucleoATAC
  - RoboCOP
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# mnase-seq-occupancy-modeling

## Summary

Apply RoboCOP, a multivariate HMM that integrates paired-end MNase-seq accessibility data with nucleotide sequence, to compute genome-wide probabilistic occupancy profiles of nucleosomes and DNA-binding factors in yeast, including condition contrasts such as wild-type versus orc1-161 mutant under G2 arrest.

## When to use

Use when paired-end MNase-seq (or ATAC-seq) data are available and you need genome-wide, single-nucleotide-resolution probabilistic occupancy scores for nucleosomes and DNA-binding factors, e.g. ORC at annotated ACS replication origins.

## When NOT to use

- The goal is simple peak calling / footprinting rather than joint probabilistic occupancy — the paper compares RoboCOP against MACS2 peak/footprint identification followed by FIMO TF-labeling and shows it outperforms that pipeline, so use RoboCOP, not that alternative, when occupancy probabilities are needed.
- Only single-end accessibility data or no nucleotide sequence are available — RoboCOP integrates paired-end MNase-seq/ATAC-seq data with nucleotide sequence as inputs.
- Analysis targets organisms or datasets outside the demonstrated yeast (S. cerevisiae) MNase-seq/ATAC-seq setting without recalibration.

## Inputs

- paired-end MNase-seq datasets for each condition (e.g. SRA accession SRP041314: wild-type and orc1-161 G2-arrested yeast)
- nucleotide (reference genome) sequence
- annotated ACS replication-origin sites and/or the most prevalent ORC binding motif from ORC ChIP-seq peaks (for origin-targeted evaluation)
- configuration file with the cshared absolute path to librobocop.so

## Outputs

- genome-wide probabilistic occupancy profile of nucleosomes and DNA-binding factors per condition
- per-origin ORC posterior probabilities with flanking nucleosome positions for each condition
- condition-contrast figure (e.g. supplementary_figure_s19.png reproducing wild-type vs orc1-161 ORC occupancy)

## How to apply

Clone the HarteminkLab RoboCOP repository and, from the robocop directory, run 'bash gccCompile' to generate the shared library librobocop.so, then register its absolute path in the configuration file via the cshared entry (modeled on analysis/config_example.ini). Create the conda environment from robocop-spec-file.txt and download the paired-end MNase-seq fastq data (e.g. SRA accession SRP041314 for wild-type and orc1-161 G2-arrested yeast). Run RoboCOP's multivariate HMM once per condition, supplying the MNase-seq data together with nucleotide sequence, to infer the genome-wide probabilistic occupancy landscape of nucleosomes and DBFs. For origin-focused analyses, extract ORC posterior probabilities at annotated ACS origins plus flanking nucleosome positions in each condition and contrast them; MNase-seq is preferred over ATAC-seq because fragment distributions and MNase exonuclease activity better reflect DBF sizes.

## Related tools

- **RoboCOP** (multivariate HMM that jointly computes genome-wide probabilistic nucleosome and DBF occupancy from MNase-seq data plus nucleotide sequence; used once per condition (wild-type and orc1-161)) — https://github.com/HarteminkLab/RoboCOP
- **COMPETE** (earlier probabilistic DBF occupancy method from the same lab; RoboCOP increases TF binding site prediction accuracy over COMPETE)
- **NucleoATAC** (comparison method; RoboCOP (ATAC-seq mode) positions many more nucleosomes than NucleoATAC on the same data)
- **MACS2** (alternative peak/footprint identification baseline used for comparison, followed by FIMO TF-labeling)
- **FIMO** (TF-labeling step in the alternative MACS2-based baseline against which RoboCOP is benchmarked)

## Examples

```
git clone https://github.com/HarteminkLab/RoboCOP.git; cd RoboCOP/robocop && bash gccCompile; echo 'cshared=/home/myhome/RoboCOP/pkg/robocop/librobocop.so' >> ../analysis/config.ini
```

## Evaluation signals

- librobocop.so is built by gccCompile and its absolute path resolves in the config.ini cshared entry (e.g. cshared=/home/myhome/RoboCOP/pkg/robocop/librobocop.so).
- RoboCOP runs to completion on each condition's MNase-seq data and emits a genome-wide posterior occupancy profile for nucleosomes and 150 TFs.
- Reproduced origin contrast (e.g. supplementary_figure_s19.png) shows strong ORC posterior probability at ACS origins in wild type and absence in the orc1-161 mutant.
- Flanking nucleosomes are spaced farther apart at origins where ORC is bound in wild type, matching the expected occluded-origin nucleosome architecture.
- Predictions validate better than MACS2+FIMO for TF sites and better than NucleoATAC for nucleosome positioning on the same data.

## Limitations

- No tool versions are specified for RoboCOP, COMPETE, NucleoATAC, MACS2, or FIMO; environment must be recreated from robocop-spec-file.txt.
- Accuracy and comprehensiveness of the occupancy profiles improve with deeper sequencing; sequencing depth requirements are not quantified.
- Prior information about DBF transition probabilities is not incorporated (deferred to future work).
- MNase-seq yields markedly better performance than ATAC-seq; ATAC-seq results are comparatively weaker for this purpose.

## Evidence
