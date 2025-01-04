# Swift Android native cross-compiler and test runner action

This GitHub action will build and run Swift package tests on an Android emulator.
This uses the [swift-android-toolchain](https://github.com/skiptools/swift-android-toolchain)
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
        uses: skiptools/swift-android-action@v2
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
        uses: skiptools/swift-android-action@v2
        with:
          swift-version: 6.0.2
```

### Configuration Options

| Parameter | Default | Description  |
|-----|-----|-----|
| swift-version | The version of the Swift toolchain to use | 6.0.2 |
| package-path | The folder where the swift package is checked out | . |
| swift-build-flags | Additional flags to pass to the swift build command |  |
| swift-test-flags | Additional flags to pass to the swift test command |  |
| build-tests | Whether to build the package tests or just the sources | true |
| run-tests | Whether to run the tests or just perform the build | true |
| copy-files | Additional files to copy to emulator for testing | |
| android-api-level | The API level of the Android emulator to run against | 29 |
| android-emulator-options | Options to pass to the Android emulator | -no-window … |
| android-emulator-boot-timeout | Emulator boot timeout in seconds | 600 |

### Platform Support

This action can be run on any of the GitHub `ubuntu-*` and `macos-*` [runner images](https://github.com/actions/runner-images). However, due to the inability of macOS on ARM to run nested virtualization ([issue](https://github.com/ReactiveCircus/android-emulator-runner/issues/350)), the Android emulator cannot be run on these platforms, and so running on any macOS image that uses ARM (including `macos-14` and `macos-15`) requires disabling tests with `run-tests: false`. Running tests are supported on `macos-13`, as well as the large Intel macOS images like `macos-14-large` and `macos-15-large`.

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
        uses: skiptools/swift-android-action@v2
  macos-ios:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v4
      - name: "Test Swift Package on macOS"
        run: swift test
      - name: "Test Swift Package on iOS"
        run: xcodebuild test -sdk "iphonesimulator" -destination "platform=iOS Simulator,name=iPhone 15" -scheme "$(xcodebuild -list -json | jq -r '.workspace.schemes[-1]')"
  windows:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4
      - name: "Setup Swift on Windows"
        uses: compnerd/gha-setup-swift@main
        with:
          branch: swift-6.0.2-release
          tag: 6.0.2-RELEASE
      - name: "Test Swift Package on Windows"
        run: swift test

```

For an example of this workflow in action, see a run history
for the [Swift Algorithms](https://github.com/skiptools/swift-algorithms) package at:
[https://github.com/skiptools/swift-algorithms/actions](https://github.com/skiptools/swift-algorithms/actions)


## Development

### Releasing new versions

To create a new release, make a new tag (like 2.0.2),
and then update the symbolic major v2 tag with:

```
git tag v2.0.2
git push --tags
git tag -fa v2 -m "Update v2 tag"
git push origin v2 --force
```


