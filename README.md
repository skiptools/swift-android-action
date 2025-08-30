# Swift Android native cross-compiler and test runner action

This GitHub action will build and run Swift package tests on an Android emulator.
This uses the [swift-android-toolchain](https://github.com/swift-android-sdk/swift-android-toolchain)
project to provide a cross-compiler for building
Swift natively for Android on a Linux or macOS host.

After building the package, it will run the SwiftPM
test targets on an Android emulator (which is provided by the
[Android Emulator Runner action](https://github.com/marketplace/actions/android-emulator-runner)).
To build the package for Android without running the tests
(which is considerably faster), set the `run-tests` option to `false`.

You can add this action to your Swift CI workflow from the
[GitHub Marketplace](https://github.com/marketplace/actions/swift-android-action),
or you can manually create a workflow by adding a
`.github/workflows/swift-ci.yml` file to your project.
This sample action will run your Swift package's test cases
on a host macOS machine, as well as on an Android emulator
and an iOS simulator.

## Example

The following `ci.yml` workflow will check out the current repository and test the Swift package on Linux and Android:

```yml
name: ci
on:
  push:
    branches:
      - '*'
  workflow_dispatch:
  pull_request:
    branches:
      - '*'
jobs:
  linux-android:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: "Test Swift Package on Linux"
        run: swift test
      - name: "Test Swift Package on Android"
        uses: swift-android-sdk/swift-android-action@v2
```


## Configuration

The following configuration options can be passed to the workflow. For example, to specify the version of Swift you want to use for building and testing, you can use the `swift-version` parameter like so:

```yml
jobs:
  linux-android:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: "Test Swift Package on Linux"
        run: swift test
      - name: "Test Swift Package on Android"
        uses: swift-android-sdk/swift-android-action@v2
        with:
          swift-version: 6.1
```

### Swift Versions

The `swift-version` input can be set to a specific version number (e.g., "6.0.3" or "6.1.1").
A shortened version like "6.0" or "6.1" will use the latest [release](https://github.com/swift-android-sdk/swift-android-toolchain/releases),
which may be a pre-release build.

> [!NOTE]
> The SDK that is installed is currently the unofficial de-facto Android SDK created from
> the [swift-android-sdk](https://github.com/swift-android-sdk/swift-android-sdk) fork.
> As the official Android WG is [formed](https://github.com/swiftlang/swift-org-website/pull/925),
> we anticipate the release of an official Android SDK, at which time this action will
> transition to using that bundle.

Snapshots can be specified with their full name, like `6.2-DEVELOPMENT-SNAPSHOT-2025-06-27-a`,
or the most recent snapshot/nightly build can be specified with `nightly-6.2`.

### Configuration Options

| Parameter | Description | Default  |
|-----|-----|-----|
| swift-version | The version of the Swift toolchain to use | 6.1 |
| ndk-version | The version of the Android NDK to use | <default> |
| package-path | The folder where the swift package is checked out | . |
| swift-configuration | Whether to build with debug or release configuration | debug |
| swift-build-flags | Additional flags to pass to the swift build command |  |
| swift-test-flags | Additional flags to pass to the swift test command |  |
| installed-sdk | The name of a pre-installed Swift SDK to use |  |
| installed-swift | The path to a pre-installed host Swift toolchain |  |
| test-env | Test environment variables key=value |  |
| build-package | Whether to build the Swit package | true |
| build-tests | Whether to build the package tests or just the sources | true |
| run-tests | Whether to run the tests or just perform the build | true |
| copy-files | Additional files to copy to emulator for testing | |
| android-emulator-test-folder | Emulator folder where tests are copied | /data/local/tmp/android-xctest |
| android-api-level | The API level of the Android emulator to run against | 30 |
| android-channel | SDK component channel: stable, beta, dev, canary | stable |
| android-profile | Hardware profile used for creating the AVD | pixel |
| android-target | Target of the system image | aosp_atd |
| android-cores | Number of cores to use for the emulator | 2 |
| android-emulator-options | Options to pass to the Android emulator | -no-window … |
| android-emulator-boot-timeout | Emulator boot timeout in seconds | 600 |

### Platform Support

This action can be run on any of the GitHub `ubuntu-*` and `macos-*` [runner images](https://github.com/actions/runner-images). However, due to the inability of macOS on ARM to run nested virtualization ([issue](https://github.com/ReactiveCircus/android-emulator-runner/issues/350)), the Android emulator cannot be run on these platforms, and so running on any macOS image that uses ARM (including `macos-14` and `macos-15`) requires disabling tests with `run-tests: false`. Running tests are supported on `macos-13`, as well as the large Intel macOS images like `macos-14-large` and `macos-15-large`.

An example of disabling running tests on ARM macOS images is:

```yml
jobs:
  macos-android:
    runs-on: macos-15
    steps:
      - uses: actions/checkout@v4
      - name: "Test Swift Package on macOS"
        run: swift test
      - name: "Build Swift Package on Android"
        uses: swift-android-sdk/swift-android-action@v2
        with:
          run-tests: false
```

### Test Resources and Environment Variables

Unit tests sometimes need to load local resources, such as configuration
files and mocking or parsing inputs. This is typically handled
using [Bundle.module](https://developer.apple.com/documentation/xcode/bundling-resources-with-a-swift-package),
which will be automatically copied up to the Android emulator when
testing and so should work transparently.

However, in some cases a test script may expect a specific set
of local files to be present, such as when a unit test needs to
examine the contents of the source code itself. In these cases,
you can use the `copy-files` input parameter to specify local files
to copy up to the emulator when running.

Similarly, some test cases may expect a certain environment to be set.
This can be accomplished using the `test-env` parameter, which
is a space separated list of key=value pairs of environment
variables to set when running the tests.

For example:

```yml
  - name: Test Android Package
    uses: swift-android-sdk/swift-android-action@v2
    with:
      copy-files: Tests
      test-env: TEST_WORKSPACE=1
```

### Installing without Building

You may wish to use this action to just install the toolchain and
Android SDK without performing the build. For example, if you
wish to build multiple packages consecutively, and don't need to
run the test cases. In this case, you can set the
`build-package` input to false, and then use the action's
`swift-build` output to get the complete `swift build` command
with all the appropriate paths and arguments to build using the
SDK.

For example:

```yml
  - name: Setup Toolchain
    id: setup-toolchain
    uses: swift-android-sdk/swift-android-action@v2
    with:
      # just set up the toolchain but don't build anything
      build-package: false
  - name: Checkout apple/swift-numerics
    uses: actions/checkout@v4
  - name: Build Package With Toolchain
    run: |
      # build twice, once with debug and once with release
      ${{ steps.setup-toolchain.outputs.swift-build }} -c debug
      ls .build/${{ steps.setup-toolchain.outputs.swift-sdk }}/debug
      ${{ steps.setup-toolchain.outputs.swift-build }} -c release
      ls .build/${{ steps.setup-toolchain.outputs.swift-sdk }}/release
```

The actual `swift-build` command will vary between operating systems
and architectures. For example, on Ubuntu 24.04, it might be:

```
/home/runner/swift/toolchains/swift-6.1-RELEASE/usr/bin/swift build --swift-sdk x86_64-unknown-linux-android24
```

while on macOS-15 it will be:

```
/Users/runner/Library/Developer/Toolchains/swift-6.1-RELEASE.xctoolchain/usr/bin/swift build --swift-sdk aarch64-unknown-linux-android24
```


## Complete Universal CI Example

Following is an example of a `ci.yml` workflow that checks out and tests a Swift package on each of macOS, iOS, Linux, Android, and Windows.

```yml
name: swift-algorithms ci
on:
  push:
    branches:
      - '*'
  workflow_dispatch:
  pull_request:
    branches:
      - '*'
jobs:
  linux-android:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: "Test Swift Package on Linux"
        run: swift test
      - name: "Test Swift Package on Android"
        uses: swift-android-sdk/swift-android-action@v2
  macos-ios:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v4
      - name: "Test Swift Package on macOS"
        run: swift test
      - name: "Test Swift Package on iOS"
        run: xcodebuild test -sdk "iphonesimulator" -destination "platform=iOS Simulator,name=$(xcrun simctl list devices --json | jq -r '.devices | to_entries[] | .value[] | select(.availability == "(available)" or .isAvailable == true) | .name' | grep -E '^iPhone [0-9]+$' | sort -V | tail -n 1)" -scheme "$(xcodebuild -list -json | jq -r '.workspace.schemes[-1]')"
  windows:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4
      - name: "Setup Swift on Windows"
        uses: compnerd/gha-setup-swift@main
        with:
          branch: swift-6.1-release
          tag: 6.1-RELEASE
      - name: "Test Swift Package on Windows"
        run: swift test

```

For an example of this workflow in action, see a run history
for the [swift-sqlite](https://github.com/swift-android-sdk/swift-sqlite/actions) package.

## Development

### Releasing new versions

To create a new release, make a new tag (like 2.0.2),
and then update the symbolic major v2 tag with:

```
git tag v2.0.2 && git push --tags
git tag -fa v2 -m "Update v2 tag" && git push origin v2 --force
gh release create --generate-notes --latest
```

## Who is using the Swift Android Action?

These are some of the open-source projects using (or used) the Swift Android Action:

- [Alamofire](https://github.com/Alamofire/Alamofire/tree/master/.github/workflows)
- [Bagbutik](https://github.com/MortenGregersen/Bagbutik/tree/main/.github/workflows)
- [BigInt](https://github.com/attaswift/BigInt/tree/master/.github/workflows)
- [CryptoSwift](https://github.com/krzyzanowskim/CryptoSwift/tree/main/.github/workflows)
- [egeniq/app-remote-config](https://github.com/egeniq/app-remote-config/tree/main/.github/workflows)
- [Forked](https://github.com/drewmccormack/Forked/tree/main/.github/workflows)
- [GraphQL](https://github.com/GraphQLSwift/GraphQL/tree/main/.github/workflows)
- [jose-swift](https://github.com/beatt83/jose-swift/tree/main/.github/workflows)
- [MemoZ](https://github.com/marcprux/MemoZ/tree/main/.github/workflows)
- [PromiseKit](https://github.com/mxcl/PromiseKit/tree/master/.github/workflows)
- [ReactiveKit](https://github.com/DeclarativeHub/ReactiveKit/tree/master/.github/workflows)
- [Skip](https://github.com/skiptools/actions/tree/main/.github/workflows)
- [soto-core](https://github.com/soto-project/soto-core/tree/main/.github/workflows)
- [SQLite.swift](https://github.com/stephencelis/SQLite.swift/tree/master/.github/workflows)
- [supabase-swift](https://github.com/supabase/supabase-swift/tree/main/.github/workflows)
- [swift-android-native](https://github.com/skiptools/swift-android-native/tree/main/.github/workflows)
- [SwiftCBOR](https://github.com/valpackett/SwiftCBOR/tree/master/.github/workflows)
- [SwiftDraw](https://github.com/swhitty/SwiftDraw/tree/main/.github/workflows)
- [swift-fakes](https://github.com/Quick/swift-fakes/tree/main/.github/workflows)
- [SwiftGodot](https://github.com/migueldeicaza/SwiftGodot/tree/main/.github/workflows)
- [swift-issue-reporting](https://github.com/pointfreeco/swift-issue-reporting/tree/main/.github/workflows)
- [swift-jni](https://github.com/skiptools/swift-jni/tree/main/.github/workflows)
- [swift-macro-testing](https://github.com/pointfreeco/swift-macro-testing/tree/main/.github/workflows)
- [swift-png](https://github.com/tayloraswift/swift-png/tree/master/.github/workflows)
- [swift-snapshot-testing](https://github.com/pointfreeco/swift-snapshot-testing/tree/main/.github/workflows)
- [swift-sqlcipher](https://github.com/skiptools/swift-sqlcipher/tree/main/.github/workflows)
- [SwifterSwift](https://github.com/SwifterSwift/SwifterSwift/tree/master/.github/workflows)
- [universal](https://github.com/marcprux/universal/tree/main/.github/workflows)
- [vapor/ci](https://github.com/vapor/ci/tree/main/.github/workflows)
- [WasmKit](https://github.com/swiftwasm/WasmKit/tree/main/.github/workflows)
- [Yams](https://github.com/jpsim/Yams/tree/main/.github/workflows)

