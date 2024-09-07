# swift-test

GitHub action to build and run Swift package tests on an Android emulator

You can add this action to your Swift CI workflow from the
[GitHub Marketplace](https://github.com/marketplace/actions/swift-android-action).

## Releasing

To create a new release, make a new tag (like 1.0.0),
and then update the symbolic major v1 tag with:

```
git tag -fa v1 -m "Update v1 tag"
git push origin v1 --force
```


