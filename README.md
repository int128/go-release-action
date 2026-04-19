# go-release-action [![test](https://github.com/int128/go-release-action/actions/workflows/test.yaml/badge.svg)](https://github.com/int128/go-release-action/actions/workflows/test.yaml)

This is a composite action to publish a Go binary into GitHub Releases.

## Getting Started

Here is an example workflow.

```yaml
jobs:
  build:
    strategy:
      matrix:
        platform:
          - runs-on: ubuntu-latest
            GOOS: linux
            GOARCH: amd64
          - runs-on: macos-latest
            GOOS: darwin
            GOARCH: amd64
          - runs-on: windows-latest
            GOOS: windows
            GOARCH: amd64
    runs-on: ${{ matrix.platform.runs-on }}
    env:
      GOOS: ${{ matrix.platform.GOOS }}
      GOARCH: ${{ matrix.platform.GOARCH }}
    timeout-minutes: 10
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version-file: go.mod
      - run: go build
      - uses: int128/go-release-action@v2
        with:
          binary: example
```

It will upload the following files to GitHub Releases:

- `example_linux_amd64.zip`
- `example_linux_amd64.zip.sha256`
- `example_darwin_amd64.zip`
- `example_darwin_amd64.zip.sha256`
- `example_windows_amd64.zip`
- `example_windows_amd64.zip.sha256`

## Immutable Releases

This action also supports the immutable releases.

Here is an example workflow:

- When you merge a pull request into the main branch, this action creates or updates a draft release with the name `next`.
- If you want to publish a new release, you can rename the draft release to the tag name, e.g. `v1.0.0`, and publish it.
- Alternatively, when you push a tag, this action creates a draft release with the tag name. You can later publish it.

```yaml
name: release

on:
  push:
    branches:
      - main
    paths:
      - .github/workflows/release.yaml
      - "**/*.go"
      - go.*
    tags:
      - v*
  pull_request:
    branches:
      - main
    paths:
      - .github/workflows/release.yaml
      - "**/*.go"
      - go.*

concurrency:
  cancel-in-progress: true
  group: ${{ github.workflow }}--${{ github.ref }}

jobs:
  create:
    runs-on: ubuntu-slim
    timeout-minutes: 10
    outputs:
      draft-release-name: ${{ steps.create-draft-release.outputs.draft-release-name }}
    permissions:
      contents: write
    steps:
      - if: github.event_name == 'push'
        id: create-draft-release
        uses: int128/go-release-action/create-draft-release@v2

  build:
    needs: create
    strategy:
      matrix:
        platform:
          - runs-on: ubuntu-latest
            GOOS: linux
            GOARCH: amd64
          - runs-on: macos-latest
            GOOS: darwin
            GOARCH: amd64
          - runs-on: windows-latest
            GOOS: windows
            GOARCH: amd64
    runs-on: ${{ matrix.platform.runs-on }}
    env:
      GOOS: ${{ matrix.platform.GOOS }}
      GOARCH: ${{ matrix.platform.GOARCH }}
    steps:
      - uses: actions/checkout@v6
      - uses: actions/setup-go@v6
        with:
          go-version-file: go.mod
      - run: go build
      - uses: int128/go-release-action@v2
        with:
          binary: example
          release-name: ${{ needs.create.outputs.draft-release-name }}
```

## Specification

This action assumes the following files exist:

- `${BINARY}` (Linux and macOS)
- `${BINARY}.exe` (Windows)
- `README.md`
- `LICENSE`

It generates the following archives:

- `${BINARY}_${GOOS}_${GOARCH}.zip`
- `${BINARY}_${GOOS}_${GOARCH}.zip.sha256`

### Inputs

| Name                | Description                                                   | Default                     |
| ------------------- | ------------------------------------------------------------- | --------------------------- |
| `binary`            | Filename of the binary (automatically add `.exe` for Windows) | (required)                  |
| `release-name`      | If set, upload the assets to the release                      | Tag name for push tag event |
| `working-directory` | The working directory                                         | `.`                         |
