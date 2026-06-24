# SROpInf — Symmetry-Reduced Operator Inference

[![PyPI](https://img.shields.io/pypi/v/sr-opinf.svg)](https://pypi.org/project/sr-opinf/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![arXiv](https://img.shields.io/badge/arXiv-2507.18780-b31b1b.svg)](https://arxiv.org/abs/2507.18780)
[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.20823370.svg)](https://doi.org/10.5281/zenodo.20823370)

`SROpInf` is a Python package for **symmetry-reduced model reduction of shift-equivariant
systems via operator inference**. For a 1D periodic PDE whose solutions are dominated by a
travelling / drifting structure (e.g. a travelling wave), it builds a reduced-order model (ROM)
that first factors out the continuous translation symmetry and then learns the reduced dynamics in
the co-moving frame — either **intrusively** (symmetry-reduced POD–Galerkin projection) or
**non-intrusively** from snapshot data (symmetry-reduced operator inference, S-R OpInf).

The package accompanies the paper

> Yu Shuai and Clarence W. Rowley,
> *Symmetry-reduced model reduction of shift-equivariant systems via operator inference*,
> arXiv:[2507.18780](https://arxiv.org/abs/2507.18780) (2025).
> Submitted to *Advances in Computational Mathematics* (in revision).

and reproduces all of its Kuramoto–Sivashinsky (KS) numerical experiments.

---

## Method at a glance

For a shift-equivariant system the solution `q(x, t)` is decomposed into a **shift amount** `c(t)`
(the location of the travelling structure) and a **co-moving profile** obtained by aligning each
snapshot to a fixed template `q_template = cos(2πx/L)` (template fitting). The reduced model then
evolves

* the reduced co-moving state `z(t)` (POD coefficients of the template-fitted, scaled snapshots), and
* the scalar shift amount `c(t)` via a reconstruction equation `dc/dt = numerator / denominator`.

Two ways to obtain the reduced operators are provided:

| ROM | Intrusive? | How the reduced operators are obtained |
|-----|------------|----------------------------------------|
| **S-R POD–Galerkin** | yes | Galerkin projection of the (scaled, symmetry-reduced) FOM right-hand side onto the POD subspace. |
| **S-R OpInf** | no | Ridge-regression of reduced polynomial operators to the data; optional **re-projection** for consistency and **penalty (Tikhonov) regularization** for stability. |

Two spatial discretizations are supported, which lets the shift-induced interpolation error be
assessed directly:

* `Grid1DUniformSpectral` — equispaced spectral grid; the shift and spatial derivatives are **exact**
  (Fourier), with 3/2-rule dealiasing for the quadratic nonlinearity.
* `Grid1DCubicSpline` — grid-based discretization; the shift, derivative, and inner-product
  operators are realized via **periodic cubic-spline interpolation** (hence an additional
  interpolation error, as encountered when non-intrusiveness is actually relevant).

---

## Repository structure

```
SROpInf/
├── pyproject.toml              # package metadata + dependencies (installable, src-layout)
├── requirements.txt            # runtime deps (see also pyproject.toml)
├── requirements-dev.txt        # dev/test deps
├── RELEASING.md                # how to publish a new version to PyPI
├── LICENSE                     # MIT License
├── .github/workflows/
│   └── publish.yml             # CI: build + publish to PyPI on a v* tag (Trusted Publishing)
├── src/SROpInf/
│   ├── grids/grid1d.py         # Grid1D base + Grid1DUniformSpectral + Grid1DCubicSpline
│   │                           #   (inner product, sqrt-mass map R, shift_x, diff_x, fft/ifft)
│   ├── models/
│   │   ├── model.py            # FullOrderModel, SymmetryReducedScaledFullOrderModel (.project),
│   │   │                       #   SymmetryReducedScaledReducedOrderModel (.solve, .sample_and_compare)
│   │   └── ks.py               # KuramotoSivashinsky equation (polynomial operators)
│   ├── sr_tools.py             # template_fitting (shift amount c(t)); sropinf (the S-R OpInf
│   │                           #   regression, with re-projection / grid-search / cross-validation)
│   ├── mode_decomposition.py   # pod (energy-truncated POD of the symmetry-reduced snapshots)
│   ├── timestepper.py          # explicit (Euler/RK2/RK4) and semi-implicit (RK2CN/RK3CN) steppers
│   ├── dataloader.py           # FOMDataloader (loads trajectories / shift amounts from disk)
│   └── typing.py               # array type aliases
├── example/ks/
│   ├── configs.py              # all KS parameters and file paths (single source of truth)
│   ├── plot.py                 # figure helpers (x–t contours, spatial profiles, error bands, ...)
│   ├── ks_base_solution.ipynb  # experiment 1: single base travelling-wave solution
│   └── ks_perturbed_solutions.ipynb  # experiment 2: family of perturbed solutions (+ amplitude sweep)
└── test/                       # pytest suite (grids + model polynomial composition)
```

> `output/` (generated trajectories, data, and figures) is **not** tracked — it is fully
> regenerated by running the example notebooks.

---

## Installation

Requires **Python ≥ 3.10** (developed and tested with Python 3.13).

**Which option do you need?**

- **Just using the library** — if you only want to call the `SROpInf` functions from
  [`src/SROpInf`](src/SROpInf/) in your own code, install from PyPI; that is all you need.
- **Reproducing the paper experiments** — the `example/` notebooks are **not** shipped on
  PyPI (only the `SROpInf` library is). Clone the repository to get them, then install the
  library — see *From source* below.

### From PyPI (recommended for library use)

```bash
python3 -m pip install sr-opinf
```

The package is then importable as `SROpInf`:

```python
import SROpInf
```

> The install name and the import name differ on purpose: you **install** the distribution
> `sr-opinf` but **import** the package `SROpInf` — the same pattern as
> `pip install scikit-learn` / `import sklearn`.

### From source (for reproducing the experiments / development)

The `example/` notebooks live **only in the repository** (they are not part of the PyPI
package), so clone it first, then install the library:

```bash
git clone https://github.com/Yu-Shuai-PU/sr-opinf
cd sr-opinf
python3 -m pip install -e ".[examples]"   # editable install + Jupyter stack to run the notebooks
# extras: ".[examples]" -> jupyter + ipykernel + ipywidgets;  ".[dev]" -> pytest + ipywidgets
# or just: python3 -m pip install -e .     # runtime deps only (numpy, scipy, matplotlib, tqdm)
```

The `-e` (editable) install puts the `SROpInf` package on the path so the example notebooks can
`import SROpInf...` directly. (You could instead `pip install sr-opinf` for the library — the
key point is that running the experiments requires the cloned `example/` files either way.)
Exact dependency versions are declared in [`pyproject.toml`](pyproject.toml).

---

## Reproducing the paper experiments

The experiments live in [`example/ks/`](example/ks/). Run the notebooks top-to-bottom:

### 1. Base travelling-wave solution — `ks_base_solution.ipynb`
Reconstruction (train = test) of a single KS travelling-wave solution. Builds the symmetry-reduced
POD subspace and compares **S-R POD–Galerkin** against **S-R OpInf** (plain, re-projected, and
penalty-regularized), on **both** the spectral and the cubic-spline grids — quantifying the effect
of the shift-induced interpolation error.

> **First run is self-contained, just slower.** The benchmark IC must sit on the KS attractor. If a
> cached `output/ks_base_solution/data/traj_init_base.npy` is not found, the notebook automatically
> runs a one-off `t = 120` burn-in to generate (and cache) it before the `T = 10` reconstruction —
> no manual configuration needed. Later runs reuse the cached IC and start immediately.

### 2. Perturbed solutions — `ks_perturbed_solutions.ipynb`
**Prerequisite:** run `ks_base_solution.ipynb` first (it generates the cached base IC this notebook
reuses), and switch [`example/ks/configs.py`](example/ks/configs.py) to the perturbed-solutions case
(`type_traj_training` and the matching `base_path`; the notebook's first cell asserts both).

Generalization across a family of solutions: trains on perturbed initial conditions and evaluates on
a disjoint, held-out testing set. Includes a **perturbation-amplitude sweep**: per-trajectory testing
errors are written to `output/<case>/sweep_rRMSE/` and the final cell aggregates them into a box-plot
of testing error vs. perturbation amplitude.

### Configuration
All parameters and paths are centralized in [`example/ks/configs.py`](example/ks/configs.py):

* `type_traj_training` — `"base_solution"` or `"perturbed_solutions"`.
* physical / discretization — `Lx = 2π`, `nx = 256`, `nu = 4/87`, `T = 10`, `dt = 1e-3`.
* ROM — `poly_comp = [1, 2]` (linear + quadratic), `pod_energy_threshold`, the OpInf
  `penalty_weight_*` (Tikhonov penalties on the inferred operators), and
  `shift_speed_denom_threshold` (regularizes the shift-reconstruction denominator).

The output directory defaults to `output/ks_multiple_solutions/` and can be overridden with the
`NISRH_OUTPUT_DIR` environment variable. Each notebook's first cell `assert`s that `configs` points
at the matching output folder and trajectory type, to prevent accidentally mixing cases.

---

## Tests

```bash
python3 -m pytest            # from the SROpInf/ directory
```

The suite checks the grid operators (spectral vs. cubic-spline shift / derivative, inner products)
and the polynomial-composition machinery of the model class.

---

## Citation

If you use this code, please cite the accompanying paper:

```bibtex
@misc{shuai2025symmetryreduced,
  title         = {Symmetry-reduced model reduction of shift-equivariant systems via operator inference},
  author        = {Shuai, Yu and Rowley, Clarence W.},
  year          = {2025},
  eprint        = {2507.18780},
  archivePrefix = {arXiv},
  primaryClass  = {math.NA},
  note          = {Submitted to Advances in Computational Mathematics (in revision)},
  url           = {https://arxiv.org/abs/2507.18780}
}
```

## License

Released under the MIT License (see [LICENSE](LICENSE)).
