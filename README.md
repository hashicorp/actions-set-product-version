# actions-set-product-version
[![Heimdall](https://heimdall.hashicorp.services/api/v1/assets/actions-set-product-version/badge.svg?key=195370081cbf50568fa41066c157e122a929e253ffa3f7e9b377c73433d31061)](https://heimdall.hashicorp.services/site/assets/actions-set-product-version) [![CI](https://github.com/hashicorp/actions-set-product-version/actions/workflows/lint.yml/badge.svg)](https://github.com/hashicorp/actions-set-product-version/actions/workflows/lint.yml)

## Description
`actions-set-product-version` is a Github action that acts as a bridge between the product repo and our new CRT feature: [automated version bumping](https://github.com/hashicorp/bob/commit/6813d9757c644679193a0af317e99570ac8cc848). This action should be used in the `build.yml` to parse the `version/VERSION` file that lives in all product repos. 

THe following describes what this action does: 

-  Allows for the static version string from the `version/VERSION` file to be read by the new CRT workflow and automagically be bumped to the next version (whether it be a minor, or patch, or major version bump). 
- Outputs an error if there’s no `VERSION` file in the `version/ dir`
- Outputs an error if there’s no version string in the VERSION file
- Is able to parse `product_version` if it is `1.3.0-alpha1` as `1.3.0` (example: when `product_version = 1.3.0-alpha1`, `base_version = 1.3.0`)
- Is able to parse prerelease product versions such as `alpha1` (example `prerelease_product_version = alpha1`) in the statement above.  

## Outputs
Note that the `version/VERSION` version should never contain metadata - only release version and prerelease information. 
This action has three outputs from parsing the release/prerelease information:
- product-version (full product version string: eg. `1.0.0-dev`)
- base-product-version (product version stripped of prerelease information: eg. `1.0.0`)
- prerelease-product-version (prerelease information `dev`)

## Use
This action should be implemented in product repo `build.yml` files. The action is intended to grab the version from the version file at the beginning of the build, then passes those versions (along with metadata, where necessary) to any workflow jobs that need version information. 

