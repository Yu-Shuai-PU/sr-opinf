# Releasing

`sr-opinf` is published to PyPI automatically by GitHub Actions
([`.github/workflows/publish.yml`](.github/workflows/publish.yml)) whenever a
version tag (`v*`) is pushed. The published package is a **frozen snapshot** —
pushing code to GitHub does *not* update PyPI; only cutting a new release does.

## Key rules

- **A version number can never be reused.** PyPI rejects re-uploading a version
  that already exists (even after you yank or delete it). Every release needs a
  new, higher version.
- **The published version is the `version` in `pyproject.toml`, not the tag
  name.** The tag only *triggers* the build. Keep them in sync:
  tag `vX.Y.Z` ⇔ `version = "X.Y.Z"`.
- **Push `main` before pushing the tag**, so the workflow file is already on
  GitHub when the tag event fires (otherwise the publish does not trigger).

## Choosing the version (SemVer)

Starting from the current `X.Y.Z`:

| Change | Bump | Example |
|--------|------|---------|
| Bug fix, no API change          | patch `Z` | `0.1.0 → 0.1.1` |
| New feature/files, back-compat  | minor `Y` | `0.1.0 → 0.2.0` |
| Breaking change                 | major `X` | `0.1.0 → 1.0.0` |

## Steps

1. Bump the version in [`pyproject.toml`](pyproject.toml):
   ```toml
   version = "0.2.0"
   ```
2. Commit and push to `main`:
   ```bash
   git add pyproject.toml
   git commit -m "Bump version to 0.2.0"
   git push origin main
   ```
3. Tag the release and push the tag (this triggers the publish workflow):
   ```bash
   git tag v0.2.0
   git push origin v0.2.0
   ```
4. Watch the run on the repo's **Actions** tab. A green check means it is live on
   PyPI: <https://pypi.org/project/sr-opinf/>.
5. Verify in a clean virtual environment:
   ```bash
   python3 -m venv /tmp/test-sropinf
   source /tmp/test-sropinf/bin/activate
   pip install --upgrade sr-opinf
   python -c "import SROpInf; print('OK')"
   deactivate && rm -rf /tmp/test-sropinf
   ```

## Will my new files be included?

Packaging is driven by `[tool.setuptools.packages.find]` in `pyproject.toml`
(`where = ["src"]`, `include = ["SROpInf*"]`):

- **New `.py` modules under `src/SROpInf/`** (including new sub-packages that have
  an `__init__.py`) are picked up **automatically** — no config change needed.
- **A new top-level package** outside `SROpInf` (e.g. `src/other/`) does *not*
  match `include = ["SROpInf*"]`; extend `include` to add it.
- **Non-`.py` data files** (e.g. `.json`, `.csv`) are *not* shipped by default;
  declare them under `[tool.setuptools.package-data]` (and/or a `MANIFEST.in` for
  the sdist).

The `example/` notebooks and `output/` live outside `src/` on purpose and are
**not** part of the installed package — clone the repository to run the
experiments.
