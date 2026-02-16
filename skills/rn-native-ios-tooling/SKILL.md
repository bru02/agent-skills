---
name: rn-native-ios-tooling
description: Covers iOS dependency manager comparison (SPM vs Carthage vs CocoaPods), OSS licensing tools for mobile apps, and Objective-C to Swift migration strategies. Use when evaluating iOS dependency managers, setting up license compliance, or planning an ObjC to Swift migration.
license: MIT
metadata:
  author: Callstack
  tags: react-native, ios, spm, cocoapods, carthage, licensing, objc, swift, migration
---

# React Native Native iOS Tooling

## Overview

Practical guidance for iOS dependency management, open-source license compliance, and Objective-C to Swift migration in React Native projects.

## When to Apply

Reference these guidelines when:
- Choosing or switching iOS dependency managers (SPM, Carthage, CocoaPods)
- Setting up OSS license compliance for iOS/Android apps
- Planning a migration from Objective-C to Swift
- Evaluating the cost of switching away from CocoaPods
- Need cross-platform license notice generation

## Quick Reference

| File | Description |
|------|-------------|
| [ios-dependency-managers.md][ios-dependency-managers] | SPM vs Carthage vs CocoaPods comparison |
| [oss-licensing-tools.md][oss-licensing-tools] | OSS license compliance tools for mobile |
| [objc-to-swift-migration.md][objc-to-swift-migration] | ObjC → Swift migration checklist |

## Problem → Skill Mapping

| Problem | Start With |
|---------|------------|
| Choosing iOS dependency manager | [ios-dependency-managers.md][ios-dependency-managers] |
| Migrating away from CocoaPods | [ios-dependency-managers.md][ios-dependency-managers] |
| Setting up license notices in app | [oss-licensing-tools.md][oss-licensing-tools] |
| Cross-platform license generation | [oss-licensing-tools.md][oss-licensing-tools] |
| Planning ObjC to Swift migration | [objc-to-swift-migration.md][objc-to-swift-migration] |
| Migrating native module to Swift | [objc-to-swift-migration.md][objc-to-swift-migration] |

[ios-dependency-managers]: references/ios-dependency-managers.md
[oss-licensing-tools]: references/oss-licensing-tools.md
[objc-to-swift-migration]: references/objc-to-swift-migration.md
