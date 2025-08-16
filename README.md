# OpenWrt GitHub Action Image Builder

GitHub CI action to build images via Image Builder using official OpenWrt
Image Builder Docker containers.

## Example usage

The following YAML code can be used to build all packages of a repository and
store created image files as artifacts.

```yaml
name: Test Build

on:
  push: {}
  pull_request: {}

jobs:
  build:
    name: build ${{ matrix.release }} ${{ matrix.target }}
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        release:
          - SNAPSHOT
          - 24.10.2
        target:
          - x86-64
          - mediatek-filogic
        include:
          - target: x86-64
            device: generic
            distrib_arch: x86_64
          - target: mediatek-filogic
            device: zyxel_nwa50ax-pro openwrt_one
            distrib_arch: aarch64_cortex-a53

    steps:
      - uses: actions/checkout@v4

      - name: create files
        run: |
          mkdir -p files
          date > files/buildtime

      - name: Build
        uses: herbetom/openwrt-gh-action-imagebuilder@v1
        with:
          release: ${{ matrix.release }}
          target-name: ${{ matrix.target }}
          profile: ${{ matrix.device }}
        env:
          PACKAGES: nano
          FILES_DIR: "${{ github.workspace }}/files"
        id: build

      - uses: actions/upload-artifact@v4
        with:
          name: ${{ matrix.release }}-${{ matrix.target }}-${{ matrix.device }}
          path: |
            bin/targets/${{ steps.build.outputs.target }}/${{ steps.build.outputs.subtarget }}/
```

## Action Inputs

* `release`: the release to use (e.g. main, 24.10.2, openwrt-24.10)
* `target-name`: the target to use (e.g. x86-64)
* `profile`: The profile to generate the image for (e.g. generic). You can seperate multiple with a white space.


## Environmental variables

The action reads a few env variables:

* `PACKAGES` packages to include. You can seperate multiple with a white space.
* `FILES_DIR` location of a dir of files to include in the finished images.
* `EXTRA_IMAGE_NAME` add this to the output image filename (sanitized)
* `DISABLED_SERVICES` which services in /etc/init.d/ should be disabled
* `ADD_LOCAL_KEY` store locally generated signing key in built images
* `ROOTFS_PARTSIZE` override the default rootfs partition size in MegaBytes
* `NO_DEFAULT_REPOS` clean out the default repositories
* `CUSTOM_REPO` the content will be prepended in the repositories file
* `NO_DEFAULT_KEYS` clean out the default keys
* `KEYS_DIR` location of a dir of files with keys.
