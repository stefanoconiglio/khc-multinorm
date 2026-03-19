# K-Hyperplane Clustering with Multiple Norms

Code for the ICLR 2026 paper:

> **K-Hyperplane Clustering with Multiple Norms**
> [[OpenReview]](https://openreview.net/forum?id=VJAqqtVXfD)

---

## Overview

**K-Hyperplane Clustering (KHC)** partitions a set of *m* points in *n* dimensions into *K* clusters, where each cluster is represented by a hyperplane. The objective is to minimize the total sum of squared distances from each point to its assigned hyperplane.

Formally, given data points $a_1, \ldots, a_m \in \mathbb{R}^n$, KHC solves:

$$\min_{w_j, \gamma_j, x_{ij}} \sum_{i,j} x_{ij} (w_j^\top a_i - \gamma_j)^2$$

subject to each point being assigned to exactly one cluster and each normal vector $w_j$ satisfying a norm constraint.

This paper investigates how the choice of norm constraint on the hyperplane normals — **L1**, **L2**, **L-infinity**, or combinations thereof — affects:
- the tightness of MIP formulations,
- computational hardness,
- and solution quality on benchmark instances.

---

## Repository Structure

```
.
├── data/
│   └── small-all/         # 270+ benchmark instances (m=10–30, n=2–3, K=2–3)
├── src-instances/         # C++ heuristic solvers and instance generators
│   ├── main.cpp           # Main solver executable
│   ├── makeinstance.cpp   # Instance generator
│   ├── algorithm.cpp      # Bradley-Mangasarian and Tabu Search algorithms
│   ├── exact.cpp          # SVD-based exact hyperplane updates
│   └── ...
└── src-formulations/      # AMPL exact MIP formulations
    ├── compact.mod        # Core objective and assignment constraints
    ├── n1.mod             # L1-norm constraint
    ├── n2.mod             # L2-norm constraint
    ├── nI.mod             # L-infinity norm constraint
    ├── n1nI.mod           # Combined L1 + L-infinity constraint
    ├── smallSets.mod      # Symmetry-breaking (partial pairing) sets
    ├── fullSets.mod       # Full assignment sets
    ├── gurobi.run         # Gurobi solver script
    └── batchrun.sh        # Batch runner for all instances
```

---

## Requirements

### Exact solver (MIP formulations)
- [AMPL](https://ampl.com/)
- [Gurobi](https://www.gurobi.com/) 9.0 or higher

### Heuristic solver (C++)
- g++ with C++11
- [Boost](https://www.boost.org/) (≥ 1.47.0)
- [Lapack++](http://lapackpp.sourceforge.net/)

---

## Running the Exact Formulations

**1. Prepare the data:**

```bash
unzip data.zip
```

**2. Run all instances:**

```bash
cd src-formulations
./batchrun.sh
```

This generates a `parallelcommands` file with one AMPL call per instance and formulation. Run in parallel with e.g. `parallel < parallelcommands`.

**Formulation variants** (edit `batchrun.sh` to select):

| Model name | Norm constraints |
|---|---|
| `ssn2`    | L2 only |
| `ssn1`    | L1 only |
| `ssnI`    | L-infinity only |
| `ssn2nI`  | L2 + L-infinity |
| `ssn2n1`  | L2 + L1 |
| `ssn1nI`  | L1 + L-infinity |
| `ssn2n1nI`| L2 + L1 + L-infinity |

The prefix `ss` uses symmetry-breaking constraints (smallSets); `fs` uses full sets.

Solver output is written to `log/JOURNAL/LOGDIR/`.

---

## Running the Heuristic Solver

**Build:**

```bash
cd src-instances
make
```

This produces `bin/k-hc-2014` and `bin/makeinstance-2014`.

**Solve an instance:**

```bash
./bin/k-hc-2014 -f data/small-all/sr_10_2_2_1022_ins -m mBM -k 2
```

Key options:

| Flag | Description |
|---|---|
| `-f <file>` | Input instance file |
| `-m <method>` | Algorithm: `BM`, `mBM`, `TSBM`, `mTSBM`, `APR`, `mAPR`, ... |
| `-k <K>` | Number of clusters |
| `-t <seconds>` | Time limit |
| `-s <seed>` | Random seed |

**Generate a new instance:**

```bash
./bin/makeinstance-2014 -t sr -m 50 -n 3 -k 3 -s 42
```

Options: `-t` (type: `sr` = semi-random), `-m` (points), `-n` (dimensions), `-k` (clusters), `-s` (seed), `-v` (variance).

---

## Instance Format

Plain-text `.ins` files:

```
Semi_random
m: 10
n: 2
k: 2
seed: 1022
Variance: 0.003
Data_set:
p: 7.6  8.0
p: 3.2  4.1
...
```

The naming convention is `sr_M_N_K_SEED_ins`.

---

## Algorithms

### Heuristic solver

The heuristic algorithms (BM, TSBM, APR, and variants) are from:

> Amaldi, E., Coniglio, S. (2013). *A distance-based point-reassignment heuristic for the k-hyperplane clustering problem.* European Journal of Operational Research, 227(1), 22–29.

| Name | Description |
|---|---|
| BM / mBM | Bradley-Mangasarian local search (multi-start) |
| TSBM / mTSBM | Tabu Search variant of BM |
| APR / mAPR | Alternating Projection Reassignment |

### Exact solver

The MIP formulations (AMPL + Gurobi) are introduced in this paper (see citation below).

| Name | Description |
|---|---|
| Exact (AMPL) | MIP solved via Gurobi (time limit: 4 hours) |

---

## Citation

If you use this code, please cite:

```bibtex
@inproceedings{
  title     = {K-Hyperplane Clustering with Multiple Norms},
  booktitle = {The Thirteenth International Conference on Learning Representations},
  year      = {2026},
  url       = {https://openreview.net/forum?id=VJAqqtVXfD}
}
```
