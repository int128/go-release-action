# go-release-action

This is a composite action to publish a Go binary into GitHub Releases.

## Getting Started

Here is an example workflow.

```yaml
name: release

on:
  push:
    branches:
      - master
    paths:
      - .github/workflows/release.yaml
      - "**.go"
      - go.*
    tags:
      - v*
  pull_request:
    branches:
      - master
    paths:
      - .github/workflows/release.yaml
      - "**.go"
      - go.*

jobs:
  build:
    strategy:
      matrix:
        platform:
          - runs-on: ubuntu-latest
            GOOS: linux
            GOARCH: amd64
          - runs-on: ubuntu-latest
            GOOS: linux
            GOARCH: arm64
          - runs-on: ubuntu-latest
            GOOS: darwin
            GOARCH: amd64
          - runs-on: ubuntu-latest
            GOOS: darwin
            GOARCH: arm64
          - runs-on: ubuntu-latest
            GOOS: windows
            GOARCH: amd64
    runs-on: ${{ matrix.platform.runs-on }}
    env:
      GOOS: ${{ matrix.platform.GOOS }}
      GOARCH: ${{ matrix.platform.GOARCH }}
      CGO_ENABLED: 0
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v4
        with:
          go-version: 1.21.5
      - run: go build
      - uses: int128/go-release-action@v2
        with:
          binary: example
```

It will upload the following files to GitHub Releases:

- `example_linux_amd64.zip`
- `example_linux_amd64.zip.sha256`
- `example_linux_arm64.zip`
- `example_linux_arm64.zip.sha256`
- `example_darwin_amd64.zip`
- `example_darwin_amd64.zip.sha256`
- `example_darwin_arm64.zip`
- `example_darwin_arm64.zip.sha256`
- `example_windows_amd64.zip`
- `example_windows_amd64.zip.sha256`

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

| Name                | Description                                                   | Default    |
| ------------------- | ------------------------------------------------------------- | ---------- |
| `binary`            | Filename of the binary (automatically add `.exe` for Windows) | (required) |
| `working-directory` | The working directory                                         | `.`        |
