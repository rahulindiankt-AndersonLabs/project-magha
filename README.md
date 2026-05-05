# Project Magha — Papers

A research notebook documenting the empirical path toward a Turing-shaped child-machine: an agent that learns from its own experience via the Era-of-Experience loop (Sutton & Silver 2024) with active inference (Friston / Da Costa / Parr) as the efficiency layer.

Each paper documents a single empirical step — successes and falsifications alike. We publish failures with the same care as wins; the architectural story only makes sense if the negative results are visible.

**Author:** Rahul Chouhan ([ORCID 0009-0008-7156-6841](https://orcid.org/0009-0008-7156-6841))

---

## Paper 1 — Where the Categorical Active-Inference Architecture Wins and Loses: A Three-Domain Non-Stationary Falsification Battery

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.20036497.svg)](https://doi.org/10.5281/zenodo.20036497)

A three-domain falsification battery testing whether a Beta-Bernoulli pairwise active-inference agent dominates a memory-greedy baseline in non-stationary worlds. Result: **the agent loses by 7-15pp on a procedurally generated bandit, wins by 22pp on antimicrobial-resistance under geographic shift, and loses by 3pp on sepsis under cohort shift.** The architectural rule that emerges is narrower than the original prediction: the agent wins only in regimes where per-instance diagnostic signals carry high information about the latent state. We document the falsification because it is load-bearing for every subsequent paper in the series.

- Read: [paper01_nonstationary_battery.md](paper01_nonstationary_battery.md)
- Cite: `Chouhan, R. (2026). Where the Categorical Active-Inference Architecture Wins and Loses: A Three-Domain Non-Stationary Falsification Battery. Zenodo. https://doi.org/10.5281/zenodo.20036497`

---

## License

All papers in this repository are released under [CC-BY-4.0](https://creativecommons.org/licenses/by/4.0/).

## Contact

Rahul Chouhan — rahulindiankt@gmail.com
