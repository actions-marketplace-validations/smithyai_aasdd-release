# aasdd-release

A GitHub Action that reads the spec version from an [AASDD](https://github.com/smithyai/aasdd) `spec.md` file and optionally validates it against the current Git tag.

## Usage

```yaml
- uses: smithyai/aasdd-release@v1
  id: spec
  with:
    spec-path: spec.md               # default
    enforce-tag-match: true          # optional, default false

- name: Use spec version
  run: echo "Releasing version ${{ steps.spec.outputs.version }}"
```

## Inputs

| Input | Required | Default | Description |
| --- | --- | --- | --- |
| `spec-path` | No | `spec.md` | Path to the `spec.md` file, relative to the workspace root. AASDD does not prescribe where a spec lives — set this to wherever your `spec.md` resides. |
| `enforce-tag-match` | No | `false` | When `true`, fails if the Git tag does not match the spec version. The tag must be `v<spec-version>` (e.g. tag `v1.2.3` for spec version `1.2.3`). |

## Outputs

| Output | Description |
| --- | --- |
| `version` | The spec version parsed from `spec.md` (e.g. `1.2.3`, without a leading `v`). |

## Example: full release workflow with GoReleaser

```yaml
name: Release

on:
  push:
    tags:
      - "v*"

permissions:
  contents: write

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: smithyai/aasdd-release@v1
        id: spec
        with:
          enforce-tag-match: true

      - uses: actions/setup-go@v5
        with:
          go-version-file: go.mod

      - uses: goreleaser/goreleaser-action@v6
        with:
          version: "~> v2"
          args: release --clean
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Spec version format

The action reads the first line matching `**Version:** X.Y.Z` from the specified `spec.md` file and validates that it is well-formed semver (`X.Y.Z`). If the line is missing or malformed, the action fails with a descriptive error.
