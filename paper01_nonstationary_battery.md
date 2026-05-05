# Where the Categorical Active-Inference Architecture Wins and Loses: A Three-Domain Non-Stationary Falsification Battery

**Project Magha — Paper 1**
**Rahul Chouhan** (ORCID: [0009-0008-7156-6841](https://orcid.org/0009-0008-7156-6841))
**DOI:** [10.5281/zenodo.20036497](https://doi.org/10.5281/zenodo.20036497)
**Draft — 2026-05-03**

## Abstract

We test the architectural prediction that an active-inference agent built on a Beta-Bernoulli pairwise world model with multiplicative posterior decay should dominate a memory-greedy baseline in non-stationary worlds — the regime where, by structural argument, posterior-driven exploration ought to outperform sample-mean greedy with epsilon. Three batteries are run: a procedurally generated tabular bandit with hidden-rule shifts at trials 100/200/300; an antimicrobial-resistance (AMR) environment with mid-run geographic prevalence shift; and a sepsis environment with mid-run patient-cohort shift. Across the three domains, the active-inference agent loses by 7-15 percentage points on the bandit, wins by 22pp on AMR, and loses by 3pp on sepsis. The architectural rule that emerges is narrower than the original prediction: **the agent wins only in regimes where per-instance diagnostic signals carry high information about the latent state**. This is closer to a "diagnostic decision-support architecture" than to a general Turing-ASI substrate. We document the falsification because it is load-bearing for any subsequent claim about extending the architecture (replacing the categorical world model with a learned generative model) at scale.

## 1. Setup

The agent under test is `magha`, a four-Turing-principles active-inference loop: (i) world-from-interaction, (ii) learned generative model (no fixed A-matrix or taxonomy), (iii) world-derived reward, (iv) novelty-driven exploration. The world model is a tabular Beta-Bernoulli pairwise predictor over (test_result, treatment_outcome). Action selection uses Expected Information Gain on treatment outcomes, computed in closed form. Treatment recommendation uses Thompson sampling on the per-treatment Beta posterior. Optionally, in non-stationary mode, the posterior is multiplicatively decayed toward the prior on every step (`alpha_new = gamma * alpha_old + (1 - gamma) * alpha_prior`).

The strongest classical baseline is a **memory-greedy sliding-window agent**: it tracks per-action empirical means over the last W observations, picks argmax, and explores epsilon-greedily. This is the LLM-with-in-context-learning analog: it has no structured world model, only a finite history of past (state, action, reward) tuples that get displaced as new ones arrive.

Three domains:

**B1 — Procedurally generated tabular bandit.** State space {0..4}, action space {0..2}, hidden rule s -> a sampled uniformly. Reward Bernoulli(0.8) on the optimal action, Bernoulli(0.2) elsewhere. Hidden rule reshuffles at trials 100, 200, 300 (fresh permutation each shift). 400 trials, 8 seeds. Six agents tested including a γ-sweep on AI-discount.

**B2 — AMR environment with geographic shift.** Pharmacodynamics-grounded simulator over 8 carbapenemase / ESBL / AmpC genes, 11 phenotypic + PCR + MALDI-TOF tests, 6 antibiotics. Pre-shift: south_asia geography (NDM-dominant, 35% prevalence). Post-shift: north_america (KPC-dominant, 28%). Shift forces optimal-treatment distribution to flip from aztreonam-avibactam-leaning to ceftazidime-avibactam / imipenem-relebactam-leaning. 400 episodes, shift at episode 200, 3 seeds, max_tests=4.

**B3 — Sepsis environment with cohort shift.** PhysioNet 2019 patient profiles, 10 binary vital-signs tests, 4 treatments (broad-spectrum, gram-positive-targeted, gram-negative-targeted, supportive). Pre-shift: cohort sampling weighted toward Sepsis_GramPos (4x marginal weight). Post-shift: weighted toward Sepsis_GramNeg (4x). Same protocol as B2.

## 2. Results

**B1 (cumulative reward rate, 8 seeds, 400 trials):**

| agent | cum_rate | pre-shift | post-shift-1 | post-shift-2 | post-shift-3 |
|---|---|---|---|---|---|
| random | 40.0% | 39.0% | 41.2% | 41.0% | 38.9% |
| memory_greedy_cumulative | 50.4% | **73.9%** | 43.6% | 43.1% | 41.0% |
| memory_greedy_epsilon (eps=0.10) | 52.4% | 69.5% | 47.5% | 48.0% | 44.7% |
| **memory_greedy_sliding (W=50, eps=0.10)** | **64.8%** | 68.9% | **61.1%** | **67.4%** | **61.6%** |
| active_inference_cumulative | 54.5% | 64.9% | 54.1% | 49.9% | 49.1% |
| active_inference_discount γ=0.95 | 53.7% | 54.1% | 53.6% | 53.8% | 53.2% |

Gamma sweep on AI-discount (cum_rate vs memory-sliding 64.8%): γ=0.90: 49.4% (-15.3pp); γ=0.95: 53.7% (-11.1pp); γ=0.99: 57.1% (-7.6pp); γ=0.999: 54.9% (-9.8pp). The best gamma still loses by 7.6pp. Active-inference-discount has BETTER stability (no catastrophic forgetting) but cannot reach memory-sliding's ceiling at any tested gamma.

**B2 (AMR, 3 seeds, 400 episodes):**

| agent | cum_success | cum_optimal | pre_opt | post_opt | recovery |
|---|---|---|---|---|---|
| random_treatment | 69.2% | 16.8% +/- 0.7% | 16.2% | 17.3% | - (0/3) |
| memory_greedy_sliding | 77.4% | 18.3% +/- 2.9% | 18.2% | 18.5% | - (0/3) |
| **magha** | **80.0%** | **43.7% +/- 9.8%** | 46.5% | 40.8% | 36 (2/3) |

Delta Magha vs memory-sliding: **+25.3pp cumulative, +22.3pp post-shift**.

**B3 (sepsis, 3 seeds, 400 episodes):**

| agent | cum_success | cum_optimal | pre_opt | post_opt | recovery |
|---|---|---|---|---|---|
| random_treatment | 53.2% | 28.2% +/- 1.2% | 29.3% | 27.2% | 48 (1/3) |
| **memory_greedy_sliding** | 58.6% | **32.0% +/- 1.6%** | 31.5% | 32.5% | 111 (3/3) |
| magha | 59.3% | 28.6% +/- 4.5% | 27.3% | 29.8% | 54 (2/3) |

Delta Magha vs memory-sliding: **-3.4pp cumulative, -2.7pp post-shift**.

## 3. The architectural rule

Across the three batteries, the active-inference architecture wins exactly once and the win is in the domain with the strongest per-instance diagnostic signal. The pattern is monotonic in test-information value, not in non-stationarity:

| Battery | Test-information value | Magha vs memory-greedy |
|---|---|---|
| B1 novel game | None (no tests) | -7 to -15pp |
| B3 sepsis | Low (binary vital signs) | -3pp |
| B2 AMR | High (PCR sensitivity ≥ 0.985) | +22pp |

**The architectural rule the data supports** is: *the active-inference loop wins iff per-instance diagnostic signals carry sufficient information that the world-model's pairwise posterior strictly dominates a sliding-window history*. It is closer to a "diagnostic decision-support architecture" than to the general "child machine learning from experience" framing in Turing's original argument.

This is a much narrower claim than "active inference dominates non-stationary worlds." That broader claim is falsified by B1 and B3. Specifically: multiplicative posterior decay is a poor proxy for change-point detection; the Beta-Bernoulli posterior cannot simultaneously represent (a) high confidence in the current rule AND (b) easy ability to flip when the rule changes. Sliding-window-greedy gets both for free: forgetting is structural (drop oldest observation), not posterior decay.

## 4. Honest caveats on the B2 win

The +22pp AMR win is real but its mechanism is not "Magha adapts faster to non-stationarity." Magha's per-block trajectory is pre-shift 46.5% -> post-shift 40.8%, a drop of 5.7pp. The world model does *not* fully re-learn the new prevalence in the 200-episode post-shift block. The win against memory-greedy comes mostly from Magha being able to *use per-isolate diagnostic test information* that memory-greedy structurally cannot access (memory-greedy is treatment-history-only). A fairer follow-up baseline is a memory-greedy agent that also uses test results — for instance, a k-nearest-neighbours classifier over recent (test_pattern, treatment, outcome) triples — which would isolate WM-adaptation from test-information-access. We have not run that ablation; we flag it as the next experiment.

## 5. Implications for scaling

The categorical Beta-Bernoulli architecture has a clean ceiling defined by the test-information rule above. To extend the architecture to regimes where it currently loses (open-ended worlds, low-information observation streams, non-stationary regimes without an explicit change-point signal), one of the following extensions is required:

(a) **Change-point detection** — explicit dual-system: a fast tracker that resets the posterior when KL between predicted and observed exceeds a threshold. Adds one hyperparameter and partial state.

(b) **Hierarchical generative model** — shifts as latent variables. Requires moving from tabular Beta-Bernoulli to a learned generative model that can represent regime change as part of its variational posterior. This is the gap-1 extension documented in the project's `THEORETICAL_FOUNDATION.md`.

(c) **Information-bearing sensors as a precondition** — restrict the deployment claim to domains where the test panel has high enough Fisher information to make per-instance Bayesian updating dominate sliding-window memory.

We expect (b) to be the only path that scales to the open-ended Era-of-Experience regime. (a) is a useful patch within the categorical architecture. (c) is the honest scope statement for the architecture as currently specified.

## 6. What this paper is for

We are documenting falsifications as part of an open architectural research program. The cumulative picture across now eight experiments — two wins (within-episode AMR sample efficiency 1.48x, sepsis 4x) and six nulls/losses on materials, closed-loop self-play, novel games, and the cohort-shift batteries here — is that the categorical Magha architecture is a strong fit for a narrow class of decision-rich, signal-rich domains and not yet a general Turing-ASI substrate. The neural-generative extension is the next architectural test. Until that is built and benchmarked, the honest claim about the current code is the test-information-gated rule above.

## Files

- `experiment_novel_game_nonstationary.py`, `results_novel_game_nonstationary.json`, `results_nonstat_g{090,099,0999}.json`
- `experiment_amr_nonstationary.py`, `results_amr_nonstationary.json`
- `experiment_sepsis_nonstationary.py`, `results_sepsis_nonstationary.json`

## Status

Hold for batched send to active-inference / world-model / RL researchers per project documentation strategy.
