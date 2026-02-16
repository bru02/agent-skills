---
name: rn-new-architecture-migration
description: Guides React Native New Architecture migration, focusing on XCFramework distribution for brownfield iOS apps. Use when packaging React Native as a reusable XCFramework, integrating RN into existing iOS apps, or migrating to the New Architecture.
license: MIT
metadata:
  author: Callstack
  tags: react-native, new-architecture, xcframework, brownfield, ios, swift-package
---

# React Native New Architecture Migration

## Overview

Covers packaging and distributing React Native as an XCFramework for brownfield iOS integration, including bridge initialization, Hermes embedding, and Swift Package distribution.

## When to Apply

Reference these guidelines when:
- Packaging React Native as a reusable XCFramework for existing iOS apps
- Integrating React Native into a brownfield iOS project
- Setting up RN bridge initialization for old architecture (<0.78)
- Using RNViewFactory or ReactNativeFactory patterns
- Generating universal XCFramework binaries
- Distributing React Native via Swift Package Manager

## Quick Reference

| File | Description |
|------|-------------|
| [xcframework-distribution.md][xcframework-distribution] | Package and distribute RN as XCFramework for brownfield iOS |

## Problem → Skill Mapping

| Problem | Start With |
|---------|------------|
| Need to embed RN in existing iOS app | [xcframework-distribution.md][xcframework-distribution] |
| Building XCFramework from RN project | [xcframework-distribution.md][xcframework-distribution] |
| Bridge initialization for old arch | [xcframework-distribution.md][xcframework-distribution] |
| Hermes embedding in XCFramework | [xcframework-distribution.md][xcframework-distribution] |

[xcframework-distribution]: references/xcframework-distribution.md
