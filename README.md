# actions-set-product-version
[![Heimdall](https://heimdall.hashicorp.services/api/v1/assets/actions-set-product-version/badge.svg?key=195370081cbf50568fa41066c157e122a929e253ffa3f7e9b377c73433d31061)](https://heimdall.hashicorp.services/site/assets/actions-set-product-version) [![CI](https://github.com/hashicorp/actions-set-product-version/actions/workflows/lint.yml/badge.svg)](https://github.com/hashicorp/actions-set-product-version/actions/workflows/lint.yml)

## Description

`actions-set-product-version` is a GitHub action that acts as a bridge between the product repository and our new CRT feature: [automated version bumping](https://github.com/hashicorp/bob/commit/6813d9757c644679193a0af317e99570ac8cc848). This action should be used in the `build.yml` to parse the `version/VERSION` file that lives in all product repositories.

The following describes what this action does:

- Allows for the static version string from the `version/VERSION` file to be read by the new CRT workflow and automagically be bumped to the next version (whether it be a minor, or patch, or major version bump).
- Outputs an error if there's no `VERSION` file at the specified location
- Outputs an error if there's no version string in the VERSION file
- Is able to parse `product_version` if it is `1.3.0-alpha1` as `1.3.0` (example: when `product_version = 1.3.0-alpha1`, `base_version = 1.3.0`)
- Is able to parse prerelease product versions such as `alpha1` (example `prerelease_product_version = alpha1`) in the statement above.

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `checkout` | No | `true` | Check out the code repository. Can be disabled if checkout is handled by the caller workflow. |
| `version-file` | No | `version/VERSION` | Path to the VERSION file. For multi-product repos, specify the per-product VERSION file path (e.g., `cmd/secrets-kv/VERSION`). |

## Outputs

Note that the `version/VERSION` version should never contain metadata - only release version and prerelease information.
This action has four outputs from parsing the release/prerelease information:

- `product-version` (full product version string: eg. `1.0.0-dev`)
- `base-product-version` (product version stripped of prerelease information: eg. `1.0.0`)
- `prerelease-product-version` (prerelease information `dev`)
- `minor-product-version` (minor version number: eg. `0`)

## Use

### Single-Product Repos (default)

This action should be implemented in product repository `build.yml` files. The action is intended to grab the version from the version file at the beginning of the build, then passes those versions (along with metadata, where necessary) to any workflow jobs that need version information.

```yaml
jobs:
  set-product-version:
    runs-on: ubuntu-latest
    outputs:
      product-version: ${{ steps.set-product-version.outputs.product-version }}
      base-product-version: ${{ steps.set-product-version.outputs.base-product-version }}
      prerelease-product-version: ${{ steps.set-product-version.outputs.prerelease-product-version }}
    steps:
      - name: Set Product version
        id: set-product-version
        uses: hashicorp/actions-set-product-version@v2
        # Reads from version/VERSION by default
```

### Multi-Product Repos
For repositories that contain multiple products (e.g., vault-plugins), use the `version-file` input to specify the path to each product's VERSION file:

```yaml
env:
  PKG_NAME: "secrets-kv"

jobs:
  set-product-version:
    runs-on: ubuntu-latest
    outputs:
      product-version: ${{ steps.set-product-version.outputs.product-version }}
    steps:
      - uses: actions/checkout@v4
      - name: Set Product version
        id: set-product-version
        uses: hashicorp/actions-set-product-version@v2
        with:
          version-file: cmd/${{ env.PKG_NAME }}/VERSION
          # Reads from cmd/secrets-kv/VERSION

  build:
    needs: set-product-version
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building version ${{ needs.set-product-version.outputs.product-version }}"
```

