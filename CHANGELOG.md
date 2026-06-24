# Changelog

All notable changes to this project are documented in this file.
This project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]
### Docs
- Added PyPI, License, arXiv, and Zenodo DOI (concept) badges to the README header.

## [0.1.2] — 2026-06-24
### Changed
- Package metadata now lists both authors (added the co-author's email), so PyPI
  and the Zenodo record show **Yu Shuai** and **Clarence W. Rowley**.

## [0.1.1] — 2026-06-24
### Added
- `examples` optional-dependency extra (Jupyter stack) for running the notebooks:
  `pip install -e ".[examples]"`.

### Changed
- `ks_base_solution.ipynb` is now self-contained: when no cached attractor
  initial condition is found, it auto-generates one via a one-off `t = 120`
  burn-in before the `T = 10` reconstruction — no manual switching of `configs.T`
  between an IC-generation run and the reconstruction run.
- README distinguishes library-only installs (`pip install sr-opinf`) from
  full-reproduction installs (clone + `pip install -e ".[examples]"`).

## [0.1.0] — 2026-06-24
### Added
- Initial release. Symmetry-reduced model reduction of shift-equivariant systems
  via operator inference (S-R POD–Galerkin and S-R OpInf) for 1D periodic PDEs,
  on spectral and cubic-spline grids, with the Kuramoto–Sivashinsky experiments
  from the accompanying paper ([arXiv:2507.18780](https://arxiv.org/abs/2507.18780)).

[Unreleased]: https://github.com/Yu-Shuai-PU/sr-opinf/compare/v0.1.2...HEAD
[0.1.2]: https://github.com/Yu-Shuai-PU/sr-opinf/releases/tag/v0.1.2
[0.1.1]: https://github.com/Yu-Shuai-PU/sr-opinf/releases/tag/v0.1.1
[0.1.0]: https://github.com/Yu-Shuai-PU/sr-opinf/releases/tag/v0.1.0
