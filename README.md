# DSC-212-KARATE-CLUB-GRAPH_PROJECT
# Modularity on the Karate Club Graph

**DSC212: Graph Theory | Research Assignment**

Spectral modularity community detection implemented from scratch (NumPy + NetworkX) and applied to Zachary's Karate Club network. The notebook detects communities by recursive bisection, visualises every split, tracks four node-level centrality metrics across the splits, and discusses what the results mean.

---

## Repository contents

| File | Description |
|---|---|
| `Karate_Club_Modularity.ipynb` | The full report: code, outputs, 32 figures, discussion. Runs top-to-bottom with no manual edits. |
| `requirements.txt` | Python dependencies. |
| `README.md` | This file. |

## Quick start

```bash
git clone <your-repo-url>
cd <your-repo-name>
pip install -r requirements.txt
jupyter notebook Karate_Club_Modularity.ipynb
```

Then choose **Kernel → Restart & Run All**. Runtime is under a minute. All randomness is seeded (`SEED = 7`), so results are reproducible.

**Requirements:** Python 3.9+, `numpy`, `pandas`, `matplotlib`, `networkx >= 3.0`.

---

## Background

In the early 1970s Wayne Zachary recorded friendships among 34 members of a university karate club (78 edges). During his study the instructor ("Mr. Hi", node 0) and the administrator (node 33) fell out and the club split in two. Because the network was recorded *before* the split, it tests whether a method using only graph structure can recover the real division.

Cutting as few edges as possible is a poor objective, because it isolates low-degree nodes. **Modularity** instead compares the edges inside groups with what a degree-preserving random network would give:

```
B = A − k kᵀ / 2m        Q = (1 / 4m) · sᵀ B s
```

Maximising Q over s ∈ {−1, +1}ⁿ is relaxed to a unit-norm eigenvector problem. The leading eigenvector of B is rounded by sign, then applied recursively to each community while the leading eigenvalue of the restricted matrix is positive.

## What the notebook does

| Part | Content |
|---|---|
| 0 | Setup and plotting helpers |
| 1 | Data exploration and ground truth |
| 2 | Modularity matrix B: null model, properties (symmetry, B·1 = 0), spectrum |
| 3 | The score Q, why cut size fails, a 5,000-split random baseline |
| 4 | Spectral bipartition: eigenvector, sign rounding, threshold sweep, refinement |
| 5 | Recursive bisection: pseudocode and implementation, split log |
| 6 | Graph drawn after every split (fixed layout, stable community colours) |
| 7 | Check of the ΔQ identity and the stopping rule |
| 8 | Degree, betweenness, closeness and clustering after every split |
| 9 | Evolution plots, top-5 stability, mean ranks, Spearman correlations |
| 10 | How community structure shapes the metrics (participation, coverage) |
| 11 | Comparison with CNM, Louvain, label propagation and Girvan–Newman |
| 12 | The same algorithm on 8 other graphs, plus a detectability sweep |
| 13 | Discussion |
| 14 | Concept-check questions and references |

## Key results

| Result | Value |
|---|---|
| Q of the real split | 0.358 (about 8 SD above random splits) |
| First spectral split | matches the real factions for 33 of 34 members (only **node 8** differs), Q = 0.3715 |
| Recursive run, handout variant | 5 communities, Q = 0.409 |
| Recursive run, corrected variant | 4 communities, Q = 0.393 |
| Best Q in the comparison | Louvain, 0.4198 (best of 30 seeds) |
| Nodes always in the top 5 | degree: 0, 1, 32, 33; betweenness: 0, 32, 33; closeness: 0, 33; clustering: 7, 12, 14 |

Main findings:

- Nodes **0 and 33** (the two leaders) stay central at every iteration.
- **Betweenness** is the metric most affected by splitting. Node 2 falls from 0.144 to 0.007 because its brokerage was *between* communities.
- Raw centrality values rise as communities shrink (a size-normalisation effect), so the discussion compares **ranks**.
- Modularity > 0 does not prove real structure: a random graph still gets Q ≈ 0.3.

## Two design decisions to be aware of

1. **"Metrics after each split".** On the whole graph the four metrics never change. They are therefore computed on the subgraph induced by each node's current community. Iteration 0 equals the plain whole-graph NetworkX values (verified in the notebook). Singleton communities get 0.
2. **The handout's ΔQ formula.** `ΔQ = sᵀB⁽ᶜ⁾s / 4m` is exact only for the first split. For sub-communities the exact gain is `(sᵀB⁽ᶜ⁾s − 1ᵀB⁽ᶜ⁾1) / 4m` (Newman's corrected matrix B⁽ᵍ⁾). The notebook runs both:
   - `handout` follows the assignment exactly, with two safety guards (constant sign vector; never accept a split with ΔQ ≤ 0).
   - `corrected` uses Newman's B⁽ᵍ⁾, for which the stopping test is exact.

   Part 7 shows the numerical difference.

## Method attribution

- M. E. J. Newman (2006). *Modularity and community structure in networks.* PNAS 103(23):8577–8582.
- W. W. Zachary (1977). *An information flow model for conflict and fission in small groups.* J. Anthropological Research 33(4):452–473.

Full reference list is in Part 14 of the notebook.

## Notes

- Node ids follow NetworkX (0–33). Zachary's paper uses 1–34, so add 1 to compare.
- Only NumPy, NetworkX, pandas and matplotlib are used. NMI and ARI are implemented by hand.
