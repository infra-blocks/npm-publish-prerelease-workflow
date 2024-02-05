# npm-publish-prerelease-workflow
[![Git Tag Semver From Label](https://github.com/infrastructure-blocks/npm-publish-prerelease-workflow/actions/workflows/git-tag-semver-from-label.yml/badge.svg)](https://github.com/infrastructure-blocks/npm-publish-prerelease-workflow/actions/workflows/git-tag-semver-from-label.yml)
[![Update From Template](https://github.com/infrastructure-blocks/npm-publish-prerelease-workflow/actions/workflows/update-from-template.yml/badge.svg)](https://github.com/infrastructure-blocks/npm-publish-prerelease-workflow/actions/workflows/update-from-template.yml)

This workflow publishes prerelease NPM packages based the provided increment type input. The supported input
values are "premajor", "preminor" and "prepatch".

The workflow checkouts the code, calculate what the next package version should be based on the input and publishes
that package.

The calculation assumes that the version in the package is the most recent release (what's on the main release branch).
It does not attempt to modify the package.json file. It generates the lineage of the prereleases using a semantic
versioning increment on the package.json's version. It figures out the prerelease prefix by running `npm get preid`.
For example, if the package.json's version is "0.1.5", the prerelease prefix is "alpha", and the type input
is "preminor", then the prerelease lineage is "0.2.0-alpha.X" and it starts at "0.2.0-alpha.0".

In the event that the lineage does not exist on the NPM registry, then the first lineage version will be the one
used in publication. That's "0.2.0-alpha.0" in our example.

When the lineage does exist, then we find the latest version and run a semantic prerelease increment on it. The
result version is the one used for publication. For example, if the latest prerelease version is "0.2.0-alpha.12",
then the version bump will result in "0.2.0-alpha.13".

## Inputs

|   Name    | Required | Description                                                                                                                                                   |
|:---------:|:--------:|---------------------------------------------------------------------------------------------------------------------------------------------------------------|
|   type    |   true   | The prerelease type. Either "premajor", "preminor", or "prepatch"                                                                                             |
| dist-tags |   true   | A stringified JSON array of the dist-tags to apply. Because we are publishing prereleases, this is required and it *shouldn't include the "latest" dist-tag*. |

## Secrets

|   Name    | Required | Description                                                                                                                                                                      |
|:---------:|:--------:|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| npm-token |   true   | The NPM token used to publish the package. This is passed as the NPM_TOKEN environment variable. See [ts-lib-template](https://github.com/infrastructure-blocks/ts-lib-template) |

## Outputs

|     Name     | Description                                                                                                                                                   |
|:------------:|---------------------------------------------------------------------------------------------------------------------------------------------------------------|
| package-name | The name of the processed package. Example: @infra-blocks/lib-template                                                                                        |
|   version    | The version of the released package. Example: 1.2.3-alpha.5                                                                                                   |
|  dist-tags   | The stringified JSON array of dist-tags applied. This will match the input.                                                                                   |
|    links     | A stringified JSON array of markdown links to the released package's registry versions of the form:<br/> ["\[<package-identifier>\](<version-registry-url>)"] |

## Permissions

|  Scope   | Level | Reason                |
|:--------:|:-----:|-----------------------|
| contents | read  | To checkout the code. |

## Concurrency controls

N/A

## Timeouts

N/A

## Usage

```yaml
name: NPM Publish Prerelease

on:
  push:
    branches:
      - "patch/**"

permissions:
  contents: read

jobs:
  npm-publish-prerelease:
    uses: infrastructure-blocks/npm-publish-prerelease-workflow/.github/workflows/workflow.yml@v1
    permissions:
      contents: read
    with:
      type: prepatch
      dist-tags: '["git-sha-${{ github.sha }}"]'
    secrets:
      npm-token: ${{ secrets.NPM_PUBLISH_TOKEN }}
```

### Releasing

The releasing is handled at git level with semantic versioning tags. Those are automatically generated and managed
by the [git-tag-semver-from-label-workflow](https://github.com/infrastructure-blocks/git-tag-semver-from-label-workflow).
