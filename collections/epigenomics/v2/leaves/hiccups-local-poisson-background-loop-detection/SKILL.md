---
name: hiccups-local-poisson-background-loop-detection
description: Use when you have a high-resolution (e.g., 10 kb) normalized Hi-C matrix
  in .hic or .cool format and need to call statistically enriched loop pixels with
  a local Poisson background model, or when benchmarking loop callers against each
  other.
when_to_use_negative:
- Input Hi-C data is at very low sequencing depth (e.g., 1.5% down-sampled), where
  Fit-Hi-C with q-value < 1e-5 retains no interactions and HiCCUPS with default parameters
  outputs less than 1/3 of the Peakachu loops
- The goal is de novo loop prediction in a platform lacking dedicated tools (e.g.,
  DNA SPRITE) rather than enrichment-based calling on a Hi-C matrix
- You need robust calling at ~30 million intrachromosomal reads, where Peakachu's
  supervised model rather than Poisson enrichment is the appropriate choice
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: HiCCUPS
  role: Enrichment-based loop caller that incorporates local background via a Poisson
    test; provides the benchmark loop set for overlap and validation comparisons
  repo: https://github.com/aidenlab/juicer
- name: Fit-Hi-C
  role: Distance-dependent two-pass spline loop caller whose second pass is filtered
    at q-value < 1e-5; second benchmark method
- name: Peakachu
  role: Random-Forest supervised loop predictor (score_genome + pool at probability
    threshold 0.9-0.95) whose 17,171 loops form the comparator set for the 72% overlap
    and 58% validation rate
  repo: https://github.com/tariks/peakachu
- name: Juicer
  role: Platform supplying the HiCCUPS command-line tools and .hic handling used for
    loop detection on Hi-C maps
  repo: https://github.com/aidenlab/juicer
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-17239-9
    title: A supervised learning framework for chromatin loop detection in genome-wide
      contact maps
  source_content_hash: sha256:42496be5acb8d02a681a39fd09b2e60ef27aa9589b4da1a94fbef82486a76ef8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aidenlab/juicer
  tools:
  - Fit-Hi-C
  - HiCCUPS
  - Juicer
  - Peakachu
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# hiccups-local-poisson-background-loop-detection

## Summary

Detect chromatin loops in Hi-C contact maps using HiCCUPS, an enrichment-based method that incorporates local background into its model via a Poisson test, and benchmark the resulting loop set against Peakachu and Fit-Hi-C predictions using three-way overlap and orthogonal validation ratios.

## When to use

Use when you have a high-resolution (e.g., 10 kb) normalized Hi-C matrix in .hic or .cool format and need to call statistically enriched loop pixels with a local Poisson background model, or when benchmarking loop callers against each other.

## When NOT to use

- Input Hi-C data is at very low sequencing depth (e.g., 1.5% down-sampled), where Fit-Hi-C with q-value < 1e-5 retains no interactions and HiCCUPS with default parameters outputs less than 1/3 of the Peakachu loops
- The goal is de novo loop prediction in a platform lacking dedicated tools (e.g., DNA SPRITE) rather than enrichment-based calling on a Hi-C matrix
- You need robust calling at ~30 million intrachromosomal reads, where Peakachu's supervised model rather than Poisson enrichment is the appropriate choice

## Inputs

- Rao2014-GM12878-MboI-allreps-filtered.10kb.cool (10 kb cooler Hi-C matrix)
- Trained Peakachu models (chromosome-wise pickled Random Forest models)
- Fit-Hi-C two-pass spline output with q-values
- Orthogonal validation sets (e.g., ENCODE ChIP-Seq peaks for 133 TFs and 10 histone modifications in GM12878, H3K27ac HiChIP bedpe)

## Outputs

- HiCCUPS loop list (bedpe) from the Poisson local-background test
- Fit-Hi-C interaction list filtered at q-value < 1e-5, top 140,000 by p-value
- Three-way Venn overlap summary of Peakachu/HiCCUPS/Fit-Hi-C loop sets
- method_comparison_summary.tsv with per-method validation ratios

## How to apply

Run HiCCUPS (Juicer/JuicerTools) on the 10 kb-binned GM12878 Hi-C matrix, which tests each pixel for enrichment against its local background using a Poisson test rather than a global distance curve. In parallel, Fit-Hi-C applies a distance-dependent two-pass spline to the same matrix, filtering the second pass at q-value < 1e-5, then sort surviving interactions by p-value and pool the top 140,000 for a comparable loop set size. Generate the Peakachu comparator set by running peakachu score_genome on the Rao2014-GM12878 10 kb cooler followed by peakachu pool with a probability threshold of 0.9-0.95. Intersect the three loop sets to build a three-way Venn overlap summary (expect ~72% of Peakachu loops to overlap HiCCUPS or Fit-Hi-C, i.e., 12,398/17,171). Finally, compute validation ratios of method-specific loops against orthogonal sets (e.g., ENCODE ChIP-Seq peaks, HiChIP), expecting Peakachu-specific 58%, HiCCUPS-specific 48%, and Fit-Hi-C-specific 40%.

## Related tools

- **HiCCUPS** (Enrichment-based loop caller that incorporates local background via a Poisson test; provides the benchmark loop set for overlap and validation comparisons) — https://github.com/aidenlab/juicer
- **Fit-Hi-C** (Distance-dependent two-pass spline loop caller whose second pass is filtered at q-value < 1e-5; second benchmark method)
- **Peakachu** (Random-Forest supervised loop predictor (score_genome + pool at probability threshold 0.9-0.95) whose 17,171 loops form the comparator set for the 72% overlap and 58% validation rate) — https://github.com/tariks/peakachu
- **Juicer** (Platform supplying the HiCCUPS command-line tools and .hic handling used for loop detection on Hi-C maps) — https://github.com/aidenlab/juicer

## Evaluation signals

- The three-way Venn overlap summary reproduces the reported 72% (12,398/17,171) of Peakachu loops overlapping HiCCUPS or Fit-Hi-C predictions
- Validation ratios of method-specific loops against orthogonal data match 58% (Peakachu-specific), 48% (HiCCUPS-specific), and 40% (Fit-Hi-C-specific)
- Fit-Hi-C second-pass results are filtered at q-value < 1e-5 and the top 140,000 interactions by p-value are pooled
- Peakachu comparator loops are generated with peakachu pool at a probability threshold in the 0.9-0.95 range after score_genome
- The output method_comparison_summary.tsv is well-formed and contains rows for each method-specific loop set with computed validation ratios

## Limitations

- Each method detects a unique set of chromatin loops with various distance distributions, CTCF binding, and validation ratios, so no single method captures the complete loop set
- HiCCUPS with default parameters outputted less than 1/3 of the Peakachu loops at each down-sample rate
- At very low depth (1.5% down-sampling), Fit-Hi-C with q-value < 1e-5 retains no interactions (cutoff must be relaxed to 0.1) and HiCCUPS parameters must be retuned to -f 0.9 -t 3.6,1,1,1

## Evidence

- [intro] HiCCUPS incorporates local background into its model and utilizes the Poisson test: "HiCCUPS8,23 incorporates local background into its model and utilizes the Poisson test"
- [intro] Fit-Hi-C performs a distance-dependent spline fitting procedure to refine its global background: "Fit-Hi-C21 performs a distance-dependent spline fitting procedure to refine its global background"
- [discussion] The results of the 2nd spline pass were then filtered with the q-value cutoff < 1e-5: "The results of the 2nd spline pass were then filtered with the q-value cutoff < 1e-5"
- [discussion] each method detects a unique set of chromatin loops with various distance distributions, CTCF binding, and validation ratios: "each method detects a unique set of chromatin loops with various distance distributions, CTCF binding, and validation ratios"
- [methods] The pool function serves to select the most significant non-redundant results from per-pixel probabilities calculated by the score functions: "The pool function serves to select the most significant non-redundant results from per-pixel probabilities calculated by the score functions"
