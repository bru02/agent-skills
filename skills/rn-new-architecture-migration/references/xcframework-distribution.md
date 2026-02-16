---
title: XCFramework Distribution
impact: HIGH
tags: xcframework, brownfield, ios, swift-package, hermes, bridge, new-architecture
---

# Skill: XCFramework Distribution

Package React Native as a reusable XCFramework for brownfield iOS app integration.

## Quick Reference

| Step | Action |
|------|--------|
| 1 | Create RN app with `npx react-native init` |
| 2 | Add Swift Framework target in Xcode |
| 3 | Configure Build Settings |
| 4 | Add Run Script Phase for Metro bundle |
| 5 | Set up Podfile integration |
| 6 | Initialize RN bridge (old arch) or use ReactNativeFactory (0.78+) |
| 7 | Generate XCFramework with `xcodebuild` |

## When to Use

- Embedding React Native into an existing native iOS app
- Distributing RN-based features as a prebuilt binary
- Building a reusable SDK powered by React Native
- Brownfield integration requiring framework-level packaging

## Prerequisites

- Xcode 15+
- React Native 0.73+ (0.78+ recommended for ReactNativeFactory)
- CocoaPods installed
- Familiarity with Xcode build settings and targets

## Step-by-Step Instructions

### 1. Create React Native App

```bash
npx @react-native-community/cli@latest init MyRNApp
cd MyRNApp
```

This serves as the source project from which the framework is built.

### 2. Add Swift Framework Target

In Xcode:
1. File → New → Target → Framework
2. Select Swift as the language
3. Set "Embed in Application" to **None**
4. Name it (e.g., `MyRNFramework`)
5. Set deployment target to match the RN app

### 3. Configure Build Settings

Set these on the framework target:

| Setting | Value | Why |
|---------|-------|-----|
| `User Script Sandboxing` | `NO` | Allows Run Script phases to access Metro bundle |
| `Skip Install` | `NO` | Ensures the framework is included in archives |
| `Build Libraries for Distribution` | `YES` | Required for XCFramework generation |
| `DEFINES_MODULE` | `YES` | Generates module map for Swift imports |

### 4. Add Run Script Phase

Add a Run Script build phase to the framework target that bundles JS via Metro:

```bash
set -e

WITH_ENVIRONMENT="$REACT_NATIVE_PATH/scripts/xcode/with-environment.sh"
REACT_NATIVE_XCODE="$REACT_NATIVE_PATH/scripts/react-native-xcode.sh"

/bin/sh -c "$WITH_ENVIRONMENT $REACT_NATIVE_XCODE"
```

This embeds the JS bundle into the framework during build.

### 5. Podfile Integration

Nest the framework target inside the app target in the Podfile with `inherit! :complete`:

```ruby
target 'MyRNApp' do
  config = use_native_modules!

  use_react_native!(
    :path => config[:reactNativePath],
    :app_path => "#{Pod::Config.instance.installation_root}/.."
  )

  # Nest framework target to inherit all RN pods
  target 'MyRNFramework' do
    inherit! :complete
  end
end
```

Then install:

```bash
npx pod-install ios
```

### 6. Bridge Initialization

#### React Native 0.78+ (ReactNativeFactory)

```swift
import React
import ReactNativeFactory

public class MyRNView {
    private var factory: ReactNativeFactory?

    public func createView() -> UIView {
        factory = ReactNativeFactory()
        return factory!.createRootView(
            moduleName: "MyRNApp",
            initialProperties: nil
        )
    }
}
```

#### React Native <0.78 (Old Architecture)

Use a singleton bridge manager with `@_implementationOnly import` to hide React headers from consumers:

```swift
import UIKit
@_implementationOnly import React

public class RNBridgeManager {
    public static let shared = RNBridgeManager()

    private var bridge: RCTBridge?

    public func initialize(launchOptions: [UIApplication.LaunchOptionsKey: Any]?) {
        bridge = RCTBridge(delegate: RNBridgeDelegate(), launchOptions: launchOptions)
    }

    func getBridge() -> RCTBridge {
        guard let bridge = bridge else {
            fatalError("RN bridge must be initialized before creating a RN view!")
        }
        return bridge
    }
}

private class RNBridgeDelegate: NSObject, RCTBridgeDelegate {
    func sourceURL(for _: RCTBridge) -> URL? {
        #if DEBUG
        return RCTBundleURLProvider.sharedSettings().jsBundleURL(forBundleRoot: "index")
        #else
        return Bundle(for: Self.self).url(forResource: "main", withExtension: "jsbundle")
        #endif
    }
}
```

### 7. RNViewFactory Pattern

Expose a factory with typed module names for consuming apps:

```swift
import UIKit
@_implementationOnly import React

public struct RNViewFactory {
    public static func createView(
        moduleName: RNModule,
        initialProperties: [String: Any]? = nil
    ) -> UIView {
        let bridge = RNBridgeManager.shared.getBridge()
        return RCTRootView(
            bridge: bridge,
            moduleName: moduleName.rawValue,
            initialProperties: initialProperties
        )
    }
}

public enum RNModule: String {
    case myFeature = "MyFeature"
}
```

The consuming app calls `RNViewFactory.createView(moduleName: .myFeature)` without needing React Native knowledge.

### 8. Hermes Embedding

Hermes is the recommended JS engine but complicates XCFramework distribution:

- **Recommended**: Treat Hermes as a peer dependency — require the consuming app to include `hermes-engine` via CocoaPods
- **Not recommended**: Embedding `Hermes.xcframework` directly into the output framework (causes symbol duplication issues)

#### dSYM Fix for React Native <0.76

Builds may fail due to a `DebugSymbolsPath` bug in `hermes.xcframework/Info.plist` (fixed in RN 0.76). Remove the problematic entries in Podfile post-install using PlistBuddy:

```ruby
post_install do |installer|
  react_native_post_install(
    installer,
    config[:reactNativePath],
    :mac_catalyst_enabled => false,
  )

  # Remove broken DebugSymbolsPath from hermes.xcframework
  PLIST_BUDDY_PATH = '/usr/libexec/PlistBuddy'
  installer.pods_project.targets.each do |target|
    next unless target.name == "hermes-engine"
    installer.pods_project.files.each do |fileref|
      next unless fileref.path.end_with?("hermes.xcframework")
      plist_file = "#{fileref.real_path}/Info.plist"
      stdout, _stderr, _status = Open3.capture3(
        "#{PLIST_BUDDY_PATH} -c 'Print :AvailableLibraries' #{plist_file}"
      )
      stdout.scan(/Dict/).count.times do |i|
        Open3.capture3(
          "#{PLIST_BUDDY_PATH} -c 'Delete :AvailableLibraries:#{i}:DebugSymbolsPath' #{plist_file}"
        )
      end
    end
  end
end
```

### 9. Generate XCFramework

Build for both simulator and device, then combine:

```bash
#!/bin/bash

WORKSPACE=${WORKSPACE:-MyRNApp}
SCHEME=${SCHEME:-MyRNFramework}
CONFIGURATION=${BUILD_TYPE:-Release}
ARCHIVE_DIR="archives"

cd ios

# Build for iOS Simulator
xcodebuild archive \
  -workspace $WORKSPACE.xcworkspace \
  -scheme $SCHEME \
  -destination "generic/platform=iOS Simulator" \
  -configuration $CONFIGURATION \
  -archivePath $ARCHIVE_DIR/${SCHEME}-iOS_Simulator.xcarchive \
  -quiet || exit 1

# Build for iOS device
xcodebuild archive \
  -workspace $WORKSPACE.xcworkspace \
  -scheme $SCHEME \
  -destination "generic/platform=iOS" \
  -configuration $CONFIGURATION \
  -archivePath $ARCHIVE_DIR/${SCHEME}-iOS.xcarchive \
  -quiet || exit 1

# Remove previous output
rm -rf $ARCHIVE_DIR/$SCHEME.xcframework

# Create XCFramework
xcodebuild -create-xcframework \
  -archive $ARCHIVE_DIR/${SCHEME}-iOS_Simulator.xcarchive -framework $SCHEME.framework \
  -archive $ARCHIVE_DIR/${SCHEME}-iOS.xcarchive -framework $SCHEME.framework \
  -output $ARCHIVE_DIR/$SCHEME.xcframework

cd -
```

### 10. Distribution via Swift Package (Future)

Swift Package Manager distribution requires:
- Hosting the XCFramework as a binary target (zip + checksum)
- Creating a `Package.swift` with `.binaryTarget`

```swift
// Package.swift (example structure)
let package = Package(
    name: "MyRNFramework",
    platforms: [.iOS(.v15)],
    products: [
        .library(name: "MyRNFramework", targets: ["MyRNFramework"]),
    ],
    targets: [
        .binaryTarget(
            name: "MyRNFramework",
            url: "https://example.com/MyRNFramework.xcframework.zip",
            checksum: "<sha256>"
        ),
    ]
)
```

## FAQ

### Does this support both old and new architecture?
The bridge-based approach (steps 6-7) only supports the old architecture. On RN 0.78+, use `ReactNativeFactory` (see [PR #46298](https://github.com/facebook/react-native/pull/46298)) which supports both architectures.

### What architectures are supported?
XCFramework supports arm64 (device) and arm64 + x86_64 (simulator). Use `lipo -info` to verify slices.

### Can Hermes be bundled inside the XCFramework?
No — there is no support for including an XCFramework inside another XCFramework. Include `hermes.xcframework` directly in the brownfield app as a "peer dependency."

### pod install fails on Xcode 16?
RN 0.76 doesn't have proper Xcode 16 support. Update CocoaPods:
```bash
gem install cocoapods --pre
```

## Common Pitfalls

- **User Script Sandboxing enabled**: Metro bundling fails silently. Always set to `NO`.
- **Skip Install = YES**: The framework won't appear in the archive. Set to `NO`.
- **Embedding Hermes directly**: Causes duplicate symbol errors in consuming apps.
- **Missing JS bundle**: Ensure the Run Script phase runs before the framework's Compile Sources phase.
- **Architecture mismatch**: Always build both arm64 and x86_64 simulator slices.

## Related Skills

- [rn-native-ios-tooling](../../rn-native-ios-tooling/SKILL.md) — iOS dependency managers and tooling
