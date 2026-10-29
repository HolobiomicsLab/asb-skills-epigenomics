---
name: multivariate-hmm-chromatin-inference
description: Use when paired-end MNase-seq (or ATAC-seq) reads and a reference genome
  sequence are available and the goal is a probabilistic occupancy landscape of nucleosomes
  and transcription factors across the genome.
when_to_use_negative:
- The goal is peak calling of a single factor rather than joint probabilistic occupancy
  of hundreds of DBFs and nucleosomes — the paper shows MACS2 peak/footprint identification
  followed by FIMO TF-labeling is the alternative (lower-performing) approach
- Only ATAC-seq data are available and MNase-seq could be used instead — the article
  reports markedly better performance with MNase-seq for two primary reasons (genome-wide
  fragment distribution and MNase exonuclease activity reflecting DBF sizes)
- No nucleotide sequence input can be supplied — RoboCOP integrates accessibility
  data with nucleotide sequence as joint input
edam_operation: http://edamontology.org/operation_0432
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0749
tools:
- name: RoboCOP
  role: Primary multivariate HMM model that integrates MNase-seq/ATAC-seq accessibility
    with nucleotide sequence to jointly compute probabilistic occupancy scores of
    nucleosomes and TFs
  repo: https://github.com/HarteminkLab/RoboCOP
- name: gccCompile
  role: Build script run via bash inside the robocop directory to generate the librobocop.so
    shared library required by the pipeline
  repo: https://github.com/HarteminkLab/RoboCOP
- name: DynaCOP
  role: Related vignette/workflow (example_dynacop.ipynb) for dynamic chromatin occupancy
    analysis within the same repository
  repo: https://github.com/HarteminkLab/RoboCOP/blob/master/analysis/example_dynacop.ipynb
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkab553
    title: 'RoboCOP: jointly computing chromatin occupancy profiles for numerous factors
      from chromatin accessibility data'
  source_content_hash: sha256:0128552560df372ac67363d8fe8e441046262a3578e12c6f45d59fa3e8cc4633
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HarteminkLab/RoboCOP
  tools:
  - DynaCOP
  - RoboCOP
  - gccCompile
  edam_operation: http://edamontology.org/operation_0432
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# multivariate-hmm-chromatin-inference

## Summary

Infers genome-wide probabilistic occupancy profiles of nucleosomes and DNA-binding factors by running RoboCOP's multivariate HMM, which integrates chromatin accessibility data (paired-end MNase-seq or ATAC-seq) with nucleotide sequence at single-nucleotide resolution.

## When to use

Use when paired-end MNase-seq (or ATAC-seq) reads and a reference genome sequence are available and the goal is a probabilistic occupancy landscape of nucleosomes and transcription factors across the genome.

## When NOT to use

- The goal is peak calling of a single factor rather than joint probabilistic occupancy of hundreds of DBFs and nucleosomes — the paper shows MACS2 peak/footprint identification followed by FIMO TF-labeling is the alternative (lower-performing) approach
- Only ATAC-seq data are available and MNase-seq could be used instead — the article reports markedly better performance with MNase-seq for two primary reasons (genome-wide fragment distribution and MNase exonuclease activity reflecting DBF sizes)
- No nucleotide sequence input can be supplied — RoboCOP integrates accessibility data with nucleotide sequence as joint input

## Inputs

- Paired-end MNase-seq data (e.g. SRA accession SRP041314 for wild-type and orc1-161 G2-arrested yeast)
- ATAC-seq data (e.g. GEO accession GSE66386)
- Nucleotide (reference genome) sequence
- Configuration file with cshared path to librobocop.so (modeled on analysis/config_example.ini)

## Outputs

- Genome-wide probabilistic occupancy profile of nucleosomes and DNA-binding factors at single-nucleotide resolution
- Factor-specific posterior probabilities at genomic loci of interest (e.g. ORC posterior probability at ACS replication origins)
- Figure artifacts (e.g. supplementary_figure_s19.png comparing wild-type vs mutant ORC occupancy)

## How to apply

Clone the HarteminkLab/RoboCOP repository, create the conda environment from robocop-spec-file.txt, and build the shared library with 'bash gccCompile' inside the robocop directory to produce librobocop.so. Register the absolute path of librobocop.so in the configuration file under the cshared entry, following the structure of analysis/config_example.ini. Run RoboCOP's multivariate HMM on the accessibility data together with nucleotide sequence to compute the probabilistic occupancy profile of nucleosomes and DBFs; note that MNase-seq yields markedly better performance than ATAC-seq because fragment distributions reflect DBF sizes. For multi-condition comparisons (e.g. wild-type versus orc1-161 G2-arrested yeast from SRA accession SRP041314), run the inference separately on each condition and extract factor-specific posterior probabilities (e.g. ORC at annotated ACS origins) for contrast and visualization.

## Related tools

- **RoboCOP** (Primary multivariate HMM model that integrates MNase-seq/ATAC-seq accessibility with nucleotide sequence to jointly compute probabilistic occupancy scores of nucleosomes and TFs) — https://github.com/HarteminkLab/RoboCOP
- **gccCompile** (Build script run via bash inside the robocop directory to generate the librobocop.so shared library required by the pipeline) — https://github.com/HarteminkLab/RoboCOP
- **DynaCOP** (Related vignette/workflow (example_dynacop.ipynb) for dynamic chromatin occupancy analysis within the same repository) — https://github.com/HarteminkLab/RoboCOP/blob/master/analysis/example_dynacop.ipynb

## Examples

```
git clone https://github.com/HarteminkLab/RoboCOP.git; cd RoboCOP/robocop; bash gccCompile; ls "`pwd`/librobocop.so"
```

## Evaluation signals

- librobocop.so exists at the path registered in the cshared entry of the configuration file (verify with ls `pwd`/librobocop.so)
- Reproduction of Supplementary Figure S19 shows strong ORC posterior probability at annotated ACS origins in wild type but absence in the orc1-161 mutant
- Flanking nucleosomes appear spaced farther apart at origins where ORC is bound in the wild-type occupancy profile
- TF binding site and nucleosome positioning predictions outperform the MACS2-plus-FIMO and NucleoATAC baselines reported in the paper
- Output is a probabilistic (posterior) occupancy score at every genomic position, not a binary peak call

## Limitations

- No version numbers are stated for RoboCOP or companion tools; pin the environment via robocop-spec-file.txt conda spec
- Accuracy improves with deeper sequencing; shallow MNase-seq data will yield less accurate occupancy profiles
- Prior information about DBF transition probabilities is not incorporated and is deferred to future work
- RoboCOP's performance is markedly better with MNase-seq than ATAC-seq, so ATAC-seq-only inputs are expected to underperform

## Evidence
