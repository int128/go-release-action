# go-actions

This repository provides general-purpose actions for Go.


## setup [![setup](https://github.com/int128/go-actions/actions/workflows/setup.yaml/badge.svg)](https://github.com/int128/go-actions/actions/workflows/setup.yaml)

This action runs `actions/setup-go` and `actions/cache`.

For example,

```yaml
jobs
  lint:
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v2
      - uses: int128/go-actions/setup@v1
        with:
          go-version: 1.17
      - uses: golangci/golangci-lint-action@v2
        with:
          version: v1.38.0
          skip-go-installation: true
          skip-pkg-cache: true
          skip-build-cache: true

  test:
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v2
      - uses: int128/go-actions/setup@v1
        with:
          go-version: 1.17
      - run: go test -v -race ./...
```


## release [![release](https://github.com/int128/go-actions/actions/workflows/release.yaml/badge.svg)](https://github.com/int128/go-actions/actions/workflows/release.yaml)

This action builds an archive and digest, and releases them into GitHub Releases on tag event.

- `NAME_GOOS_GOARCH.zip`
- `NAME_GOOS_GOARCH.zip.sha256`

For example,

```yaml
jobs:
  build:
    strategy:
      matrix:
        platform:
          - runs-on: ubuntu-latest
            GOOS: linux
            GOARCH: amd64
          - runs-on: ubuntu-latest
            GOOS: darwin
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
          go-version: 1.17
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
