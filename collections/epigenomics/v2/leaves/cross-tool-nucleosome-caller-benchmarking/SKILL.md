---
name: cross-tool-nucleosome-caller-benchmarking
description: Use when comparing nucleosome-calling tools on identical chromatin accessibility
  data — e.g., deciding whether RoboCOP run on ATAC-seq alone suffices versus NucleoATAC,
  or whether RoboCOP's MNase+ATAC profile outperforms DANPOS2 — by nucleosome-position
  AUPR on a shared validation set.
when_to_use_negative:
- No shared ground-truth annotation set (e.g., chemical-cleavage nucleosome positions)
  is available, so AUPR cannot be computed on an identical validation set and raw
  positioned-nucleosome counts would give a misleading comparison.
- The question concerns TF/DBF binding-site prediction rather than nucleosome positioning
  — that benchmark uses ChIP-chip, ChIP-exo, and ORGANIC annotations, not nucleosome-position
  AUPR.
- The accessibility data are neither MNase-seq nor ATAC-seq; the RoboCOP README states
  the model currently uses only MNase-seq or ATAC-seq.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: RoboCOP
  role: Nucleosome/DBF occupancy caller under benchmark; run in ATAC-seq-only mode
    and MNase+ATAC mode after compiling librobocop.so and setting the cshared path
    in config.ini
  repo: https://github.com/HarteminkLab/RoboCOP
- name: NucleoATAC
  role: Baseline nucleosome-positioning caller run on the same ATAC-seq data for the
    Supplementary Figure S12A comparison (RoboCOPATAC vs NucleoATAC)
- name: DANPOS2
  role: Baseline nucleosome-positioning caller run on the same MNase-seq data for
    the Supplementary Figure S12 comparison (RoboCOP_(MNase+ATAC) vs DANPOS2)
- name: GCC
  role: Compiler invoked by RoboCOP's bash gccCompile script to generate the librobocop.so
    shared library required by the RoboCOP engine
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkab553
    title: 'RoboCOP: jointly computing chromatin occupancy profiles for numerous factors
      from chromatin accessibility data'
  source_content_hash: sha256:c426d9835cd7134821b794354b9cb035d61d93d96d73b9c48c54ee2c426777c9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HarteminkLab/RoboCOP
  tools:
  - DANPOS2
  - GCC
  - NucleoATAC
  - RoboCOP
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# cross-tool-nucleosome-caller-benchmarking

## Summary

Benchmark nucleosome-positioning callers head-to-head by running RoboCOP on ATAC-seq alone and on MNase-seq+ATAC-seq against NucleoATAC (same ATAC-seq data) and DANPOS2 (same MNase-seq data), scoring all four methods by nucleosome-position AUPR against identical chemical-cleavage validation annotations. The benchmark answers whether ATAC-seq alone suffices for nucleosome calling, given that NucleoATAC reports only a small fraction (~20%) of genomic nucleosomes.

## When to use

Use when comparing nucleosome-calling tools on identical chromatin accessibility data — e.g., deciding whether RoboCOP run on ATAC-seq alone suffices versus NucleoATAC, or whether RoboCOP's MNase+ATAC profile outperforms DANPOS2 — by nucleosome-position AUPR on a shared validation set.

## When NOT to use

- No shared ground-truth annotation set (e.g., chemical-cleavage nucleosome positions) is available, so AUPR cannot be computed on an identical validation set and raw positioned-nucleosome counts would give a misleading comparison.
- The question concerns TF/DBF binding-site prediction rather than nucleosome positioning — that benchmark uses ChIP-chip, ChIP-exo, and ORGANIC annotations, not nucleosome-position AUPR.
- The accessibility data are neither MNase-seq nor ATAC-seq; the RoboCOP README states the model currently uses only MNase-seq or ATAC-seq.

## Inputs

- Paired-end ATAC-seq data for S. cerevisiae (GEO accession GSE66386)
- Paired-end MNase-seq data (GEO accession GSE153609)
- Reference nucleotide sequence of the genome (RoboCOP integrates nucleotide sequence with chromatin accessibility data)
- High-precision chemical-cleavage nucleosome annotations used as the validation ground truth
- RoboCOP configuration file (config.ini, modeled on config_example.ini) with cshared pointing to the compiled librobocop.so

## Outputs

- Genome-wide probabilistic nucleosome occupancy profiles from RoboCOP (ATAC-seq-only run and MNase+ATAC run)
- Nucleosome-position calls from NucleoATAC (same ATAC-seq data) and DANPOS2 (same MNase-seq data)
- nucleosome_aupr_scores.csv containing nucleosome-position AUPR for RoboCOP (ATAC-only and MNase+ATAC), DANPOS2, and NucleoATAC
- supplementary_figure_S12_aupr_comparison.png rendering the cross-tool AUPR comparison of Supplementary Figure S12

## How to apply

Clone HarteminkLab/RoboCOP, create the robocop-env conda environment from robocop-spec-file.txt, build the shared object with `bash gccCompile`, and set `cshared` in config.ini (modeled on config_example.ini) to the absolute path of librobocop.so, e.g. cshared=/home/myhome/RoboCOP/pkg/robocop/librobocop.so, so the RoboCOP engine can be invoked. Run RoboCOP on the ATAC-seq data alone (GEO GSE66386) and on MNase-seq (GSE153609) combined with ATAC-seq to produce genome-wide probabilistic nucleosome occupancy profiles, integrating nucleotide sequence alongside accessibility data. Run NucleoATAC on the identical ATAC-seq data and DANPOS2 on the identical MNase-seq data to obtain baseline nucleosome-position calls from the alternative tools. Compute nucleosome-position AUPR for all four methods against the same high-precision chemical-cleavage validation annotations — evaluating on an identical validation set is essential because NucleoATAC reports only ~20% of genomic nucleosomes, making raw call counts misleading. Write the scores to nucleosome_aupr_scores.csv and render the Supplementary Figure S12 comparison (RoboCOP_ATAC vs NucleoATAC; RoboCOP_(MNase+ATAC) vs DANPOS2) as supplementary_figure_S12_aupr_comparison.png; paper figures can be regenerated with dynacop_figures.ipynb. Interpret against the paper's expectations: RoboCOPATAC positions many more nucleosomes than NucleoATAC on the same data, and MNase-seq input yields markedly better performance than ATAC-seq alone.

## Related tools

- **RoboCOP** (Nucleosome/DBF occupancy caller under benchmark; run in ATAC-seq-only mode and MNase+ATAC mode after compiling librobocop.so and setting the cshared path in config.ini) — https://github.com/HarteminkLab/RoboCOP
- **NucleoATAC** (Baseline nucleosome-positioning caller run on the same ATAC-seq data for the Supplementary Figure S12A comparison (RoboCOPATAC vs NucleoATAC))
- **DANPOS2** (Baseline nucleosome-positioning caller run on the same MNase-seq data for the Supplementary Figure S12 comparison (RoboCOP_(MNase+ATAC) vs DANPOS2))
- **GCC** (Compiler invoked by RoboCOP's bash gccCompile script to generate the librobocop.so shared library required by the RoboCOP engine)

## Examples

```
git clone https://github.com/HarteminkLab/RoboCOP.git && cd RoboCOP/robocop && bash gccCompile
```

## Evaluation signals

- bash gccCompile produces librobocop.so and `ls `pwd`/librobocop.so` returns the same absolute path configured as cshared in config.ini
- All four methods' AUPR values (RoboCOP ATAC-only, RoboCOP MNase+ATAC, DANPOS2, NucleoATAC) are computed against the identical chemical-cleavage validation set before any comparison
- nucleosome_aupr_scores.csv contains an AUPR entry for each of the four methods, and supplementary_figure_S12_aupr_comparison.png reproduces the Supplementary Figure S12 layout pairing RoboCOP_ATAC with NucleoATAC and RoboCOP_(MNase+ATAC) with DANPOS2
- Result direction matches the paper: RoboCOP run with ATAC-seq alone positions more nucleosomes than NucleoATAC on the same data (Supplementary Figure S12A)
- RoboCOP's MNase-seq-based occupancy profile outperforms its ATAC-seq-only profile, consistent with the reported markedly better MNase performance

## Limitations

- NucleoATAC reports only a small fraction (~20%) of genomic nucleosomes, so raw positioned-nucleosome counts are not comparable across callers without scoring against a shared validation set
- No version numbers are stated for RoboCOP, NucleoATAC, DANPOS2, or any dependency, so exact reproduction of the published benchmark is not guaranteed
- MNase-seq input yields markedly better RoboCOP performance than ATAC-seq, so ATAC-only results underestimate the method's achievable accuracy
- Sequencing depth requirements are unquantified; the article notes the accuracy and comprehensiveness of chromatin occupancy profiles improve with deeper sequencing

## Evidence
