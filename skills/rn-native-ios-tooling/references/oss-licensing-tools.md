---
title: OSS Licensing Tools
impact: MEDIUM
tags: licensing, oss, compliance, android, ios, react-native, legal
---

# Skill: OSS Licensing Tools

Set up open-source license compliance and notice generation for mobile apps.

## Quick Reference

| Tool | Platform | Type |
|------|----------|------|
| AboutLibraries | Android | Native Gradle plugin |
| LicensePlist | iOS | CocoaPods/SPM integration |
| Google OSS Licenses Plugin | Android | Gradle plugin |
| `with-react-native-oss-notice` | Cross-platform | Expo config plugin (Callstack, internal) |
| ORT (OSS Review Toolkit) | Cross-platform | Full compliance pipeline |
| FOSSology | Cross-platform | Compliance scanning |

## When to Use

- App store submission requires license attribution
- Legal/compliance review of third-party dependencies
- Need to generate a "licenses" or "acknowledgements" screen in the app
- Auditing OSS license compatibility

## Platform-Specific Tools

### Android: AboutLibraries

Gradle plugin that automatically collects license info from dependencies:

```groovy
// build.gradle
plugins {
    id "com.mikepenz.aboutlibraries.plugin" version "10.10.0"
}
```

```kotlin
// In-app usage
AboutLibraries.Builder()
    .withAutoDetect(true)
    .start(context)
```

**Strengths**: Automatic detection, UI component included, widely used. Can use the built-in UI or extract generated metadata for custom display.

### Android: Google OSS Licenses Plugin

```groovy
// build.gradle
plugins {
    id 'com.google.android.gms.oss-licenses-plugin'
}

dependencies {
    implementation 'com.google.android.gms:play-services-oss-licenses:17.0.1'
}
```

```kotlin
startActivity(Intent(this, OssLicensesMenuActivity::class.java))
```

**Strengths**: Google-maintained, simple integration.
**Weaknesses**: Only detects Google Play Services and Gradle dependencies.

### iOS: LicensePlist

Generates a Settings.bundle plist from CocoaPods/Carthage/SPM dependencies:

```bash
# Install
brew install licenseplist

# Generate
license-plist --output-path Settings.bundle
```

Or via SPM/CocoaPods integration as a build phase:

```bash
# Xcode Run Script Phase
if which license-plist > /dev/null; then
    license-plist --output-path $SRCROOT/Settings.bundle \
                  --github-token $LICENSE_PLIST_GITHUB_TOKEN
fi
```

Licenses appear in iOS Settings app under the app's entry.

**Strengths**: Automatic, integrates with iOS Settings.
**Weaknesses**: iOS only, Settings.bundle approach may not match desired UX.

## Cross-Platform Tools

### with-react-native-oss-notice (Callstack)

Expo config plugin for generating license notices on both platforms:

```bash
npx expo install with-react-native-oss-notice
```

```json
// app.json
{
  "expo": {
    "plugins": ["with-react-native-oss-notice"]
  }
}
```

Generates platform-appropriate license files during the build process. Combines platform-specific tools (AboutLibraries + LicensePlist) under one config plugin.

> **Note**: This is a Callstack internal library, not yet publicly released.

### ORT (OSS Review Toolkit)

Full compliance pipeline: analyze → scan → evaluate → report.

```bash
# Analyze project dependencies
ort analyze -i /path/to/project -o /path/to/output

# Scan for license texts
ort scan -i /path/to/output/analyzer-result.yml -o /path/to/output

# Generate report
ort report -i /path/to/output/scan-result.yml -o /path/to/output \
  -f NoticeTemplate
```

**Best for**: Enterprise compliance requirements, large projects needing full audit trail.

### FOSSology

Open-source compliance scanning platform:
- Server-based scanning
- License identification from source code
- Bulk scanning of dependencies
- Compliance workflow management

**Caveat**: Very generic scanner — does not specifically scan for licenses in third-party libraries. Better as a supplement to platform-specific tools.

**Best for**: Organizations needing ongoing compliance scanning infrastructure.

## Decision Matrix

| Scenario | Recommendation |
|----------|---------------|
| Android-only app | AboutLibraries |
| iOS-only app | LicensePlist |
| React Native / Expo app | `with-react-native-oss-notice` |
| Enterprise compliance | ORT + FOSSology |
| Quick "licenses" screen | AboutLibraries (Android) + LicensePlist (iOS) |

## Common Pitfalls

- **Relying only on automatic detection**: Some transitive dependencies may be missed — verify manually
- **Ignoring license compatibility**: MIT + GPL in same app can create legal issues
- **Not updating before release**: Run license generation as part of release process
- **Missing native dependency licenses**: JS-only tools miss CocoaPods/Gradle-only dependencies

## Related Skills

- [ios-dependency-managers.md](./ios-dependency-managers.md) — Understanding dependency sources
- [objc-to-swift-migration.md](./objc-to-swift-migration.md) — Migration may change dependency licensing
