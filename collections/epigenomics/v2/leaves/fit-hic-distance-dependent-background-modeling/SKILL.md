---
name: fit-hic-distance-dependent-background-modeling
description: Use when calling significant chromatin interactions from a Hi-C matrix
  with an enrichment-based method to benchmark against Peakachu/HiCCUPS, or when a
  distance-dependent background null is needed for interaction significance.
when_to_use_negative:
- At very shallow sequencing depth (e.g., 1.5% down-sampled GM12878), where no interactions
  survive the q < 1e-5 cutoff and the method yields an empty loop set without relaxing
  the threshold to 0.1.
- When the goal is predicting loops in platforms lacking a compatible enrichment caller
  (e.g., DNA SPRITE), where a supervised framework like Peakachu is the intended approach
  instead.
- As a replacement for Peakachu's Random Forest supervised model — Fit-Hi-C is a background/enrichment-based
  caller, not a classifier trained on orthogonal positive sets.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Fit-Hi-C
  role: Runs the distance-dependent two-pass spline background model producing the
    q-value-filtered loop set used in the benchmark comparison
- name: Peakachu
  role: Provides the supervised-learning loop predictions (score_genome + pool at
    probability 0.9-0.95) intersected against the Fit-Hi-C set
  repo: https://github.com/tariks/peakachu
- name: HiCCUPS (Juicer)
  role: Provides the local-background Poisson-test loop calls (via Juicer tools) compared
    alongside Fit-Hi-C
  repo: https://github.com/aidenlab/juicer
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-17239-9
    title: A supervised learning framework for chromatin loop detection in genome-wide
      contact maps
  source_content_hash: sha256:34e16ca6aec748b4fcab85fc1db18b932cbe82713b7bdb77a6f927b622c3c604
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tariks/peakachu
  tools:
  - Fit-Hi-C
  - HiCCUPS (Juicer)
  - Peakachu
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# fit-hic-distance-dependent-background-modeling

## Summary

Benchmark loop-calling with Fit-Hi-C's distance-dependent two-pass spline background model on a Hi-C contact matrix, so its enrichment-based loop set can be compared against Peakachu predictions and HiCCUPS calls in terms of overlap and orthogonal validation rates.

## When to use

Use when calling significant chromatin interactions from a Hi-C matrix with an enrichment-based method to benchmark against Peakachu/HiCCUPS, or when a distance-dependent background null is needed for interaction significance.

## When NOT to use

- At very shallow sequencing depth (e.g., 1.5% down-sampled GM12878), where no interactions survive the q < 1e-5 cutoff and the method yields an empty loop set without relaxing the threshold to 0.1.
- When the goal is predicting loops in platforms lacking a compatible enrichment caller (e.g., DNA SPRITE), where a supervised framework like Peakachu is the intended approach instead.
- As a replacement for Peakachu's Random Forest supervised model — Fit-Hi-C is a background/enrichment-based caller, not a classifier trained on orthogonal positive sets.

## Inputs

- Intrachromosomal Hi-C contact matrix at 10 kb resolution (e.g., Rao2014-GM12878-MboI-allreps-filtered.10kb.cool / .hic)
- Chromosome-arm or genome-wide distance bins for the spline background fit
- Orthogonal validation sets (e.g., ENCODE ChIP-Seq peaks for 133 TFs and 10 histone modifications in GM12878) for computing loop validation ratios

## Outputs

- Second-pass Fit-Hi-C interaction list filtered at q-value < 1e-5, sorted by p-value, top 140,000 pooled
- Three-way Venn overlap summary of Peakachu/HiCCUPS/Fit-Hi-C loop sets (e.g., 72% Peakachu overlap with either method)
- Method comparison summary table (method_comparison_summary.tsv) with validation ratios for method-specific loops

## How to apply

Run Fit-Hi-C's distance-dependent spline fitting procedure on the same contact matrix used for the competing methods (e.g., the 10 kb-binned Rao2014-GM12878 cooler). The method builds a global background corrected for genomic distance via spline fitting; run the first pass, then a second spline pass, and filter the second pass at q-value < 1e-5. Sort the surviving interactions by p-value and pool the top 140,000 to define the Fit-Hi-C loop set for the Venn comparison. Intersect this set with Peakachu (probability threshold 0.9-0.95 via score_genome + pool) and HiCCUPS (local-background Poisson test) calls, then compute validation ratios of method-specific loops against orthogonal validation sets. Accept the run if Peakachu loops overlap ~72% with HiCCUPS or Fit-Hi-C and validation ratios reach ~58%/48%/40% for Peakachu-/HiCCUPS-/Fit-Hi-C-specific loops. Note that at very low sequencing depth (e.g., 1.5% down-sampling) no interactions survive q < 1e-5, so the cutoff must be relaxed to 0.1.

## Related tools

- **Fit-Hi-C** (Runs the distance-dependent two-pass spline background model producing the q-value-filtered loop set used in the benchmark comparison)
- **Peakachu** (Provides the supervised-learning loop predictions (score_genome + pool at probability 0.9-0.95) intersected against the Fit-Hi-C set) — https://github.com/tariks/peakachu
- **HiCCUPS (Juicer)** (Provides the local-background Poisson-test loop calls (via Juicer tools) compared alongside Fit-Hi-C) — https://github.com/aidenlab/juicer

## Evaluation signals

- Second-pass spline output retains interactions only after filtering at q-value < 1e-5; pooling the p-value-sorted survivors must yield ~140,000 interactions.
- Three-way Venn intersection should reproduce ~72% (12,398/17,171) of Peakachu loops overlapping HiCCUPS or Fit-Hi-C calls.
- Orthogonal validation ratios of method-specific loops should approximate 58% (Peakachu), 48% (HiCCUPS), and 40% (Fit-Hi-C).
- Final comparison artifacts assemble into method_comparison_summary.tsv with the expected overlap and validation columns.
- If re-run at a 1.5% down-sample rate, verify no interactions survive q < 1e-5 (empty set) and that the q-value cutoff has been relaxed to 0.1.

## Limitations

- At the 1.5% down-sample rate, no interactions remain with q-value < 1e-5, forcing a relaxed cutoff of 0.1 and limiting comparability at low depth.
- Each method (Peakachu, HiCCUPS, Fit-Hi-C) detects a unique set of loops with different distance distributions, CTCF binding, and validation ratios — no single method captures the complete loop set.
- HiCCUPS with default parameters outputs less than 1/3 of the Peakachu loops at each down-sample rate, complicating like-for-like set-size comparisons.
- No software version numbers are stated for Fit-Hi-C (or HiCCUPS/Boost-HiC) in the source material, so exact reproducibility of parameter settings may vary.

## Evidence

- [intro] Fit-Hi-C performs a distance-dependent spline fitting procedure to refine its global background: "Fit-Hi-C21 performs a distance-dependent spline fitting procedure to refine its global background"
- [discussion] Second-pass q-value filter at 1e-5: "The results of the 2nd spline pass were then filtered with the q-value cutoff < 1e-5"
- [discussion] Low-depth cutoff relaxation to 0.1: "At the 1.5% down-sample rate, we changed the q-value cutoff to 0.1 because no interactions remained with q-value < 1e-5."
- [discussion] Each method detects a unique loop set: "each method detects a unique set of chromatin loops with various distance distributions, CTCF binding, and validation ratios"
- [methods] Pool function selects non-redundant significant results: "The pool function serves to select the most significant non-redundant results from per-pixel probabilities calculated by the score functions"
