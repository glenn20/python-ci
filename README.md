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

To get started, copy these three files to the `.github/workflows/` directory of
the project repo and adjust according to the project needs.

- [`examples/ci-tests.yaml`](./examples/ci-tests.yaml):

  When a commit is pushed to the `main` or `dev` branches, call the
  `check-test-build` workflow (see below) to:
  - Run code checks with `mypy` and `ruff`;
  - Run an abbreviated CI pytest workflow using `pytest`;
  - Build a python package and upload to github as a workflow artifact; and
  - Publish to <https://test.pypi.org> (if pushing to the `main` branch).

  Can also be run from a manual `workflow_dispatch` event (eg. from the github
  "Actions" tab).

- [`examples/ci-release.yaml`](./examples/ci-release.yaml):

  When a git `tag` is pushed (eg. `git tag -a -m v0.3.5 v0.3.5; git push
  --tags`):

  - Run code checks and comprehensive CI test suite;
  - Build and publish to <https://test.pypi.org> and <https://pypi.org>; and
  - Create a github release and upload the signed python package files.

  Can also be run from a manual `workflow_dispatch` event, which will produce a
  release with a version tag like "v0.3.5.devN".

- [`examples/publish.yaml`](./examples/publish.yaml):

  A callable (reusable) workflow to publish a python package to
  <https://pypi.org> or <https://test.pypi.org>. For trusted publishing, this
  workflow can not be called from this repo, but must be copied to your
  repository and called from your own top-level workflow files (eg.
  [`examples/ci-tests.yaml`](./examples/ci-tests.yaml)).

  Input options:

  - `pypi: test.pypi`: Publish to test.pypi.org (default).
  - `pypi: upload.pypi`: Publish to pypi.org.

  Requirements (to support [pypi trusted
  publishing](https://docs.pypi.org/trusted-publishers/)):

  1. Copy this workflow file to `.github/workflows/publish.yaml` in your
     repository.
  2. Create the `publish-test.pypi` and `publish-upload.pypi` Environments in
     your github repository (Settings->Environments->New Environment).
  3. Add this workflow file as a "trusted publisher" on your pypi and test.pypi
     project pages
     - add the name of the relevant Environment for additional access control.
  4. Call this workflow from a parent workflow with the `pypi` input set to
     "upload.pypi" or "test.pypi" (default).

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
  - `uv run --frozen hatch build --hooks-only > /dev/null`

## Reusable Github Workflows provided

- [`.github/workflows/check.yaml`](.github/workflows/check.yaml):

  Run code checks on the codebase. Uses `mypy` for type checking and `ruff` for
  linting and format checking. Assumes `mypy` and `ruff` configuration is
  completely provided in configuration files (eg. `pyproject.toml`).

  **This workflow is called by `test.yaml`.**

  Invoke with: `uses: glenn20/python-ci/.github/workflows/check.yaml@v1`

- [`.github/workflows/test.yaml`](.github/workflows/test.yaml):

  Run CI test workflows on the project. Uses `pytest` to run the c tests; and
  `uv` to manage the python versions and virtual env. Assumes `pytest`
  configuration is completely provided in configuration files (eg.
  `pyproject.toml`).

  Input options:
  - `os`: a json string containing the list of operating systems on which to
    run tests.
    - Default is `["ubuntu-latest", "windows-latest", "macos-latest"]`.
  - `python-version`: a json string containing the list of python versions on
    which to run tests.
    - Default is `["3.9", "3.10", "3.11", "3.12", "3.13"]`.

  Invoke with: `uses: glenn20/python-ci/.github/workflows/test.yaml@v1`

- [`.github/workflows/build.yaml`](.github/workflows/build.yaml):

  Build the python package and upload as a workflow artifact to the github repo.
  Makes the following outputs available for use by other workflow jobs (eg. used
  in the [`examples/publish.yaml`](./examples/publish.yaml) workflow):

  - `package-name`: Name of the package.
  - `package-version`: Version number of the package.

  Invoke with `uses: glenn20/python-ci/.github/workflow/build.yaml@v1`

- [`.github/workflows/check-test-build.yaml`](.github/workflows/check-test-build.yaml):

  Combine the `check`, `test` and `build` workflows into a single workflow.

  Input options: (passed to `test.yaml` workflow above)
  - `os`: a json string containing the list of operating systems on which to
    run tests.
    - Default is `["ubuntu-latest", "windows-latest", "macos-latest"]`.
  - `python-version`: a json string containing the list of python versions on
    which to run tests.
    - Default is `["3.9", "3.10", "3.11", "3.12", "3.13"]`.

  Invoke with: `uses: glenn20/python-ci/.github/workflows/check-build-test.yaml@v1`

- [`.github/workflows/release.yaml`](.github/workflows/release.yaml):

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

  Invoke with `uses: glenn20/python-ci/.github/workflow/release.yaml@v1`

## Github Actions provided

- [`publish/action.yaml`](publish/action.yaml):

  A github action to publish a python package to <https://pypi.org> or
  <https://pypi.org>. The publish workflow can't support [pypi trusted
  publishing](https://docs.pypi.org/trusted-publishers/) when run as a reusable
  workflow from an external repo, so it is provided here as a [github composite
    action](
    https://docs.github.com/en/actions/sharing-automations/avoiding-duplication#about-reusable-workflows-and-composite-actions).

  Input options:
  - `pypi: test.pypi` - publish to <https://test.pypi.org> (default)
  - `pypi: upload.pypi` - publish to <https://pypi.org>.

  Outputs:
  - `package-name`: the name of the package file published
  - `package-version`: the version number of the package, eg. "1.3.2"

  Invoke with: `uses: glenn20/python-ci/publish@v1`

- [`setup/action.yaml`](setup/action.yaml):

  Install `uv` and python and setup the python virtual environment (venv).

  This action is used internally by the reusable workflows above.

  Input options:
  - `python-version:` set the python version to install (default
    "3.12").

  Invoke with: `uses: glenn20/python-ci/setup@v1`
