# Github CI Workflows for Python Packages

My collection of github [reusable workflows and composite actions](
  https://docs.github.com/en/actions/sharing-automations/avoiding-duplication#about-reusable-workflows-and-composite-actions)
for checking, testing, publishing and releasing
python packages. I have put these in this repo so I can re-use them in my python projects.

These workflows use the [`uv`](https://docs.astral.sh/uv/) python package tool
for managing the project virtual environments, running tests and building and
publishing python packages. All tests and publishing workflows run substantially
more quickly with `uv`.

## Usage Examples

To get started, copy [`examples/ci.yaml`](./examples/ci.yaml) to the
`.github/workflows/` directory of your project repo and adjust according to the
project needs. Configure the CI workflow by editting the `env` variables which
select the workflow depending on what event triggered the workflow.

## Example pyproject file

- [`examples/pyproject.yaml`](./examples/pyproject.toml):

  An example `pyproject.toml` file to use with the github workflow files:
  - Uses `hatch-vcs` to dynamically update the project version (in `_version.py`
    file) based on git tags
  - Compatible `tox` configuration to run tests using: `uv run tox`
  - Default configs for `uv`, `mypy`, `ruff` and `pytest`.
  - Assumes:
    - project python code is in the `src` sub-directory
    - pytest-based tests are in the `tests` sub-directory

  To ensure that the `_version.py` file is updated with a new version after
  every commit or checkout, add this command to the `.git/hooks/post-commit` and
  `.git/hooks/post-checkout` files:
  - `uv run hatch build --hooks-only > /dev/null`

## Reusable Github Workflows provided

- [`.github/workflows/test-tox.yaml`](.github/workflows/test-tox.yaml):

  Run CI test workflows on the project. Uses `tox` to run the ci tests; and
  `uv` to manage the python versions and virtual env. Assumes `tox`
  configuration is completely provided in configuration files and the CI
  workflow is configured in the `tox-gh` config (eg. `pyproject.toml`).
  (See <https://github.com/tox-dev/tox-gh>).

  Input options:
  - `os`: a json string containing the list of operating systems on which to
    run tests.
    - Default is `'["ubuntu-latest", "windows-latest", "macos-latest"]'`.
  - `python-version`: a json string containing the list of python versions on
    which to run tests.
    - Default is `'["3.9", "3.10", "3.11", "3.12", "3.13"]'`.

  Invoke with: `uses: glenn20/python-ci/.github/workflows/test-tox.yaml@v2`

- [`.github/workflows/code-check.yaml`](.github/workflows/code-check.yaml):

  Run code checks on the codebase. Uses `mypy` for type checking and `ruff` for
  linting and format checking. Assumes `mypy` and `ruff` configuration is
  completely provided in configuration files (eg. `pyproject.toml`).

  This workflow is unnecessary if using `test-tox.yaml` and the static code
  checks are configured via `tox-gh`.

  Invoke with: `uses: glenn20/python-ci/.github/workflows/code-check.yaml@v2`

- [`.github/workflows/test-pytest.yaml`](.github/workflows/test-pytest.yaml):

  Run CI test workflows on the project. Uses `pytest` to run the ci tests; and
  `uv` to manage the python versions and virtual env. Assumes `pytest`
  configuration is completely provided in configuration files (eg.
  `pyproject.toml`).

  Input options:
  - `os`: a json string containing the list of operating systems on which to
    run tests.
    - Default is `'["ubuntu-latest", "windows-latest", "macos-latest"]'`.
  - `python-version`: a json string containing the list of python versions on
    which to run tests.
    - Default is `'["3.9", "3.10", "3.11", "3.12", "3.13"]'`.

  Invoke with: `uses: glenn20/python-ci/.github/workflows/test-pytest.yaml@v2`

- [`.github/workflows/build.yaml`](.github/workflows/build.yaml):

  Build the python package (using `uv build`) and upload as a workflow artifact
  to the github repo.

  Invoke with `uses: glenn20/python-ci/.github/workflow/build.yaml@v1`

- [`.github/workflows/github-release.yaml`](.github/workflows/github-release.yaml):

  Use [sigstore](https://github.com/sigstore/gh-action-sigstore-python) to sign
  the python package files and upload to a GitHub Release. Assumes the python
  package has been saved as a workflow artifact. Adapted from the [PYPA Python
  Packaging User Guide](
  https://packaging.python.org/en/latest/guides/publishing-package-distribution-releases-using-github-actions-ci-cd-workflows/#signing-the-distribution-packages
  )

  Input options:
  - `tag: tagname`: name of the release. Defaults to the current git tag if the
    workflow is triggered by a tag push, else the version number from the python
    package wheel file.

  Invoke with `uses: glenn20/python-ci/.github/workflow/github-release.yaml@v2`

## Github Actions provided

- [`actions/publish/action.yaml`](actions/publish/action.yaml):

  A github action to publish a python package to <https://pypi.org> or
  <https://pypi.org>. The publish workflow can't support [pypi trusted
  publishing](https://docs.pypi.org/trusted-publishers/) when run as a reusable
  workflow from an external repo, so it is provided here as a [github composite
    action](
    https://docs.github.com/en/actions/sharing-automations/avoiding-duplication#about-reusable-workflows-and-composite-actions).

  Input options:
  - `test-only` - Only publish to <https://test.pypi.org> if set 'true' (default)

  Outputs:
  - `package-name`: the name of the package file published
  - `package-version`: the version number of the package, eg. "1.3.2"

  Invoke with: `uses: glenn20/python-ci/actions/publish@v2`

- [`actions/setup/action.yaml`](actions/setup/action.yaml):

  Install `uv` and python and setup the python virtual environment (venv).

  This action is used internally by the reusable workflows above.

  Input options:
  - `python-version:` set the python version to install (default
    "3.12").

  Invoke with: `uses: glenn20/python-ci/actions/setup@v2`
