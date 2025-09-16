# GitHub Action for Swift armhf (armv6 & armv7)

This repo contains a GitHub action to download and install Swift built for Linux armhf (armv6 and armv7) distributions. This action is easily integrated into an existing CI and provides destination files that can be used to cross-compile any Swift package for the desired target.

The SDKs that are built and hosted on [swift-armhf](https://github.com/xtremekforever/swift-armhf) are downloaded and installed. The following versions are supported:

| Architecture | Swift Versions          | Supported Distributions                                                |
|--------------|-------------------------|------------------------------------------------------------------------|
| armv6        | Swift 6.0 and later     | `raspios-bullseye` `raspios-bookworm` `raspios-trixie`                 |
| armv7        | Swift 5.10 and later    | `debian-bullseye` `debian-bookworm` `debian-trixie` `ubuntu-focal` `ubuntu-jammy` `ubuntu-noble` `raspios-bullseye` `raspios-bookworm` `raspios-trixie` |

The action is compatible with and has been tested to work with the following runners/environments:

- container: `swift:<swift_version>`
- x86_64 runners: `ubuntu-20.04`, `ubuntu-22.04`, `ubuntu-24.04`, `ubuntu-latest`
- arm64 runners: `ubuntu-22.04-arm`, `ubuntu-24.04-arm`

Notes:

1. In order to use the SDKs, the version of Swift that is installed on the host **MUST** match the version of the SDK that is installed. I.e., ensure that Swift `6.1.2` is installed in the host when installing any `6.1.2` SDK for armhf.
2. This action requires root access to install the `wget` dependency for downloading the SDK, and root access for installing the SDK to `/opt`.

## Usage

To start using the action, include `swift-embedded-linux/armhf-action@main` in a step:

```yml
    - uses: swift-embedded-linux/armhf-action@main
      name: Install armhf SDK
      id: armhf-sdk
      with:
        swift-version: 6.2
        distribution: debian-bookworm
        arch: armv7
```

See [action.yml](action.yml) for a complete list of supported Swift versions and distributions.

> [!WARNING]  
> Cross-compiling Swift modules is broken when `.Cxx` interop is enabled due to this bug: https://github.com/swiftlang/swift/issues/83915. Some targets may work, but if you need C++ interop you may want to skip this release.

The action provides the following outputs that can be used from other steps:

- `sdk_name`: Name of the SDK that is installed to /opt.
- `destination_json`: The destination.json file to use to dynamically cross-compile.
- `destination_static_json`: The destination.json file to use to statically cross-compile.

To cross-compile a Swift package dynamically using the the `destination_json` file, use it like this:

```yml
    - name: Build (Include Tests)
      run: swift build --build-tests --destination ${{ steps.armhf-sdk.outputs.destination_json }}
```

Static linking can only be done against executable or library targets. Both the `XCTest` and `Testing`
modules are _NOT_ statically compiled in the provided armhf SDKs, so tests cannot be built using static linking.
To statically link a target, use `destination_static_json`:

```yml
    - name: Static Build
      run: swift build -c release --destination ${{ steps.armhf-sdk.outputs.destination_static_json }} --static-swift-stdlib
```
