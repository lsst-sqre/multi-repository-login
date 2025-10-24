# lsst-sqre/multi-repository-login

A GitHub Actions composite action that uses standard SQuaRE secrets to log into one or more of Github Container Registry, Google Artifact Registry, and Docker Hub in order to push images.

The standard secrets, which must be passed as inputs if used, are:

* `DOCKER_USERNAME`
* `DOCKER_TOKEN`
* `GITHUB_TOKEN`
* `GAR_PUSH_TOKEN`

The action will only authenticate to registries referenced in the ``images`` input, and then only if the corresponding inputs are nonempty.

## Usage

```yaml
name: CI

"on":
  merge_group: {}
  pull_request: {}
  push:
    tags:
      - "*"

jobs:
  build:
    runs-on: ubuntu-latest

    # (optional) only build on tags or ticket branches
    if: >
      startsWith(github.ref, 'refs/tags/')
      || startsWith(github.head_ref, 'tickets/')

    steps:
      - uses: actions/checkout@v3

      - id: image_names
        env:
          repo: ${{ github.repository }}
        shell: bash
        run: |
          reponame=$(echo ${repo} | cut -d '/' -f 2)
          gar_repo="us-central1.docker.pkg.dev/rubin-shared-services-71ec/sciplat/${reponame}"
          ghcr_repo="ghcr.io/${repo}"
          images="${ghcr_repo},${gar_repo}"
          echo "images=${images}" >> ${GITHUB_OUTPUT}

      - uses: lsst-sqre/multi-repository-login@v1
        id: login
        with:
          images: ${{ steps.image_names.outputs.images }}
          GAR_TOKEN: ${{ secrets.GAR_TOKEN }}
          DOCKER_USERNAME: ${{ secrets.DOCKER_USERNAME }}  # Unused
          # First, because no Docker Hub target in images.
          # Also unused because no DOCKER_PASSWORD is set.
```

## Action reference

### Inputs

* `images` (string, required) a string representing the untagged image or images to build. This may be a list in the form of a comma separated string.
  For example, `ghcr.io/lsst-sqre/nublado-jupyterlab-base,us-central1.docker.pkg.dev/rubin-shared-services-71ec/sciplat/jupyterlab-base`.
* `DOCKER_USERNAME` All of these are found in lsst-sqre as org-level secrets.
* `DOCKER_TOKEN`
* `GAR_PUSH_TOKEN`

## Developer guide

This repository provides a **composite** GitHub Action, a type of action that packages multiple regular actions into a single step.
We do this to make the GitHub Actions of all our software projects more consistent and easier to maintain.
[You can learn more about composite actions in the GitHub documentation.](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action)

Create new releases using the GitHub Releases UI and assign a tag with a [semantic version](https://semver.org), including a `v` prefix. Choose the semantic version based on compatibility for users of this workflow. If backwards compatibility is broken, bump the major version.

When a release is made, a new major version tag (i.e. `v1`, `v2`) is also made or moved using [nowactions/update-majorver](https://github.com/marketplace/actions/update-major-version).
We generally expect that most users will track these major version tags.
