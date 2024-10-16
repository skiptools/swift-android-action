# Swift Android native cross-compiler and test runner action

GitHub action to build and run Swift package tests on an Android emulator.
This uses the [swift-android-toolchain](https://github.com/skiptools/swift-android-toolchain)
project to provide a cross-compiler for building
Swift natively for Android on a macOS host.

After building the package, it will run the SwiftPM
test targets on an Android emulator (which it provided by the 
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

```yml
name: Swift Package CI
on:
  push:
    branches: [ main ]
  pull_request:
    branches:
      - '*'
jobs:
  test:
    # Runner must be macos-13, since lack of nested virtualization support on macos-14 prevents the Android emulator from working
    # see: https://docs.github.com/en/actions/using-github-hosted-runners/using-github-hosted-runners/about-github-hosted-runners#limitations-for-arm64-macos-runners
    runs-on: macos-13
    steps:
      - uses: actions/checkout@v4
      - name: "Test Swift Package on macOS"
        run: swift test
      - name: "Test Swift Package on Android"
        uses: skiptools/swift-android-action@v1
      - name: "Test Swift Package on iOS"
        run: xcodebuild test -sdk "iphonesimulator" -destination "platform=iOS Simulator,name=iPhone 15" -scheme "$(xcodebuild -list -json | jq -r '.workspace.schemes[-1]')" -skipPackagePluginValidation

```

For an example of a workflow in action, see a run history
for the [Swift Algorithms](https://github.com/skiptools/swift-algorithms) package at:
[https://github.com/skiptools/swift-algorithms/actions](https://github.com/skiptools/swift-algorithms/actions)

## Configuration Options


```
  swift-version:
    description: 'The version of the Swift toolchain to use'
    default: '6.0.1'
  package-path:
    description: 'The folder where the swift package is checked out'
    default: '.'
  swift-build-flags:
    description: 'Additional flags to pass to the swift build command'
    default: ''
  swift-test-flags:
    description: 'Additional flags to pass to the swift test command'
    default: ''
  run-tests:
    description: 'Whether to run the tests or just the build'
    default: 'true'
  android-api-level:
    description: 'The API level of the Android emulator to run against'
    default: 29
  android-emulator-options:
    description: 'Options to pass to the Android emulator'
    default: '-no-window -gpu swiftshader_indirect -no-snapshot -noaudio -no-boot-anim'
  android-emulator-arch:
    description: 'Architecture for the Android emulator (x86_64 or arm64-v8)'
    required: true
    default: 'x86_64'
  android-emulator-boot-timeout:
    description: 'Emulator boot timeout in seconds'
    default: 600
```

## Releasing

To create a new release, make a new tag (like 1.0.0),
and then update the symbolic major v1 tag with:

```
git tag v1.0.2 && git push --tags
git tag -fa v1 -m "Update v1 tag" && git push origin v1 --force
```


