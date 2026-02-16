---
title: iOS Dependency Managers
impact: HIGH
tags: spm, cocoapods, carthage, ios, dependency-management, xcode
---

# Skill: iOS Dependency Managers

Compare and choose between SPM, Carthage, and CocoaPods for iOS dependency management.

## Quick Reference

**Recommended ranking**: SPM > Carthage > CocoaPods

| Feature | SPM | Carthage | CocoaPods |
|---------|-----|----------|-----------|
| Swift support | Full (native) | Full | Full |
| ObjC support | Limited | Full | Full |
| Installation | Built into Xcode | `brew install carthage` | `gem install cocoapods` |
| Project impact | **None** | Almost none | **Heavy** (wraps into workspace, modifies xcodeproj) |
| Removal complexity | Trivial | Easy | Very complex |
| Build speed | Fast (Apple's caching) | Slow initial (prebuilds once) | Rebuilds deps every time |
| Market share | ~46% | ~11% | ~61% |
| Apple maintained | Yes | Community | Community |
| Binary distribution | Supported (XCFramework) | Supported | Limited |

## When to Use

- Starting a new iOS/React Native project and choosing a dependency manager
- Evaluating whether to migrate away from CocoaPods
- Need to understand trade-offs between available options

## Comparison Details

### Swift Package Manager (SPM)

**Strengths:**
- Built into Xcode — no external tools needed
- Minimal project file changes
- Fast dependency resolution with caching
- Apple-maintained, long-term investment
- Easy to add/remove packages
- First-class binary target (XCFramework) support

**Weaknesses:**
- Limited Objective-C library support
- Some RN dependencies still require CocoaPods
- Resource bundle handling can be tricky
- Cannot mix with CocoaPods for the same target easily

**Best for**: New Swift-first projects, libraries with SPM support.

### Carthage

**Strengths:**
- Decentralized — no central spec repo
- Prebuilds frameworks (faster CI if cached)
- Minimal project file intrusion
- Easy to remove
- Good ObjC and Swift support

**Weaknesses:**
- Slow initial build (compiles all dependencies from source)
- Requires manual framework linking
- Smaller ecosystem than CocoaPods
- Some libraries don't support Carthage

**Best for**: Projects wanting prebuilt binaries without project file modification.

### CocoaPods

**Strengths:**
- Largest library ecosystem
- Excellent Objective-C support
- React Native's default dependency manager
- Well-documented integration
- Handles complex dependency graphs

**Weaknesses:**
- **Extremely hard to switch away from** — modifies xcodeproj, creates workspace, adds build phases
- Slow `pod install` on large projects
- Ruby dependency management adds complexity
- Central spec repo can be slow
- Xcode 16 compatibility issues (frequently)

**Best for**: React Native projects (required by most RN libraries), large ObjC codebases.

## Key Insight: CocoaPods Lock-In

CocoaPods is extremely hard to switch away from because it:
1. Creates and manages a `.xcworkspace`
2. Modifies the `.xcodeproj` with custom build phases
3. Manages header search paths and framework search paths
4. Many React Native libraries only provide Podspecs

**Migration effort from CocoaPods**: Weeks to months for a medium-sized RN project. Requires rewriting build configuration and potentially forking libraries.

## Decision Matrix

| Scenario | Recommendation |
|----------|---------------|
| New RN project | CocoaPods (required by RN ecosystem) |
| Pure Swift project, no RN | SPM |
| Need easy dependency removal | SPM or Carthage |
| Large ObjC codebase | CocoaPods |
| Want prebuilt frameworks | Carthage or SPM (binary targets) |
| Evaluating migration from CocoaPods | Only if strong business case justifies the effort |

## Common Pitfalls

- **Attempting to remove CocoaPods casually**: Understand the full scope of project file changes before starting
- **Mixing SPM and CocoaPods for same dependency**: Can cause duplicate symbol errors
- **Not caching Carthage builds**: Initial builds are slow; cache `Carthage/Build` in CI
- **Assuming all RN libraries support SPM**: Most still require CocoaPods

## Related Skills

- [oss-licensing-tools.md](./oss-licensing-tools.md) — License compliance for dependencies
- [objc-to-swift-migration.md](./objc-to-swift-migration.md) — ObjC → Swift migration strategies
