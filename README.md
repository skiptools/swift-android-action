# Swift Android native cross-compiler and test runner action

GitHub action to build and run Swift package tests on an Android emulator

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
    # important: must be macos-13, since the Android emulator does not work on macos-14
    runs-on: macos-13
    steps:
      - uses: actions/checkout@v4
      - name: "Test Swift Package Locally"
        run: swift test
      - name: "Test Swift Package on Android"
        uses: skiptools/swift-android-action@v1
      - name: "Test Swift Package on iOS"
        run: xcodebuild test -scheme "$(xcodebuild -list -json | jq -r '.workspace.schemes[-1]')" -sdk "iphonesimulator" -destination "platform=iOS Simulator,name=iPhone 15"

```



## Releasing

To create a new release, make a new tag (like 1.0.0),
and then update the symbolic major v1 tag with:

```
git tag -fa v1 -m "Update v1 tag"
git push origin v1 --force
```


