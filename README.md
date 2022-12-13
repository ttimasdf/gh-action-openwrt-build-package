# OpenWrt GitHub Action SDK

GitHub CI action to build packages via SDK using official OpenWrt SDK Docker
containers.

## Example usage

### Multi-dimension build matrix for multiple architectures and OpenWrt releases

The following YAML snippet builds ipk packages for two architectures
on OpenWrt 22.03.2 and 21.02.5, then upload packages as artifects.

```yaml
name: Build Packages

on:
  pull_request:
    branches:
      - master
  push:

jobs:
  build:
    name: Build on ${{ matrix.version }} for ${{ matrix.arch }}
    runs-on: ubuntu-latest
    strategy:
      matrix:
        arch: [x86-64, arm_cortex-a15_neon-vfpv4]
        version: [22.03.2, 21.02.5]

    steps:
      - uses: actions/checkout@v2
        with:
          fetch-depth: 0

      - name: Build ipk
        uses: ttimasdf/gh-action-openwrt-build-package@master
        env:
          ARCH: ${{ matrix.arch }}-${{ matrix.version }}

      - name: Upload ipk packages
        uses: actions/upload-artifact@v2
        with:
          name: ${{ matrix.arch }}-${{ matrix.version }}-package
          # path: bin/packages/${{ matrix.arch }}/action/*.ipk
          path: bin/packages/*/action/*.ipk
```

### Single-dimension build matrix on OpenWrt snapshot

The following YAML code can be used to build packages (on OpenWrt snapshot)
for the repository and store created `ipk` files as artifacts.

```yaml
name: Test Build

on:
  pull_request:
    branches:
      - main

jobs:
  build:
    name: ${{ matrix.arch }} build
    runs-on: ubuntu-latest
    strategy:
      matrix:
        arch:
          - x86_64
          - mips_24kc

    steps:
      - uses: actions/checkout@v2
        with:
          fetch-depth: 0

      - name: Build
        uses: ttimasdf/gh-action-openwrt-build-package@master
        env:
          ARCH: ${{ matrix.arch }}

      - name: Store packages
        uses: actions/upload-artifact@v2
        with:
          name: ${{ matrix.arch }}-packages
          path: bin/packages/*/action/*.ipk
```

## Environmental variables

The action reads a few env variables:

* `ARCH` determines the used OpenWrt SDK Docker container.
  E.g. `x86_64` or `x86_64-22.03.2`.
* `ARTIFACTS_DIR` determines where built packages and build logs are saved.
  Defaults to the default working directory (`GITHUB_WORKSPACE`).
* `BUILD_LOG` stores build logs in `./logs`.
* `CONTAINER` can set other SDK containers than `openwrt/sdk`.
* `EXTRA_FEEDS` are added to the `feeds.conf`, where `|` are replaced by white
  spaces.
* `FEED_DIR` used in the created `feeds.conf` for the current repo. Defaults to
  the default working directory (`GITHUB_WORKSPACE`).
* `FEEDNAME` used in the created `feeds.conf` for the current repo. Defaults to
  `action`.
* `IGNORE_ERRORS` can ignore failing packages builds.
* `INDEX` makes the action build the package index. Default is 0. Set to 1 to enable.
* `KEY_BUILD` can be a private Signify/`usign` key to sign the packages feed.
* `NO_DEFAULT_FEEDS` disable adding the default SDK feeds
* `NO_REFRESH_CHECK` disable check if patches need a refresh.
* `NO_SHFMT_CHECK` disable check if init files are formated
* `PACKAGES` (Optional) specify the list of packages (space separated) to be built
* `V` changes the build verbosity level.
