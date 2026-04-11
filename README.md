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
