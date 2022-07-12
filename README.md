# go-actions

This repository provides general-purpose actions for Go.


## setup [![setup](https://github.com/int128/go-actions/actions/workflows/setup.yaml/badge.svg)](https://github.com/int128/go-actions/actions/workflows/setup.yaml)

This action runs `actions/setup-go` with `actions/cache`.

For example,

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v2
      - uses: int128/go-actions/setup@v1
        with:
          go-version: 1.18.4
      - uses: golangci/golangci-lint-action@v3
        with:
          version: v1.46.2

  test:
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v2
      - uses: int128/go-actions/setup@v1
        with:
          go-version: 1.18.4
      - run: go test -v -race ./...
```


## release [![release](https://github.com/int128/go-actions/actions/workflows/release.yaml/badge.svg)](https://github.com/int128/go-actions/actions/workflows/release.yaml)

This action builds an archive and digest, and releases them into GitHub Releases on tag event.

- `NAME_GOOS_GOARCH.zip`
- `NAME_GOOS_GOARCH.zip.sha256`

For example,

```yaml
name: release

on:
  push:
    branches:
      - master
    paths:
      - .github/workflows/release.yaml
      - '**.go'
      - go.*
    tags:
      - v*
  pull_request:
    branches:
      - master
    paths:
      - .github/workflows/release.yaml
      - '**.go'
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
            GOOS: linux
            GOARCH: arm
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
      - uses: actions/checkout@v2
      - uses: int128/go-actions/setup@v1
        with:
          go-version: 1.18.4
      - run: go build -ldflags "-X main.version=${GITHUB_REF##*/}"
      - uses: int128/go-actions/release@v1
        with:
          binary: example
```


## Renovate config

This provides a general-purpose config for Renovate.
See [`default.json`](default.json) for details.

```json
{
  "extends": [
    "config:base",
    "github>int128/go-actions",
  ],
}
```
